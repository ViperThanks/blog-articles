---
title: "JWT的实现邮箱验证小Demo"
datePublished: Thu Apr 27 2023 13:44:09 GMT+0000 (Coordinated Universal Time)
cuid: cljeca5yc001c09l9gnr3aejn
slug: jwtdemo
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1687873351387/da71007f-94f9-4884-a552-cac9537c8dee.png
tags: jwt, springboot

---

## 1 案例1邮件激活

### 1.1 需求与分析

**需求：当用户注册成功，给指定邮箱发送一个激活链接，当用户点击激活链接后，激活账号。**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687873289253/b95dd7a2-ae79-483b-bb77-38efc574dc70.png align="center")

**分析：**

1&gt;设计一个注册接口/regist，请求成功后模拟下发激活链接(链接本质是激活接口参数为：jwt)

2&gt;设计激活接口/active，接收参数为jwt

### 1.2 代码实现

**步骤1：创建项目：mail-active-demo**

**步骤2：导入相关依赖**

具体选哪个JWT工具包，可以看官网推荐：[https://jwt.io/libraries?language=Java](https://jwt.io/libraries?language=Java)

此处选择：jjwt

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.2.RELEASE</version>
    <relativePath/>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <scope>compile</scope>
    </dependency>

    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.11.5</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId> 
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

**步骤3：代码编写**

用户注册实体类

```java
package com.langfeiyes.mail.entity;

public class User {
    private Long id;
    private String username;
    private String password;
    private int state;


    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public int getState() {
        return state;
    }

    public void setState(int state) {
        this.state = state;
    }
}
```

JWT常量类：

```java
package com.langfeiyes.mail.util;

/**
 * 常量类
 */
public class JwtConstant {

    // 基本url
    public static final String BASE_DOMAIN_URL = "http://localhost:8080/";

    //jwt密码
    public static final String JWT_SECRET = "langfeiyesabcdefghijklmnopqrstuvwxyz11111111111";
    //jwt失效时间，单位秒
    public static final Long JWT_EXPIRATION = 24 * 60 * 60 * 1000L;
    //jwt 创建时间
    public static final String JWT_CREATE_TIME = "jwt_create_time";

    //jwt 用户信息-key
    public static final String USER_INFO_KEY = "user_info_key";
    //jwt 用户信息-id
    public static final String USER_INFO_ID = "user_info_id";
    //jwt 用户信息-username
    public static final String USER_INFO_USERNAME = "user_info_username";

}
```

JWT工具类:

```java
package com.langfeiyes.mail.util;

import com.langfeiyes.mail.entity.User;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * JwtToken工具类
 */
public class JwtTokenUtil {


    /**
     * 从数据声明生成令牌
     *
     * @param claims 数据声明
     * @return 令牌
     */
    public static String createToken(Map<String, Object> claims) {
        String token = Jwts.builder()
                //.setHeader(new HashMap<>())
                //.setAudience("Audience")
                //.setIssuer("Issuer")
                //.setSubject("Subject")
                //.setNotBefore(new Date())
                //.setIssuedAt(new Date())
                //.setId("jwt id")
                .setClaims(claims)//把荷载存储到里面
                .setExpiration(generateExpirationDate())//设置失效时间
                .signWith(Keys.hmacShaKeyFor(Decoders.BASE64.decode(JwtConstant.JWT_SECRET))) //签名
                .compact();
        return token;
    }

    /**
     * 从令牌中获取数据声明
     *
     * @param token 令牌
     * @return 数据声明
     */
    public static Claims parseToken(String token){
        Claims claims=null;
        try{
            claims = Jwts.parserBuilder()
                    .setSigningKey(Decoders.BASE64.decode(JwtConstant.JWT_SECRET))
                    .build()
                    .parseClaimsJws(token)
                    .getBody();
        }catch (Exception e){
            e.printStackTrace();
        }
        return claims;
    }

    /**
     * 生成token失效时间
     */
    private static Date generateExpirationDate() {
        //失效时间是当前系统的时间+我们在配置文件里定义的时间
        return new Date(System.currentTimeMillis()+JwtConstant.JWT_EXPIRATION);
    }

    /**
     * 根据token获取用户名
     */
    public static String getUserName(String token){
        Claims claims = parseToken(token);
        return getValue(claims, JwtConstant.USER_INFO_USERNAME);
    }

    /**
     * 验证token是否有效
     */
    public static boolean validateToken(String token){
        //claims 为null 意味着要门jwt被修改
        Claims claims = parseToken(token);
        return claims != null &&!isTokenExpired(token);
    }

    /**
     * 判断token是否已经失效
     * @param token
     * @return
     */
    public static boolean isTokenExpired(String token) {
        //先获取之前设置的token的失效时间
        Date expireDate=getExpiredDate(token);
        return expireDate.before(new Date()); //判断下，当前时间是都已经在expireDate之后
    }

    /**
     * 根据token获取失效时间
     * 也是先从token中获取荷载
     * 然后从荷载中拿到到设置的失效时间
     * @param token
     * @return
     */
    private static Date getExpiredDate(String token) {
        Claims claims=parseToken(token);
        return claims.getExpiration();
    }


    /**
     * 刷新我们的token：重新构建jwt
     */
    public static String refreshToken(String token){
        Claims claims=parseToken(token);
        claims.put(JwtConstant.JWT_CREATE_TIME,new Date());
        return createToken(claims);
    }

    /**
     * 根据身份信息获取键值
     *
     * @param claims 身份信息
     * @param key 键
     * @return 值
     */
    public static String getValue(Claims claims, String key){
        return claims.get(key) != null ? claims.get(key).toString():null;
    }
}
```

接口：

```java
package com.langfeiyes.mail.controller;

import com.langfeiyes.mail.util.JwtConstant;
import com.langfeiyes.mail.entity.User;
import com.langfeiyes.mail.util.JwtTokenUtil;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.Random;

@RestController
public class UserController {
    //模拟需要缓存的用户库
    //key: url, value: 要激活用户
    private static Map<String, User> map = new HashMap<>();

    @GetMapping("/regist")
    public String regist(User user){
        //假装成功
        System.out.println("注册成功");
        user.setId(new Random().nextLong());
        //创建jwt
        Map<String, Object> claims = new HashMap<>();
        claims.put(JwtConstant.USER_INFO_ID, user.getId());
        claims.put(JwtConstant.USER_INFO_USERNAME, user.getUsername());
        claims.put(JwtConstant.JWT_CREATE_TIME, new Date());
        String jwt = JwtTokenUtil.createToken(claims);

        //缓存jwt
        map.put(jwt, user);
        return JwtConstant.BASE_DOMAIN_URL + "/active?jwt=" + jwt;
    }


    @GetMapping("/active")
    public String active(String jwt){
        User user = map.get(jwt);
        if(user != null && JwtTokenUtil.validateToken(jwt)){
            map.remove(jwt);
            return "执行激活逻辑...";
        }else{
            return "参数不合法...";
        }
    }
}
```

**步骤4：启动项目**

```java
package com.langfeiyes.mail;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

**步骤5：测试**

浏览器发起2个请求：

注册

[http://localhost:8080/regist?username=dafei&password=666](http://localhost:8080/regist?username=dafei&password=666)

激活

[http://localhost:8080//active?jwt=eyJhbGciOiJIUzUxMiJ9.eyJjcmVhdGVfdGltZSI6MTY4MTYxNzA2MDg3NSwiaWQiOjEsInVzZXJuYW1lIjoiZGFmZWkiLCJleHAiOjE2ODE3MDM0NjB9.vQcsXUaEictz3QgjUBKwAV1qlou9yFCSMo4H6OaArz1ReEFzXt6klziHqonvsEfkv9aYdDc6G-vKVO9Zh1kcXw](http://localhost:8080//active?jwt=eyJhbGciOiJIUzUxMiJ9.eyJjcmVhdGVfdGltZSI6MTY4MTYxNzA2MDg3NSwiaWQiOjEsInVzZXJuYW1lIjoiZGFmZWkiLCJleHAiOjE2ODE3MDM0NjB9.vQcsXUaEictz3QgjUBKwAV1qlou9yFCSMo4H6OaArz1ReEFzXt6klziHqonvsEfkv9aYdDc6G-vKVO9Zh1kcXw)