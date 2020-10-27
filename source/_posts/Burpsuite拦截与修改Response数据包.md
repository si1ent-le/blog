---
title: Burpsuite拦截与修改Response数据包
date: 2019-03-11 10:02:11
tag:
	- Burpsuite
---
测试中我们知道,通过Burpsuite可以完成客户端请求服务器端时对数据包拦截、重放以及修改等操作,但可能部分还不了解还可以修改返回数据包,而修改服务器端响应包的好处是什么呢?
下面我们尝试修改Burpsuite配置完成修改Response方式

### Burpsuite拦截
1.Burpsuite默认是拦截、修改Request数据包(也可以修改其他规则拦截其他类型数据)
![](/images/Burpsuite/bp1.png)
1.1.Request拦截配置(红色标记处默认两处已选择)
![](/images/Burpsuite/bp2.png)
1.2.通过修改proxy选项，option内修改Response
配置勾选如下图所示，新建规则/修改之前规则也可以.
![](/images/Burpsuite/bp3.png)

### 方法一

1.通过修改proxy选项，option内修改Response配置.
配置勾选如下图所示，新建规则/修改之前规则也可以.
![](/images/Burpsuite/bp4.png)
2.测试
![](/images/Burpsuite/bp5.png)

<center>Request请求数据包</center>

![](/images/Burpsuite/bp6.png)

<center>Response拦截</center> 

### 方法二
1.正常数据包拦截
![](/images/Burpsuite/bp7.png)

<center>点击后即可完成此请求的服务器返回数据包拦截</center> 

  ![](/images/Burpsuite/bp8.png)

 <center>Response拦截成功</center> 

### 实战
1.用户注册
```BASH
username:si1entt
passwd:si1enta123
```
![](/images/Burpsuite/bp9.png)
2.手机号绑定
业务逻辑：输入正确手机号并获取验证码输入后方可验证通过
![](/images/Burpsuite/bp10.png)
3.此时我们尝试输入非使用手机号绕过验证
随机输入手机号、手机号验证码、以及校验码
![](/images/Burpsuite/bp11.png)
![](/images/Burpsuite/bp12.png)

<center>Response数据包获取</center> 

4.修改response数据，false为true并重放
![](/images/Burpsuite/bp13.png)
![](/images/Burpsuite/bp14.png)
5.用户登陆查看数据绑定手机号信息
![](/images/Burpsuite/bp15.png)

### 结束
Burpsuite功能点还有很多需要测试、学习.