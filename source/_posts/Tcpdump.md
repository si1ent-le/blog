---
title: Tcpdump
date: 2020-07-16 22:11:39
---

tcpdump - dump traffic on a network

根据使用者的定义对网络上的数据包进行截获的包分析工具。 tcpdump可以将网络中传送的数据包的“头”完全截获下来提供分析。它支持针对网络层、协议、主机、网络或端口的过滤，并提供and、or、not等逻辑语句来帮助你去掉无用的信息。

tcpdump的报文获取功能是通过libpcap库实现。tcpdump需要root权限才可执行，不然攻击者有普通账户权限就能获取很多信息啊！

```shell
# 获取版本信息时会显示libpcap对应版本信息
# tcpdump --version
tcpdump version 4.9.2
libpcap version 1.7.4
OpenSSL 1.0.2g  1 Mar 2016
```

#### 原理

linux下的抓包是通过注册一种虚拟的`底层网络协议`来完成对网络报文(准确的说是网络设备)消息的处理权。当网卡接收到一个网络报文之后，它会遍历系统中所有已经注册的网络协议，例如以太网协议、x25协议处理模块来尝试进行报文的解析处理，这一点和一些文件系统的挂载相似，就是让系统中所有的已经注册的文件系统来进行尝试挂载，如果哪一个认为自己可以处理，那么就完成挂载。

当抓包模块把自己伪装成一个网络协议的时候，系统在收到报文的时候就会给这个伪协议一次机会，让它来对网卡收到的报文进行一次处理，此时该模块就会趁机对报文进行窥探，也就是把这个报文完完整整的复制一份，假装是自己接收到的报文，汇报给抓包模块。

#### 如何实现

包传递流程：网卡-->内存-->内核态-->用户程序；tcpdump程序运行在用户态，那如何实现从内核态的抓包？

![793dd841e89e5eb113c041860dfeea234153cfa4](/images/tcpdump/793dd841e89e5eb113c041860dfeea234153cfa4.png)

此时通过`libpcap库`来实现的，tcpdump调用libpcap的api函数，由libpcap进入到内核态到链路层来抓包,如下图：

BPF是过滤器，可以根据用户设置用于数据包过滤减少应用程序的数据包的包数和字节数从而提高性能。

BufferQ是缓存供应用程序读取的数据包；可以说tcpdump底层原理其实就是libpcap的实现原理。

![bc8a1dd22a5130474a0fa6945089fa2a4ccda680](/images/tcpdump/bc8a1dd22a5130474a0fa6945089fa2a4ccda680.png)

而libpcap在linux系统链路层中抓包是通过PF_PACKET套接字来实现的(不同的系统其实现机制是由差异的)，该方法在创建的时候，可以指定第二参数为SOCK_DGRAM或者SOCK_RAW，影响是否扣除链路层的首部。

libpcap在内核收发包的接口处将skb_clone()拿走的包。

#### libpcap

最早的编译系统和过滤引擎是在tcpdump项目中的，后来为了编译其他抓包的应用，将其独立出来。现在libpcap提供独立于平台的库和API，来满足执行网络嗅探。

tcpdump.c正式使用libpcap里的函数完成两个最关键的动作：获取捕获报文的接口，和捕获报文并将报文交给callback。

libpcap支持“伯克利包过滤（BPF）”语法。BPF能够通过比较第2、3、4层协议中各个数据字段值的方法对流量进行过滤。Libpcap的使用逻辑如下图:

![9fc92b7b41bb8d51fcb471f76ce6c3be3f564e2a](/images/tcpdump/9fc92b7b41bb8d51fcb471f76ce6c3be3f564e2a.png)

##### libpcap函数

