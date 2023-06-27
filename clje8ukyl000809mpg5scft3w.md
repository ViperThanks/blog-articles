---
title: "基于aop实现的注解日志"
datePublished: Fri May 26 2023 12:07:43 GMT+0000 (Coordinated Universal Time)
cuid: clje8ukyl000809mpg5scft3w
slug: aop
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1687867334113/083cb07b-5ed5-4ad8-a24b-d50f886be4f4.png
tags: java, springboot, spring-aop, mybatisplus

---

$$基于SpringBoot+MybatisPlus+AOP快速开发$$

# Introduction

由于业务的需求，最好知道每一次在 **Controller** 层的用户信息和 *request* 请求中的 IP，请求方式（*Post，Get，Put，Delete* 等） ，所以就写了 demo 来实现

大概长成这样子 ：

```java

@RestController
@Slf4j
@RequestMapping("/user")
public class UserController {

  @RequestMapping(value = "/get/{id}", method = RequestMethod.GET)
  @Log(value = "获取用户名，并且返回数据")
  public Response<User> getById(@PathVariable(value = "id") Long userId) {
    User user = userService.getById(userId);
    return ResponseFactory.getSuccessResponse(user);
  }

}
```

当用户请求 [https://hostip/user/get/1](https://hostip/user/get/1) 这个 url 的时候在日志输出

业务名称：获取用户名，并且返回数据

方法名是：getById

Url 是：/user/get/1

方式是：GET

ip 是：0:0:0:0:0:0:0:1

操作时间是：2023-06-27 T17:12:04.176

# 实现 ：

## 导坐标

首先，因为要用到 **AOP** （*面向切片编程*）的技术，所以需要导坐标, 由于是 **springboot**，所以不用导 GAV 的 *Version*

```xml
    <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
    </dependency>
```

## SpringApplication 启动类加@EnableAspectJAutoProxy

```java
@SpringBootApplication
@EnableAspectJAutoProxy // 开启AOP
public class SpringBootAopPractiseApplication {
  public static void main(String[] args) {
    SpringApplication.run(SpringBootAopPractiseApplication.class, args);
  }

}
```

## 编写注解类

这里可以自定义名字，我就用 Log 了。由于我只想作用到 Method 方法上，所以就在 **@Target** 放 *ElementType.METHOD*

这里 Value 的别名是因为我喜欢

```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Log {
  /**
   * 业务名称
   */
  String name() default "";
  /**
   * 别名
   */
  @AliasFor(value = "name")
  String value() default "";
}
```

## 具体实现

### 1.以注解为 pointcut 然后我们做环绕（round）就行了

**executeJob()** 是我们具体实现的方法

**LogAspect** 类通过结合 *@Component* 、*@Aspect* 和切点表达式，实现了将切面类纳入 **Spring** 管理，并对使用了 *@Log* 注解的方法进行切面增强。并在 **around()** 方法上使用 *@Around* 注解定义了环绕通知，它将在匹配到的目标方法执行前后执行自定义的逻辑。

```java
@Component
@Aspect
public class LogAspect {

  private static final Logger log = org.slf4j.LoggerFactory.getLogger(LogAspect.class);

  @Pointcut("@annotation(com.yiren.practise.annotation.Log)")
  private void pointcut() {
  }

  @Around("pointcut()")
  public Object around(ProceedingJoinPoint pjp) throws Throwable {
    executeJob(pjp);
    return pjp.proceed();
  }
}
```

### 2.实现 executeJob（）方法 **⚠️significant**

#### 尝试去获取 Mehod 的所有信息

首先我们获得了 **ProceedingJoinPoint** 的 *pjp* 对象，我们可以获得 *signature*，再转成**MethodSignature**就可以获得 Method 也就是 **需要增强的方法了**，这样我们就可以获得*方法名*和*注解里面的值*。再通过工具类来获得 **HttpServeletRequest** 对象

```java
RequestAttributes requestAttribute = RequestContextHolder.getRequestAttributes();
    ServletRequestAttributes servletRequestAttributes = (ServletRequestAttributes) requestAttribute;
    assert servletRequestAttributes != null;
    HttpServletRequest request = servletRequestAttributes.getRequest();
```

获得 *request* 对象我们就可以实现剩下的*用户方*的信息了，接下来的实现就比较简单。

完整代码如下

```java
@Component
@Aspect
public class LogAspect {

  private static final Logger log = org.slf4j.LoggerFactory.getLogger(LogAspect.class);

  @Pointcut("@annotation(com.yiren.practise.annotation.Log)")
  private void pointcut() {
  }

  @Around("pointcut()")
  public Object around(ProceedingJoinPoint pjp) throws Throwable {
    executeJob(pjp);
    return pjp.proceed();
  }


  /**
   * 执行任务 目的是为了获取日志信息
   */
  private void executeJob(ProceedingJoinPoint pjp) {
    final StringBuffer requestLog = new StringBuffer();
    //获取你的用户名  （用seesion获取）

    //获取被增强的方法的信息
    Signature signature = pjp.getSignature();
    MethodSignature methodSignature = (MethodSignature) signature;
    //获取方法对象
    Method method = methodSignature.getMethod();

    if (Objects.isNull(method)) {
      log.error("方法上没有注解,请检查");
      return;
    }

    //获取方法上的注解
    Log log = method.getAnnotation(Log.class);
    //获取注解的值
    String value = log.value();
    System.out.println("业务名称：" + value);
    requestLog.append("业务名称：").append(value).append("\n");
    //获取方法名
    String name = method.getName();
    System.out.println("方法名是：" + name);
    requestLog.append("方法名是：").append(name).append("\n");

    //想方设法获取Request对象
    RequestAttributes requestAttribute = RequestContextHolder.getRequestAttributes();
    ServletRequestAttributes servletRequestAttributes = (ServletRequestAttributes) requestAttribute;
    assert servletRequestAttributes != null;
    HttpServletRequest request = servletRequestAttributes.getRequest();

    //url
    String url = request.getRequestURI();
    System.out.println("url是：" + url);
    requestLog.append("url是：").append(url).append("\n");
    //方式
    String method1 = request.getMethod();
    System.out.println("方式是：" + method1);
    requestLog.append("方式是：").append(method1).append("\n");
    //ip
    String ip = request.getRemoteAddr();
    System.out.println("ip是：" + ip);
    requestLog.append("ip是：").append(ip).append("\n");
    //操作时间
    LocalDateTime operateTime = LocalDateTime.now();
    System.out.println("操作时间是：" + operateTime);
    requestLog.append("操作时间是：").append(operateTime).append("\n");

    System.out.println("\n=======happy everyday=======\n");
    System.out.println(requestLog.toString());
    //这里可以分布式锁，保证日志不会被多个线程写入
    synchronized (this) {
      //写入日志 这里可以写入数据库或者文件，一般推荐到一定量或者一定时间写入数据库
      //code 实现
    }
  }
}
```

#### 注意事项

首先 *System.out.println* 是一般开发不可能用到的东西，速度慢。一般来说是打 *log* 日志，但是没有人线上部署会盯着控制台看，所有这里推荐使用**其他 API** （XXL-JOB 这些定时比较好）来实现定时刷入 *IO 磁盘*或者*数据库*，如果每次来请求都刷到磁盘和数据库，那你的机器可能顶不住

<mark>过几天我会实现一些基于 XXL-JOB 或者其他 JavaSe 的实现来完成, 这样就可以实现统计等其他对于业务的优化</mark>

# 结果图

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687867357157/5c52b8b8-9fb3-44b8-8115-49d4e5cf3b8f.png align="left")