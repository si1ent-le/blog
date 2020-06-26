---
title: 'Passionfruit:iOS应用黑盒评估工具'

date: 2020-06-01 11:34:27
---
虽没有 Android 平台那么多的攻击面和利用姿势,iOS 应用依然有安全审计的需求。移动平台的安全目前采用的策略基本上都是扫描器加上一部分人工的逆向和动态分析.

![](/images/passionfruit/log.png)

### 概述

一款名叫Passionfruit的iOS应用程序黑盒审计工具，该工具由[frida.re](https://www.frida.re/)和[vuejs](https://www.vuejs.org/)共同驱动.支持Web GUI界面操作,检测目标APP是否经过加密、可记录SQLite操作等共功能.

### 审计覆盖面

- 分析应用是否开启了必要的编译器保护
- 分析应用沙盒内的文件内容和权限
- 分析应用使用到的 framework 和动态链接库
- 分析应用存储的数据，如 UserDefaults, BinaryCookie 和 KeyChain
- 分析剪贴板的使用
- 动态拦截和分析 Objective C 运行时方法
- 动态拦截和分析本地代码的参数调用和堆栈追踪
- 分析 UIView 的层级结构和属性
- 一些基于 hook 实现的修改功能，如设备特征伪造、绕过越狱检测、绕过 SSL Pinning 等

![](/images/passionfruit/log1.png)

### 设计

在实现方案上,选择了功能极为强大的 hook 框架 [frida.re](http//www.frida.re/)。关于这个框架不需要我再过多介绍,它在 iOS 平台上支持对 native 函数、Objective C 运行时的 hook 和调用,可以满足多种移动安全运行时分析的自动化需求.

![](/images/passionfruit/log3.png)

Passionfruit 通过 frida 注入代码到目标应用实现功能,再通过 node.js 服务端消息代理与浏览器通信,用户通过访问网页即可对 App 实现常规的检测任务.

### 功能

1. 完整的网页版GUI操作界面；

2. 支持未越狱的iOS设备；

3. 屏幕截图；

4. 提供了可读的App元数据；

5. 可检测目标App是否经过加密，以及是否开启了PIE和ARC；

6. App沙盒文件浏览器，可直接预览设备图片、SQLite数据库以及plist文件，你还可以直接下载文件以便进行深入研究；

7. 检测已加载的框架，并挂钩本地原生函数；

8. 记录SQLite操作；

9. 记录并尝试绕过越狱检测；

10. 查看App中的Objective-C类、设置方法和函数钩子、拦截参数；

11. 读取keychain对象；

12. 读取cookie；

### 安装步骤

1. 安装 node.js,版本推荐使用 LTS

   ![](/images/passionfruit/log5.png)

2. 使用 npm 安装命令行工具:npm install -g passionfruit(建议使用cnpm进行安装)

3. 在越狱 iOS 设备上安装 frida[参考iOS脱壳部分]()

4. 执行 passionfruit 运行服务端

5. 在浏览器中访问 localhost:31337

```
sudo npm install -g cnpm --registry=https://registry.npm.taobao.org
```

```
sudo cnpm install -g passionfruit
```

![](/images/passionfruit/log6.png)

![](/images/passionfruit/log7.png)

手机USB链接电脑后访问会出现下图:

![](/images/passionfruit/log8.png)

### 安装时报错

在国内的网络可能会遇到软件源访问困难,导致无法完成安装的问题.

对于 npm，可使用淘宝的镜像服务和命令行工具 cnpm:[https://npm.taobao.org/](https://link.zhihu.com/?target=https%3A//npm.taobao.org/)

```
sudo npm install -g cnpm --registry=https://registry.npm.taobao.org
sudo cnpm install -g passionfruit
```

![](/images/passionfruit/log9.png)

即使使用了 cnpm，由于部分预编译包需要从 GitHub 或者 AWS 下载,也可能遇到网络障碍.

如有以下报错需更换镜像源

![](/images/passionfruit/log10.png)

### 注意

注：cnpm 存在一个 bug,如果上一次 cnpm install 被中断,第二次安装将会误认为安装成功而跳过应有的项目,导致实际上没有安装.解决方案是删除 node_modules 后重新安装.

如果删除node_modules目录将会删除npm并导致命令未找到,重新安装nodejs即可即可,然后再执行cnpm安装步骤.

### 参考

```bash
https://zhuanlan.zhihu.com/p/29761306
https://github.com/chaitin/passionfruit/wiki/%E4%BD%BF%E7%94%A8%E5%9B%BD%E5%86%85%E9%95%9C%E5%83%8F%E5%8A%A0%E9%80%9F%E5%AE%89%E8%A3%85
https://github.com/chaitin/passionfruit.git
```