函数分类：[tcpdmp_pcap官方](https://yq.aliyun.com/go/articleRenderRedirect?spm=a2c4e.11153940.0.0.4532acc0MF0CgG&url=http%3A%2F%2Fwww.tcpdump.org%2Fmanpages%2Fpcap.3pcap.html)

1、读包打开句柄

2、抓包选择链路层

3、抓包函数

4、过滤器

5、选定抓包方向（进还是出）

6、抓统计信息

7、将包写入文件打开句柄

8、写包

9、注入包

10、报告错误

11、获取库版本信息

##### 常见函数

- pcap_lookupdev      如果分组捕获设备未曾指定(-i命令行选项)，该函数选择一个设备。
- pcap_open_offine    打开一个保存的文件。
- pcap_setfilter            设置过滤器
- pcap_open_live         打开选择的设备。
- pcap_next                   接收一个包
- pcap_dump                将包写入到pcap_dump_t结构体
- pcap_loopupnet        返回分组捕获设备的网络地址和子网掩码，然后在调用pcap_compile时必须指定这个子网掩码。
- pcap_compile             把cmd字符数组中构造的过滤器字符串编译成一个过滤器程序，存放在fcode中。
- pcap_setfilter              把编译出来的过滤器程序装载到分组捕获设备，同时引发用该过滤器选取的分组的捕获。
- pcap_datalink              返回分组捕获设备的数据链路类型。

#### 协议注册

对于以上介绍的协议，也只有在需要的时候才会注册，因为它毕竟增加了系统报文的处理速度并且会消耗大量的系统skb。当抓包开始的时候，它会创建一个对应的网络套接口，这种套接口的类型就是`af_packet`类型。相关实现参考C源码：[af_packet.c](http://lxr.linux.no/linux+v3.4/net/packet/af_packet.c)。

````c
static int packet_create(struct socket *sock, int protocol)
{
    struct sock *sk;
    struct packet_opt *po;
    int err;
    if (!capable(CAP_NET_RAW))
        return -EPERM;
    if (sock->type != SOCK_DGRAM && sock->type != SOCK_RAW
#ifdef CONFIG_SOCK_PACKET
     && sock->type != SOCK_PACKET
#endif
     )
        return -ESOCKTNOSUPPORT;
    sock->state = SS_UNCONNECTED;
    err = -ENOBUFS;
    sk = sk_alloc(PF_PACKET, GFP_KERNEL, 1, NULL);
    if (sk == NULL)
        goto out;
    sock->ops = &packet_ops;
#ifdef CONFIG_SOCK_PACKET
    if (sock->type == SOCK_PACKET)
        sock->ops = &packet_ops_spkt;
#endif
    sock_init_data(sock,sk);
    sk_set_owner(sk, THIS_MODULE);

    po = sk->sk_protinfo = kmalloc(sizeof(*po), GFP_KERNEL);
    if (!po)
        goto out_free;
    memset(po, 0, sizeof(*po));
    sk->sk_family = PF_PACKET;
    po->num = protocol;

    sk->sk_destruct = packet_sock_destruct;
    atomic_inc(&packet_socks_nr);

    /*
     *    Attach a protocol block
     */

    spin_lock_init(&po->bind_lock);
    po->prot_hook.func = packet_rcv;       ----- 这个地方挂接处理函数，注册为packet_rcv。
#ifdef CONFIG_SOCK_PACKET
    if (sock->type == SOCK_PACKET)
        po->prot_hook.func = packet_rcv_spkt;
#endif
    po->prot_hook.af_packet_priv = sk;

    if (protocol) {
        po->prot_hook.type = protocol;
        dev_add_pack(&po->prot_hook);      ----- dev_add_pack将协议加入到ptype_all链表中，具体参考下面函数代码
        sock_hold(sk);
        po->running = 1;
    }
    write_lock_bh(&packet_sklist_lock);
    sk_add_node(sk, &packet_sklist);
    write_unlock_bh(&packet_sklist_lock);
    return(0);

out_free:
    sk_free(sk);
out:
    return err;
}
````

#### Options

```shell
# tcpdump -n -vvvv -i ens33 host 172.16.144.141 and port 80 -w web.pcap

-n：不转换主机名、端口号（开启后看到的是IP地址，而不是主机名，实际使用中我们一般都比较关注服务器IP地址）
-v：显示详细信息，v越多信息越多
-i：指定网络接口，也就网卡的名字，常用的有eth0,eth1等，如果要监听所有网卡就是用-i any。
-w：抓取的包写入到文件，方便后续分析。实际中经常使用tcpdump抓包保存，然后使用Wireshark分析
-r：抓到的包也可以tcpdump打开再分析，tcpdump -n -vvvv -r data.cap
-c：指定抓取的包的数目
-s：指定抓取的数据的长度
```

![image-20200716175428104](/images/tcpdump/image-20200716175428104.png)

Wireshark打开

![image-20200716184023410](/images/tcpdump/image-20200716184023410.png)

#### 常见命令

这部分简单演示了tcpdump的使用，如需常见语法可直接查看`基本语法`

```shell
# "GET"的十六进制是 47455420
# 监听对本地进行GET请求数据包
tcpdump -i ens33 'tcp[(tcp[12]>>2):4] = 0x47455420'
```

![image-20200716185547770](/images/tcpdump/image-20200716185547770.png)

```shell
# "SSH-"的十六进制是 0x5353482D
tcpdump -i ens33 'tcp[(tcp[12]>>2):4] = 0x5353482D'
```

![image-20200716184806562](/images/tcpdump/image-20200716184806562.png)

```shell
# 截获DNS请求数据
tcpdump -i ens33 udp dst port 53
```

```shell
# 截获大于600字节
tcpdump -i ens33 'ip[2:2] > 600'
```

#### 过滤器

- 过滤器的关键词有：host、port、 net、src、 dst、icmp、tcp、udp、http

- 逻辑关键词：and、or、not

- 包头过滤

  ````shell
  tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack != 0
  ````

#### 数据分析

```shell
tcpdump: listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
21:45:10.830435 IP (tos 0x0, ttl 64, id 63824, offset 0, flags [DF], proto TCP (6), length 60)
    18:54:59.000951 IP 172.16.144.1.64074 > 172.16.144.141.http: Flags [P.], seq 352511301:352511594, ack 1460171085, win 4117, options [nop,nop,TS val 872905073 ecr 10965627], length 293: HTTP: GET /robots.txt HTTP/1.1
```

`172.16.144.1.64074 > 172.16.144.141.http` 这里源地址(`172.16.144.1.64074`)到目的地址(`172.16.144.141.http`)

`Flags [S]`是包的标志，`[S]`表示是`SYC`:

```shell
[S] – SYN (开始连接)
[.] – 没有标记 (一般是确认)
[P] – PSH (数据推送)
[F] – FIN (结束连接)
[R] – RST (重启连接)
```

为了提高网络效率，一个包也可以包含标志的组合，比如`[S.]`，`[FP.]`

- `seq`：同步序列编号，Synchronize Sequence Numbers；
- `ack`：确认编号，Acknowledgement Number；
- `win`：滑动窗口大小
- `length`：承载的数据(payload)长度length，如果没有数据则为0

![1760830-e18fbb8480375fcd](/images/tcpdump/1760830-e18fbb8480375fcd.jpeg)

#### 基本语法

##### 1.过滤主机

抓取所有经过ens33，目的或源地址是172.16.144.141的网络数据

```shell
tcpdump -i ens33 host 172.16.144.141
```

指定源地址

```shell
tcpdump -i ens33 src host 172.16.144.141
```

指定目的地址

```shell
tcpdump -i ens33 dst host 172.16.144.1
```

##### 2.过滤端口

抓取所有经过ens33，目的或源端口是80的网络数据

```shell
tcpdump -i ens33 port 80
```

指定源端口

```shell
tcpdump -i ens33 src port 80
```

指定目的端口

```shell
tcpdump -i ens33 dst port 80
```

##### 3.网络过滤

```shell
tcpdump -i ens33 net 172.16
tcpdump -i ens33 src net 172.16
tcpdump -i ens33 dst net 172.16
```

##### 4.协议过滤

```shell
tcpdump -i ens33 arp
tcpdump -i ens33 ip
tcpdump -i eth1 tcp
tcpdump -i eth1 udp
tcpdump -i eth1 icmp
```

##### 5.表达式

```shell
非 : ! or "not" (去掉双引号)  
且 : && or "and"  
或 : || or "or"
```

抓取所有经过ens33，目的地址是172.16.144.141或172.16.141.1端口是80的TCP数据

```shell
tcpdump -i ens33 '((tcp) and (port 80) and ((dst host 172.16.144.141) or (dst host 172.16.141.1)))'
```

抓取所有经过ens33，目标MAC地址是00:0c:29:aa:b2:93的ICMP数据

```shell
tcpdump -i ens33 '((icmp) and ((ether dst host 00:0c:29:aa:b2:93)))'
```

抓取所有经过ens33，目的ip是172.16，但目的主机不是172.16.144.1的TCP数据

```shell
tcpdump -i ens33 '((tcp) and ((dst net 172.16) and (not dst host 172.16.144.141)))'
```

##### 6.运维

```shell
tcpdump -s 0 -w filename
-s 0是抓取完整数据包，否则默认只抓68字节
```

`-c`参数对于运维人员来说也比较常用，因为流量比较大的服务器，靠人工CTRL+C还是抓的太多，甚至导致服务器宕机，于是可以用`-c`参数指定抓多少个包。

```shell
tcpdump -nn -i ens33 'tcp[tcpflags] = tcp-syn' -c 10000
```

##### 7.高级

高级包头过滤可参考[这里](https://linuxwiki.github.io/NetTools/tcpdump.html)

#### 常见问题

- 如果数据包中出现很多的`cksum 0xxxx incorrect`错误：是因为操作系统为了提高网络效率不再计算校验码，而是交给网卡计算

#### 参考

```
Linux抓包原理:
https://www.iyunv.com/thread-149869-1-1.html
tcpdump源码分析:
https://yq.aliyun.com/articles/573120
tcpdump语法:
https://linuxwiki.github.io/NetTools/tcpdump.html
```

