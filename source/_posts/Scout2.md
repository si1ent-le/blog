---
title: Scout2
date: 2020-05-23 02:27:09
---
Scout2 是一款用于 AWS 环境的安全审计工具.有了Scout2,不必人工浏览所有网页,就可以获取到 AWS 环境的配置数据;它甚至还能生成一份攻击面报告.Scout2 带有预配置的规则,并且很容易扩展以支持更多的服务和测试用例.因为 Scout2 仅通过 AWS的API去获取配置信息和发现安全隐患,所以无需提交 AWS 安全漏洞和渗透测试申请表.
## Scout 2使用
### 介绍
1. Scout2 是一款用于 AWS 环境的安全审计工具.有了Scout2,不必人工浏览所有网页,就可以获取到 AWS 环境的配置数据;它甚至还能生成一份攻击面报告.Scout2 带有预配置的规则,并且很容易扩展以支持更多的服务和测试用例.因为 Scout2 仅通过 AWS的API去获取配置信息和发现安全隐患,所以无需提交 AWS 安全漏洞和渗透测试申请表.
2. 有时候我们不经意间会将我们的某些端口或服务暴露给全世界,这样做的危害是什么呢?
    1. 暴露的数据:简单地让 S3 桶对世界开放可能对客户造成极大的破坏,因为他们的数据暴露在外并对公司的声誉造成不可挽回的损害.
    2. 暴露服务:安全组上的开放端口可能会使 MySQL 或 MongoDB 处于打开状态并且数据处于打开状态.
    3. 帐户泄露:AWS root或 IAM 用户的泄露凭据可能导致帐户完全失去控制权或大额帐单.
3. Scout2用Python编写的,运行版本为 2.7、3.3到3.6,需要AWS python库boto3.Scout2 根据GPL v2.0获得许可,可免费获得,并且拥有一组活跃的贡献者.目前，Scout2 会收集有关以下关键AWS服务的信息，并在本地生成的HTML报告中显示问题，并使用仪表板深入了解详细信息:
    1. EC2
    2. IAM
    3. RDS
    4. S3
    5. CloudTrail
    6. CloudFormation
    7. CloudWatch
    8. Route53
    9. SES
    10. SQS
    11. VPC
    12. SNS
    13. Redshift
    14. Elasticache
    15. EMR

## Scout2安装

``` bash
# 项目拉取
git clone https://github.com/nccgroup/Scout2
cd Scout2
# 环境配置
sudo pip install -r requirements.txt
# 安装
python setup.py install
Scout2 -h（如下图所示）
```
![](/images/scout2/image-20200523150151979.png)

## AWS凭证

``` bash
安装 Scout2 之后,需要确保拥有一些AWS凭证（密钥/令牌）,这些凭据允许对 Scout2 将要检查的 AWS 服务进行只读访问,需要将此访问策略授予将运行 Scout2 的用户或角色.
```
### 创建用户
在创建一个名为admin的用户，并将访问类型勾选为“编程访问”
如下图所示:
![](/images/scout2/image-20200523154146997.png)

### 新建组并赋予策略
![](/images/scout2/image-20200523154332264.png)
![](/images/scout2/image-20200523154352903.png)

### CVS 文件下载
点击“下一步”
![](/images/scout2/image-20200523154645948.png)

## Scout2 运行
``` bash
Scout2 --csv-credentials credentials.csv
```
### 选项
* profile：使用特定AWS配置文件的凭据

* regions：评估特定区域而不是所有区域的默认区域

* mfa-serial：允许使用需要MFA的凭据

* no-browser：完成运行后跳过打开浏览器

* report-dir：生成Scout2 HTML报告的路径

  例如:
```bash
scout2 ––profile credentials.csv ––regions us-west-2 ––no-browser
仅针对 “us-west-2” 中的资源运行它.完成后,它不会在浏览器中打开报告.
```
### 结果
![](/images/scout2/image-20200523155502048.png)
![](/images/scout2/image-20200523155544705.png)
仪表板根据评估状况显示不同的颜色,可以快速查看问题;该报告采用颜色编码.
* 绿色=good

* 黄色=warning

* 红色=bad

  

  它显示了为服务评估的资源数量,应用的规则以及结果和检查的完成情况.
  然后针对不同的种类,我们就可以再次点进去去查看具体的情况.
  Scout2 的输出易于查看和遍历,因为每个页面都使用颜色来突出显示问题,而大多数页面都是可以深入查看的仪表板.
## 参考
```bash
https://www.jianshu.com/p/838a7575c85f
https://github.com/nccgroup/Scout2
```
