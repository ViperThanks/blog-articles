---
title: "从0开始编写秒杀-1 (乐观锁version版本)"
datePublished: Fri Jul 07 2023 10:22:31 GMT+0000 (Coordinated Universal Time)
cuid: clk2fl2im000b09mk5ogjfdr4
slug: 0-1-version
tags: mysql

---

秒杀业务在黑马点评上是一个侧重点。但是黑马点评用的是 Redis 的五大基本数据类型之外的publisher + consumer实现的，看起来好像不专业。不如直接用MQ去实现。

# 准备数据

SQL数据的init准备

```sql
-- ----------------------------
-- Table structure for stock
-- ----------------------------
DROP TABLE IF EXISTS `stock`;
CREATE TABLE `stock`
(
    `id`      int(11) unsigned NOT NULL AUTO_INCREMENT,
    `name`    varchar(50)      NOT NULL DEFAULT '' COMMENT '名称',
    `count`   int(11)          NOT NULL COMMENT '库存',
    `sale`    int(11)          NOT NULL COMMENT '已售',
    `version` int(11)          NOT NULL COMMENT '乐观锁，版本号',
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- ----------------------------
-- Table structure for stock_order
-- ----------------------------
DROP TABLE IF EXISTS `stock_order`;
CREATE TABLE `stock_order`
(
    `id`          int(11) unsigned NOT NULL AUTO_INCREMENT,
    `sid`         int(11)          NOT NULL COMMENT '库存ID',
    `name`        varchar(30)      NOT NULL DEFAULT '' COMMENT '商品名称',
    `create_time` timestamp        NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- init stock data
-- 默认自增id，小demo不考虑业务上的订单id问题
insert into stock (id, name, count, sale, version)
values (1, 'iphone', 100, 0, 0);
insert into stock (id, name, count, sale, version)
values (2, 'huawei', 100, 0, 0);
insert into stock (id, name, count, sale, version)
values (3, 'xiaomi', 100, 0, 0);
```

# Java和maven准备

本次基于SpringBoot+Mybatis-plus快速开发写的

目录结构图 ：

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689328226161/98951577-812b-41ce-bd3e-1bdc5c37880d.png align="center")

## Controller层

```java
  @RequestMapping(value = "createWrongOrder/{id}", method = RequestMethod.GET)
  public String createWrongOrder(@PathVariable(value = "id") Integer sid) {
    LOGGER.info("购买物品的编号id：[{}]", sid);
    int id = 0;
    try {
      id = stockOrderService.createWrongOrder(sid);
      LOGGER.info("创建订单id[{}]", id);
    } catch (Exception e) {
      LOGGER.error(String.valueOf(e));
    }
    return String.valueOf(id);
  }
```

## Service 层

```java
  @Override
  public int createWrongOrder(Integer sid) {
    //校验库存
    Stock stock = checkStock(sid);
    //扣库存
    saleWrongStock(stock);
    //创建订单
    int id = createOrder(stock);
    return id;
  }

  private void saleWrongStock(Stock stock) {
    stockMapper.saleWrongStock(stock);
  }

  private Stock checkStock(int sid) {
    Stock stock = stockMapper.selectById(sid);
    if (stock.getSale() > (stock.getCount())) {
      throw new RuntimeException("库存不足");
    }
    return stock;
  }

  private int createOrder(Stock stock) {
    StockOrder order = new StockOrder();
    order.setSid(stock.getId());
    order.setName(stock.getName());
    this.save(order);
    return order.getId();
  }
```

## Mapper层

```xml
<update id="saleWrongStock" parameterType="com.yiren.sec.domain.Stock">
        update seckill_demo.stock
        <set>
            sale = sale + 1
        </set>
        where id = #{id}
 </update>
```

# Jmeter并发测试

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689328474818/7f25d374-b66b-4dd7-ac63-3b4d2d067e19.png align="center")

# 结果（没有用乐观锁）

多买了71件，太赚了（等着跑路吧）

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689328633077/c6eff644-d74b-4ec0-9c6c-8511b6c0c8de.png align="center")

# 乐观锁

先从乐观锁的定义开始聊，乐观锁与悲观锁是相对的。  

* 乐观锁适用于并发冲突较少发生的场景，具有较高的并发性能，但是需要额外的校验和处理机制来处理冲突。
    
* 悲观锁适用于并发冲突频繁发生的场景，具有较高的安全性，但是会降低并发性能。
    

再说实现

* 乐观锁有TimeStamp时间戳实现，Version版本实现，CAS
    

其实在Mysql乐观锁实现很简单，但是很底层，因为要修改到表的操作。只需要加个字段，比如 时间戳，版本号。

然后核心语句就是

```sql
update tb_xxx
set bussiness_field = xxx
and version = version + 1
where sign = #{xxx} 
and version = #{version}
```

```xml
<update id="saleOptimismStock" parameterType="com.yiren.sec.domain.Stock" >
        update seckill_demo.stock
        <set>
            sale = sale + 1,
            version = version + 1
        </set>
        where id = #{id,jdbcType=INTEGER}
        and version = #{version,jdbcType=INTEGER}
    </update>
```

## 结果

注意：乐观锁还是会有超售的现象的，我sale之前有一次101了

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689329821842/52c9cac9-a423-4a80-b63c-3ef2e54a9edc.png align="center")