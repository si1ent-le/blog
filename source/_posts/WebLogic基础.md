---
title: WebLogic基础
date: 2019-04-28 00:28:04
---
Weblogic是美国Oracle公司出品的一个application server，确切的说是一个基于JAVAEE架构的中间件，WebLogic是用于开发、集成、部署和管理大型分布式Web应用、网络应用和数据库应用的Java应用服务器。将Java的动态功能和Java Enterprise标准的安全性引入大型网络应用的开发、集成、部署和管理之中.

## 和Tomcat区别
1. Tomcat只能算是web container(web 容器)，是官方指定的jsp&servlet的容器，只实现了jsp/servlet的相关规范，不支持EJB.
2. eblogic是将j2ee的应用服务器（web container+EJB container，web容器和EJB容器融合在一起），包括ejb、jsp、servlet、jms等，属于全能型的.
3. Weblogic server凭借其出色的群集技术，拥有处理关键Web应用系统问题所需的性能、可扩展性和高可用性。Weblogic server既实现了网页群集，也实现了EJB组件 群集，而且不需要任何专门的硬件或操作系统支持。网页群集可以实现透明的复制、负载平衡以及表示内容容错 。无论是网页群集，还是组件群集，对于电子商务解决方案所要求的可扩展性和可用性都是至关重要的。共享的客户机/服务器和数据库连接以及数据缓存和EJB都增强了性能表现。这是其它Web应用系统所不具备的;所以,在扩展性方面WebLogic是远远超越了Tomcat.
4. 它们均免费试用次软件.

### 扩展EJB

EJB是Sun的JavaEE服务器端组件模型，企业级JavaBean（Enterprise JavaBean, EJB）设计目标与核心应用是部署分布式应用程序。简单来说就是把已经编写好的程序（即：类）打包放在服务器上执行。凭借java跨平台的优势，用EJB技术部署的分布式系统可以不限于特定的平台.

EJB 是为了"服务集群"和"企业级开发”所使用，那么，总得说说什么是所谓的"服务集群"和"企业级开发"吧！这个问题其实挺关键的，因为J2EE 中并没有说明白，也没有具体的指标或者事例告诉广大程序员什么时候用EJB 什么时候不用。于是大家都产生一些联想，认为EJB"分布式运算"指得是"负载均衡"提高系统的运行效率。然而，估计很多人都搞错了，这个"服务群集"和"分布式运算"并没有根本解决运行负载的问题，尤其是针对数据库的应用系统.
``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

