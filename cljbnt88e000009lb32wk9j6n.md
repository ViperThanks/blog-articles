---
title: "Vps 折腾指北"
datePublished: Thu Jun 01 2023 12:09:17 GMT+0000 (Coordinated Universal Time)
cuid: cljbnt88e000009lb32wk9j6n
slug: vps
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1687710677746/2d0f81de-0f3a-4ae3-8c7a-6ae9182f9e41.png
tags: dns, cdn, ubuntu, docker, vps

---

# 域名

## 购买

这里推荐 NameSilo [网址](https://www.namesilo.com) 如果你有 Github 学生包, [name](https://www.name.com/) [会送你一个免费的自选的.live](http://xn--4gqvdo6a9h58fh53ga754unixklgzc.live) 的网址, 这里我白嫖了个 [laoda.live](http://laoda.live) 的

## DNS manager (管理你的DNS)

### 前言 (废话)

以下默认你的 vps 的 ipv4 addr 为 10.10.10.10

不可能买了一个域名, 就可以和你的 VPS 的 IP 就对上了的, 还是需要经过一些处理, 这里以我白嫖的 [laoda.live](http://laoda.live) 域名和我 $25 买的 GreenCloud JP 来实验

先 ping 一下看看能不能 ping 到你想要的网址 这里是我之前玩的 DigitalOcean 过期了, 所以请求超时

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687710762139/6a720748-49be-4d88-83b9-29b567e9bab1.png align="center")

### 实际操作

#### 步骤一, 选择 Manage DNS Records

Java 人的面经 -&gt; DNS 和 CDN 的区别, 他们分别能干嘛? 不懂的小伙伴自己去GPT

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687710797747/244e7f1b-004b-446f-b0d7-a893734d1526.png align="center")

#### 步骤二, 映射你的域名 到你的 vps 服务器

先 TYPE 选 A , HOST 填@代表根目录或者域名本身, ANSWER 填你的 vps 的 ipv4 addr (*for example : 10.10.10.10*)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687710826121/07f02495-aff7-476a-85b1-48b3bee067f4.png align="center")

大功告成!!!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687710943347/5fb449e2-df0d-4666-9912-ae34aa98d7f4.png align="center")

以下是各种 DNS 记录类型的描述：

| 记录类型 | 描述 |
| --- | --- |
| A | 将域名映射到 IPv 4 地址 |
| CNAME | 为一个域名创建别名，指向另一个域名 |
| TXT | 存储与域名相关的文本信息，通常用于验证、配置或说明 |
| SRV | 指定提供特定服务的服务器的位置和端口信息 |
| AAAA | 将域名映射到 IPv 6 地址 |
| NS | 指定用于特定域名的域名服务器 |
| ANAME | 提供动态解析功能，类似于 CNAME，但根据查询的 IP 地址返回结果 |

#### 步骤三, 去 pull 个 docker image 并且 run

这里我选择 Nginx 然后端口映射 8888: 80 (Nignx 默认 80) 这句话的意思是本机的 8888 端口映射到 container 的 80 端口上

浏览器能访问就行了

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687868534228/08f6b7b2-ad6b-4b0e-8b4c-21148fb66923.png align="center")

### 把解析丢给 CF -&gt; CDN 加速

由于 DigitalOcean 的新加坡路线不容乐观

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687710969769/33339cbc-12de-4e5a-8635-32d6e1bb8488.png align="center")

丢给 CDN 应该没问题

#### 步骤一:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687710976628/7d03d5ce-df25-4333-90f0-9b3e4ead76ac.png align="center")

#### 步骤二修改所有的 nameserver 改成 cf的

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687710988526/43d288f1-b59c-4fb5-9c71-610da6f6274e.png align="center")

[`dan.ns.cloudflare.com`](http://dan.ns.cloudflare.com)

[`lara.ns.cloudflare.com`](http://lara.ns.cloudflare.com)

变成这样就行了

**<mark>Warning: 如果这样操作，你就不能用name.com 的 DNS了，不知道为什么，它是这样提示的，到时候我去研究一下</mark>**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687710998938/f5ddc9ec-a171-42fd-b1ca-1e9e4b59c93d.png align="center")

# 面板

推荐 aaPanel 就是宝塔的外国版

Ubuntu 命令

```bash
wget -O install.sh http://www.aapanel.com/script/install-ubuntu_6.0_en.sh && sudo bash install.sh aapanel
```

[常见命令](https://www.aapanel.com/new/download.html#install)

看到下面这幅图就是赢麻了

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687711079069/befd59c5-8855-4f23-b510-ba3c5d96e6f1.png align="center")

然后打开就行了