---
title: Shiro漏洞学习
date: 2020-08-28 21:57:31
---

### 概述

Shiro是一个功能强大且易于使用的Java安全框架，可以执行身份验证、授权、加密和会话管理。使用Shiro易于理解的API，您可以快速且轻松地保护任何应用程序——从最小的移动应用程序到最大的web和企业应用程序。

#### Shiro主要功能

```
1、Authentication：身份认证
2、Authorization：权限校验
3、SessionManager：会话管理，用户从登录到退出是一次会话，所有的信息都保存在会话中。普通的java se环境中也支持这种会话。
4、cryptography：数据加密，如对用户密码进行加密，避免将密码明文存入数据库中。
5、Web support：非常容易集成到web环境中。
6、Caching：缓存，将用户信息和角色权限等缓存起来，不必每次去查
7、Concurrency：支持多线程，在一个线程中开启新的线程，能把权限传过去。
8、Testing：提供测试功能
9、Run As：允许一个用户假装为另一个用户的身份进行访问。
10、Remember me： 记住用户，一次登录后下次不用登录。
```

更多需要参考网络

### 环境搭建

拉取docker项目并运行

```shell
docker-compose up -d
```

![image-20200825170834627](/images/Shiro/image-20200825170834627.png)



### 漏洞实战分类

#### CVE-2016-4437

##### 漏洞描述

Apache Shiro 1.2.4及以前版本中，加密的用户信息序列化后存储在名为remember-me的Cookie中。攻击者可以使用Shiro的默认密钥伪造用户Cookie，触发Java反序列化漏洞，进而在目标机器上执行任意命令。

##### 影响范围

Apache Shiro  <=1.2.4

##### 漏洞原理

Apache Shiro默认使用了CookieRememberMeManager，其处理cookie的流程是：得到rememberMe的cookie值 > Base64解码–>AES解密–>反序列化。然而AES的密钥是硬编码的，就导致了攻击者可以构造恶意数据造成反序列化的RCE漏洞。

shiro反序列化的特征：在返回包的 Set-Cookie 中存在 rememberMe=deleteMe 字段

##### 漏洞条件

无

##### 漏洞检测

1、URL访问

```
http://192.168.3.219:8080/doLogin
```

![image-20200825170958258](/images/Shiro/image-20200825170958258.png)

2、用户登录

```
admin／vulhub
```

![image-20200825171101473](/images/Shiro/image-20200825171101473.png)



3、PoC获取

运行后如下图所示

![image-20200825171155065](/images/Shiro/image-20200825171155065.png)

##### 漏洞修复

更新至最新版本Shiro

### 参考

```
https://www.cnblogs.com/kyleinjava/p/10542800.html
https://github.com/vulhub/vulhub/tree/master/shiro/CVE-2016-4437
```