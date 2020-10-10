#### 概述

强大的 MongoDB 审计和渗透性测试（pentesting）工具。用于审计 MongoDB 服务器，检测不良安全设置和执行自动渗透测试。

MongonDB配置安全检查工具

```
https://github.com/stampery/mongoaudit
```

#### Install

```shell
pip install mongoaudit
或如下安装
curl -s https://mongoaud.it/install | bash
```

![image-20200929211504801](/images/mongodb/image-20200929211504801.png)

#### Docker install mongodb

```shell
# 拉取镜像
docker pull mongo
# 运行并授权访问mongodb
docker run -itd --name mongo -p 27017:27017 mongo --auth
docker exec -it mongo mongo admin
# 创建一个名为 admin，密码为 123456 的用户。
> db.createUser({ user:'admin',pwd:'123456',roles:[ { role:'userAdminAnyDatabase', db: 'admin'},"readWriteAnyDatabase"]});
# 设置密码
> db.auth('admin', '123456')
```

#### 使用

1、安装无报错即可执行“`mongoaudit`”

![image-20200930103553876](/images/mongodb/image-20200930103553876.png)

2、点击“OK，I'll be careful”

![image-20200930111959162](/images/mongodb/image-20200930111959162.png)

<center>Mongoaudit scan types</center>

3、输入域名信息

![image-20200930112054225](/images/mongodb/image-20200930112054225.png)

4、输入域名及密码进行安全检查

```shell
mongodb://admin:123456@192.168.3.77:27017/database
```

![image-20200930115800598](/images/mongodb/image-20200930115800598.png)

5、检查结果

![image-20200930115946711](/images/mongodb/image-20200930115946711.png)
