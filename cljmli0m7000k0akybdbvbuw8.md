---
title: "重装系统（因为c盘太小了）"
datePublished: Wed May 17 2023 08:23:55 GMT+0000 (Coordinated Universal Time)
cuid: cljmli0m7000k0akybdbvbuw8
slug: c
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1688372238760/f8cc7175-78f3-457c-881c-0aa73da495f1.png
tags: system

---

# 1.备份文件

## 废话

老生常谈的 Java CV工程师了，对本地主机的依赖（耦合度）还是很高的。比如Mysql ， Redis ， Nacos， ES（elastic search），RabbitMQ -&gt; erlang , eureka，Nginx 等。就不一一列举了。

我在前几个月就已经把RabbitMQ + Mysql丢到腾讯云的服务器里了。但是基于服务器只有一年，再加上现在有时候搞qps的测试，使用vps的服务似乎不太明智。

## 行动

1. ### Mysql的备份
    

作为免费的开源的社区版 db MySQL，他在世界的占有率是很高的，无疑就有自己的备份命令，由于我是清所有的盘，就得使用全备份了

```sql
mysqldump -u root -p --all-databases > E:\backup.sql
```

这里得注意导出来的数据要和目标的数据库版本最好一样，密码也是。

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688372389461/aaa16bad-9b6f-442d-9dd3-d93b74b66900.png align="center")

1. ### Redis的备份
    

涉及到 redis 的持久化的原理，redis持久化是用 .rdb文件 或者 .aof文件实现的持久化，所以直接bgsave就行，不过如果本机就可以直接save命令就行了，因为虽然redis是单线程的 ，但是localhost的数据量和访问次数都可以忽略不计

```sql
bgsave or save
```

1. ### RabbitMQ和ES
    
    我用Docker+vps实现的，建议到新电脑用docker就行了
    
2. ### Nginx把整个文件夹拉过来就行了
    

# 2.重装系统

在windows上下载windows10的文件就行了

# 3\. 新系统，新环境

清单如下：

1. JDK8 -&gt; JDK11
    
2. Mysql5 -&gt; Mysql8 -&gt; 尽量全部转到VPS
    
3. Redis -&gt; 交给虚拟机
    
4. RabbitMQ -&gt; 交给虚拟机
    
5. ES -&gt; 交给虚拟机 腾讯好像还有专门的es服务器
    
6. CopyQ 剪贴板记录软件
    
7. 新版 Idea 软件
    

到时候陆续完成