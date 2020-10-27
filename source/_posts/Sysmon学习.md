---
title: Sysmon学习
date: 2020-10-22 17:44:08
tags:
 - Sysmon
---

#### Sysmon概述

Sysmon 是 Windows Sysinternals 系列中的一款工具。如果你想实时监控 Windows 系统又对其他第三方软件有顾虑，使用 Sysmon 这款轻量级 Microsoft 自带内部软件是最好的选择。

应用打开或任何进程创建行为发生时，Sysmon 会使用 sha1（默认），MD5，SHA256 或 IMPHASH 记录进程镜像文件的 hash 值，包含进程创建过程中的进程 GUID，每个事件中包含 session 的 GUID。除此之外记录磁盘和卷的读取请求 / 网络连接（包括每个连接的源进程，IP 地址，端口号，主机名和端口名），重要的是还可在生成初期进程事件能记录在复杂的内核模式运行的恶意软件。

项目地址：[Sysmon](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)

#### Sysmon原理

它通过系统服务和驱动程序实现记录进程创建、文件访问以及网络信息的记录，并把相关的信息写入并展示在windows的日志事件里。

更多参考：[Sysmon原理与分析](https://www.anquanke.com/post/id/156704)

##### Sysmon安装

[补丁]()

Sysmon安装在VM中时会出现以下报错，则补丁更新即可解决。

报错如下，说明系统缺少wevtapi的dll库文件，需更新`KB2533623`

````shell
error getting the evt dll (wevtapi.dll): 87
````

但出现以下报错，需更新此补丁`KB3033929`[补丁](https://www.microsoft.com/en-us/download/details.aspx?id=46148)

````shell
StartService failed for SysmonDrv:
````

安装方式一

````shell
Sysmon64.exe -i -h md5 -l -n
-i 即install
-n会监听network连接
sysmon有32位和64位两种，根据系统选择。
````

![image-20201016115903675](/images/Sysmon/image-20201016115903675.png)

安装方式二

网上提供对应配置文件

sysmon可以使用xml配置文件进行安装，需要先配置好xml文件，该配置文件为大家过滤了一些不必要监控的系统行为以及选择捕捉适当的条目，可以在应急响应中，将注意力集中真正有意义的日志上，并尽可能减少性能影响。

[配置文件](https://github.com/SwiftOnSecurity/sysmon-config)

```shell
Sysmon64 -accepteula -i sysmonconfig-export.xml
Sysmon64 -c sysmonconfig-export.xml
```

已经安装的话,无法再进行安装，可以使用`-c`参数进行更新安装方式即可。

![image-20201016135041924](/images/Sysmon/image-20201016135041924.png)

##### Sysmon卸载

````shell
# 通过以下方式对Sysmon进行卸载操作
Sysmon64.exe -u
````

![image-20201016135157467](/images/Sysmon/image-20201016135157467.png)

日志通过事件查看器查看，因为Sysmon的日志是以`evtx`格式存储。

```shell
eventvwr.msc
```

应用程序和服务日志-Microsoft-Windows-Sysmon-Operational

如同Windows自带的系统日志，安全日志有事件ID一样，sysmon日志也有对应的事件ID，最新版本支持24种事件。

```powershell
Event ID 1: Process creation
Event ID 2: A process changed a file creation time
Event ID 3: Network connection
Event ID 4: Sysmon service state changed
Event ID 5: Process terminated
Event ID 6: Driver loaded
Event ID 7: Image loaded
Event ID 8: CreateRemoteThread
Event ID 9: RawAccessRead
Event ID 10: ProcessAccess
Event ID 11: FileCreate
Event ID 12: RegistryEvent (Object create and delete)
Event ID 13: RegistryEvent (Value Set)
Event ID 14: RegistryEvent (Key and Value Rename)
Event ID 15: FileCreateStreamHash
Event ID 17: PipeEvent (Pipe Created)
Event ID 18: PipeEvent (Pipe Connected)
Event ID 19: WmiEvent (WmiEventFilter activity detected)
Event ID 20: WmiEvent (WmiEventConsumer activity detected)
Event ID 21: WmiEvent (WmiEventConsumerToFilter activity detected)
Event ID 22: DNSEvent (DNS query)
Event ID 23: FileDelete
Event ID 24: Clipboard Changed
Event ID 255: Error
```

##### 事件ID

###### 事件 ID 1

进程创建事件ID，恶意进程的创建，包括他的父进程，PID，执行命令及对应文件所在目录记录信息等等。

![image-20201016142613582](/images/Sysmon/image-20201016142613582.png)

###### 事件 ID 3

网络连接事件ID，当恶意程序外连CC服务器或者矿地址池等操作的时候，可监控到是哪个进程发起的连接，并找到对应程序所在目录进行清理操作。

![image-20201016143121787](/images/Sysmon/image-20201016143121787.png)

###### 事件 ID 11

文件创建事件ID，创建或覆盖文件时，这些创建操作会被记录下来。此事件对于监控自动启动位置，如启动文件夹目录、临时目录、下载目录非常有用，而这些目录正是初始感染阶段恶意运行要用到的目录。

![image-20201016154726431](/images/Sysmon/image-20201016154726431.png)

###### 事件 ID 15

创建命名文件流时记录。



###### 事件 ID 22

记录DNS查询，容易受该功能影响的一种场景就是基于DNS的C2通信，其中大量请求会被记录下来。

过滤Event ID 22，可以重点关注“DNS query”（DNS请求）事件。

![img](https://cdn.nlark.com/yuque/0/2019/png/302292/1563418822549-1c945bff-a194-4c71-b5d8-4f9c71169f7b.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_14%2Ctext_U3djaGFyeg%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

<center>图片来自"语雀"</center>
###### 事件 ID 23

文件删除记录，

Sysmon版本在12之后多了一个`事件 ID 24`用于记录剪切板事件。

###### 事件 ID 24

````
# 事件 ID 24
https://www.bleepingcomputer.com/news/microsoft/microsoft-sysmon-now-logs-data-copied-to-the-windows-clipboard/
````

![logged-event](/images/Sysmon/logged-event.jpg)

<center>图片来自以上链接</center>
#### Sysmon辅助

[Sysmon辅助工具](https://github.com/nshalabi/SysmonTools.git)

Sysmon View：Sysmon日志可视化

Sysmon Shell：Sysmon配置文件生成

Sysmon Box：Sysmon和网络捕获日志记录

##### Sysmon View

Sysmon View是Sysmon产生日志的可视化查看进程信息的辅助工具。

cmd下执行以下命令导出`xml`格式的日志文档，可通过`Sysmon View`辅助工具实现对其可视化显示。

````
WEVTUtil query-events "Microsoft-Windows-Sysmon/Operational" /format:xml /e:sysmonview > eventlog.xml
````

![image-20201017155404956](/images/Sysmon/image-20201017155404956.png)

![image-20201017155424676](/images/Sysmon/image-20201017155424676.png)

如软件进程存在关联性会显示关联关系。

![image-20201017155537676](/images/Sysmon/image-20201017155537676.png)

##### Sysmon Shell

Sysmon Shell是Sysmon用户自定义配置文档的GUI辅助工具。方便用户完成根据自己需求来自己书写自己的xml配置文档。

![image-20201017164457978](/images/Sysmon/image-20201017164457978.png)



##### Sysmon Box

可以通过Sysmon Box捕获sysmon和网络流量的数据库.

```
启动Sysmon Box使用如下命令：SysmonBox -in Wi-Fi
```

然后该工具将执行以下操作：

1) 开始捕获流量(在后台使用tshark，这就是你必须指定捕获接口的原因)，完成后点击CTRL + C结束会话

2) 然后，Sysmon Box将停止流量捕获，将所有捕获的数据包转储到文件，并使用EVT实用程序导出在会话的开始和结束时间之间记录的Sysmon日志

3) 使用来自Sysmon的导入日志和捕获的流量构建Sysmon View数据库(备份现有)文件，您所要做的就是从同一文件夹运行Sysmon视图或将数据库文件(SysmonViewDB)放在与Sysmon View相同的文件夹中(保持数据包捕获在同一位置)

#### Sysmon与SIEM系统

##### 背景

Windows操作系统因本身的安全特点经常成为内网安全渗透的首选目标，如果终端上没有安装EDR产品，基于Windows本身的日志很难监控到入侵行为。

因此需使用Sysmon及其他日志分析工具相结合来对系统或整个网络环境进行监控和分析。

##### 实战

[nxlog](https://nxlog.co/)

Nxlog主要作用是进行日志转换并发送至syslog服务器中的一款工具。

![image-20201020085647637](/images/Sysmon/image-20201020085647637.png)

配置安装路径

![image-20201020085722321](/images/Sysmon/image-20201020085722321.png)

![image-20201020085743662](/images/Sysmon/image-20201020085743662.png)

![image-20201020085801552](/images/Sysmon/image-20201020085801552.png)

![image-20201020085823438](/images/Sysmon/image-20201020085823438.png)

受限于Splunk与国内出口协议规定，目前无法下载。

后期再补上吧！！

#### Referer

```
https://cloud.tencent.com/developer/article/1041591
https://www.anquanke.com/post/id/156704
Sysmon DNS查询
https://www.anquanke.com/post/id/180418
报错解决：
https://superuser.com/questions/1482486/installation-error-of-sysmon-on-windows-7-vm-sysmondrv-driver-and-startservice
辅助工具：
https://www.yuque.com/p1ut0/xer98r/ugrtrf
Sysmon官方文档
https://docs.microsoft.com/zh-cn/sysinternals/downloads/sysmon
```

#### 实战参考

```
# Sysmon实战分析Word恶意文件
https://www.syspanda.com/index.php/2017/10/10/threat-hunting-sysmon-word-document-macro/
```

