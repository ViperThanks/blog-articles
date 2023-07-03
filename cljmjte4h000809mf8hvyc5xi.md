---
title: "从baidu.com到www.baidu.com的那些事"
datePublished: Wed Jun 14 2023 07:37:15 GMT+0000 (Coordinated Universal Time)
cuid: cljmjte4h000809mf8hvyc5xi
slug: baiducomwwwbaiducom
tags: dns, cdn

---

从baidu.com输入完，按回车，突然变成了www.baidu.com

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687873935138/0e2e91c4-50f0-4107-b92c-868372cdc5f5.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687873977884/37036e21-1362-4435-8100-ec9f567dfc6a.png align="center")

# 1\. 开胃小菜

## 介绍

首先介绍两个名词的意思

**DNS ：**

DNS中文叫 *域 名 系统* 它是用于将域名（例如 [*google.com*](http://google.com)）解析为对应的IP地址的系统。它充当了域名和IP地址之间的转换器。当你在浏览器中输入一个域名时，操作系统会向DNS服务器发送查询请求，获取域名对应的IP地址，然后使用该IP地址与服务器建立连接。**DNS的主要作用是将易记的域名映射到具体的IP地址，使用户能够方便地访问网站而不需要记住复杂的IP地址。**

![undefined](https://t1.gstatic.com/licensed-image?q=tbn:ANd9GcRk_qAQ9KVwmFOstDG7vOeTn_56P9Lw8XLZZBtNzQXtAdmzIktdTv7DbOyyM9M6z0sk align="left")

**CDN ：**

CDN 中文叫 *内容 分发 网络*

CDN是一种分布式网络架构，用于提供高效的内容传输和分发服务。CDN通过将内容存储在位于全球各地的多个服务器节点上，将内容就近提供给用户，以减少网络延迟和提高用户体验。**当用户请求访问某个网站的内容时，CDN会自动选择离用户最近的服务器节点，从该节点提供内容，而不是直接从源服务器获取。这样可以加快内容传输速度，减轻源服务器的负载压力，并提供更好的可扩展性和容错性。**

![Your Top 10 CDN Service Guide 2019 - Plesk](https://scdn1.plesk.com/wp-content/uploads/2019/04/25141138/image11.jpg align="left")

先说结论 :

## 结论

总结来说，DNS 用于将域名解析为 IP 地址，而 CDN 用于在全球分布的服务器节点上缓存和分发内容，提供更快速、可靠的内容传输。DNS 是一种系统，而 CDN 是一种基于网络架构的服务。它们在互联网中发挥着不同的作用，但通常可以结合使用，以提供更优化的用户体验。

# 2\. 实现整个流程

简洁点来说,就是我访问 [baidu.com](http://baidu.com) 的时候,

1. 操作系统先向我的本地 DNS 服务器查询该域名的地址
    
2. 如果缓存有,就直接走缓存，返回即可
    
3. 如果本地DNS服务器没有缓存，它会依次向上级域名服务器查询，包括根域名服务器、顶级域名服务器，直到找到能够解析 [`baidu.com`](http://baidu.com) 的DNS服务器
    
4. 如果 [baidu.com](http://baidu.com) 走 cdn 的话，则会在第三步返回遵循 cdn 的规则获取 cdn 的服务器 ip 并层层返回，返回到本地 DNS 服务器丢到缓存 (这样看来 [baidu.com](http://baidu.com) 应该是走 CName 了)
    
5. 接下来就是我电脑和 cdn 服务器的基于 tcp/ip 的 Https 请求了