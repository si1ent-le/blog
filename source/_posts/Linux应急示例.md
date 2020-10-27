---
title: Linux应急示例
date: 2019-12-06 10:51:25
tags:
 - 应急
---
🤦‍♂公司某测试服对外的Redis被黑导致被植入挖矿程序(kdevtmpfsi),所以就想着整理一份简单的Linux应急学习学习.

![](/images/yj/gj.png)

<center>图:ECS告警</center>

### 起因

ECS告警(如上图所示)某测试服,短信和控制台中提醒存在恶意程序(备注:上图只是存在恶意下载源,并不是真实恶意程序告警).因此赶紧进行应急处置(这里说下,告警要比应急早半天甚至一天时间,太滞后一定第一时间处置);原因是Redis对外服务未设置密码导致被攻击.

### 实施

#### 1.进程分析并kill

进程分析:可结合ECS告警获取父进程路径及命令详细信息,再进行ps获取具体进程及守护进程信息;这里主要确定病毒主程序对应PID.

![](/images/yj/rm.png)

<center>图:清理记录</center>

#### 2.ClamAV扫描

因以上清理是另一同事清理,后又收到短信提醒说存在恶意进程信息(可能未清理完),干脆进行病毒扫描看下.

工具使用参数介绍

```php+HTML
https://askubuntu.com/questions/250290/how-do-i-scan-for-viruses-with-clamav
```

![](/images/yj/av.png)

<center>图:ClamAV执行</center>

![](/images/yj/av1.png)

<center>图:扫描结果</center>

#### 3.在线监测

![](/images/yj/jc.png)

<center>图:检测结果</center>

#### 4.清理

定位恶意文件并清理内部其他恶意文件(red2.so、root这部分其实可对比新安装的Redis服务可快速判断恶意程序),以上检测是事后进行在线检测,第一时间先进行移除到本地留存.

![](/images/yj/redis.png)

<center>图:定位</center>

![](/images/yj/redis1.png)

<center>图:root文件内容</center>

#### 5.shell

根据root这个二进制文件提示下载地址下载shell分析可能存在系统部分命令替换、redis计划任务等信息.具体 [点击这里](http://www.si1ent.xyz/ziliao/shell.txt)

![](/images/yj/shell.png)

#### 6.计划任务清理

![](/images/yj/cron.png)

<center>图:Redis计划任务(ECS告警下载恶意源,目测服务已停)</center>

### 总结

以上是简单的挖矿病毒的清理,其实在进行应急时自然先确定告警事件类型(挖矿、勒索、挂马等),这样有助后面进行的一系列的应急实施.

加固建议的话也可自行网上搜索也可参考如下:

1、Redis禁止对外提供服务(配置文件修改并进行reload)

2、清理计划任务crontab -r(不同账户下可能存在不同的计划任务／var/spool/cron/)

3、更换一系列密码以防存在ssh免密登陆(从shell中得知重新写入key).

4、命令可参考如下.

### 命令

#### 进程分析基础命令

1.ps
参数:(进程分析)
-a //显示所有终端机下执行的进程
-u //以用户为主的格式来显示进程状况
-x //显示所有进程,不以终端机来区分
-e //此参数的效果和指定"A"参数相同
-f //显示UID,PPIP,C与STIME栏位

常用:
ps -aux
ps -ef

#### 网络监听

2.netstat
参数:(网络监听)
-a //(all)显示所有选项,默认不显示LISTEN相关
-t //(tcp)仅显示tcp相关选项
-u //(udp)仅显示udp相关选项
-n //拒绝显示别名,能显示数字的全部转化成数字(IP会直接显示)
-l //仅列出有在Listen (监听) 的服務状态
-p //显示建立相关链接的程序名
-r //显示路由信息,路由表
-e //显示扩展信息,例如uid等
-s //按各个协议进行统计
-c //每隔一个固定时间,执行该netstat命令

常用:
netstat -lut //显示处于监听状态的tcp、udp端口
netstat -lutp //带有PID显示网络件监听情况
netstat -tpln //带有IP显示网络监听情况
netstat -tulpan -c 3 //每3秒打印监听情况到屏幕上

#### 显示文件详情

3.stat
参数:(文件&目录信息)
(备:此命令可查看一些恶意程序会替换系统内命令如:wget、curl等具有下载功能)
-l //以ls -lT格式输出(文件权限、所属用户等信息)
其他参数基本不用即可获取文件&目录详细信息.