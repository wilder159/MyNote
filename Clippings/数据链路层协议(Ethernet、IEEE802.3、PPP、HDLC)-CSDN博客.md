---
title: "数据链路层协议(Ethernet、IEEE802.3、PPP、HDLC)-CSDN博客"
source: "https://blog.csdn.net/qq_36119192/article/details/84224111"
author:
published:
created: 2025-04-12
description: "文章浏览阅读4.1w次，点赞27次，收藏161次。目录数据链路层协议Ethernet以太网协议以太网数据帧的封装IEEE802.3协议PPP协议HDLC协议数据链路层协议首先Ethernet、IEEE802.3、PPP和HDLC都是数据链路层的协议，只不过后面三个不常用而已，数据链路层最常用的协议是Etnernet以太网协议。Ethernet和IEEE802.3属于以太链路层协议 广域网中经常会使用串行链路..._数据链路层协议"
tags:
  - "clippings"
---
[数据链路层协议](https://blog.csdn.net/qq_36119192/article/details/#t0)

[Ethernet以太网协议](https://blog.csdn.net/qq_36119192/article/details/#t1)

[以太网数据帧的封装](https://blog.csdn.net/qq_36119192/article/details/#t2)

[IEEE802.3协议](https://blog.csdn.net/qq_36119192/article/details/#t3)

[PPP协议](https://blog.csdn.net/qq_36119192/article/details/#t4)

[HDLC协议](https://blog.csdn.net/qq_36119192/article/details/#t5)

## 0.1 数据链路层协议

首先 Ethernet 、IEEE802.3、PPP和HDLC都是 [数据链路层](https://so.csdn.net/so/search?q=%E6%95%B0%E6%8D%AE%E9%93%BE%E8%B7%AF%E5%B1%82&spm=1001.2101.3001.7020) 的协议，只不过后面三个不常用而已，数据链路层最常用的协议是Etnernet以太网协议。

Ethernet和 IEEE802.3属于以太链路层协议

广域网中经常会使用串行链路来提供远距离的数据传输，高级数据链路控制HDLC（High-Level Data Link Control）和点对点协议PPP（ Point to Point Protocol）是两种典型的串口封装协议。

![](https://i-blog.csdnimg.cn/blog_migrate/59d3c8b356b07eb9428d3b323aab5e86.png)

### 0.1.1 Ethernet以太网协议

Ethernet以太网协议，用于实现链路层的数据传输和地址封装

#### 0.1.1.1 以太网数据帧的封装

![](https://i-blog.csdnimg.cn/blog_migrate/b55842637d52290ebe5d1797bd9318ad.png)

![](https://i-blog.csdnimg.cn/blog_migrate/b1855693f2ced8fae553221b124bb77f.png)

从上图可以看到 Ethernet II帧，目的地址、源地址字段各占6个字节，目的地址字段确定帧的接收者，源地址字段标识帧发送者。  
当使用六个字节的源地址字段时，前三个字节表示由 IEEE 分配给厂商的地址，将烧录在每一块网络接口卡的ROM中。而制造商通  
常为其每一网络接口卡分配后字节。其实目的、源地址就是我们经常说的MAC地址，比如00:1A:A0:31:39:D4就是一个MAC地址。  
类型字段，为2字节，用来标识上一层所使用的协议类型，如IP协议（0x0800）,ARP(0x0806)等。  
数据字段 以太网包最小规定为64字节，不足的也会填充到64字节。以太网包的最大长度是1518字节，数据字段长度范围为46到1500，  
这是为什么呢？因为以太网包最小规定为64字节，不足的也会填充到64字节。而以太网帧格式的其他部分加起来是6+6+2+4=18字节，  
所以数据部分的最小长度为64-18=46字节；而以太网包的最大长度是1518字节，因此1518-18=1500字节。  
FCS字段是帧校验字段，即Frame Check Sequence，用来保存CRC(循环冗余校验)校验值。

### 0.1.2 IEEE802.3协议

IEEE 802.3 通常指以太网，一种网络协议。描述物理层和数据链路层的MAC子层的实现方法，在多种物理媒体上以多种速率采用CSMA/CD访问方式  
MAC（MediaAccessControl）媒体访问控制层，该层定义了数据包怎样在介质上进行传输。  
LLC （LogicalLinks Control）逻辑链路控制层

![](https://i-blog.csdnimg.cn/blog_migrate/78c172b82b3afe61b119b64d8532bbb1.png)

### 0.1.3 PPP协议

[PPP协议](https://so.csdn.net/so/search?q=PPP%E5%8D%8F%E8%AE%AE&spm=1001.2101.3001.7020) 是一种点到点(一根链路两端只有两个接口)链路层协议，主要用于在全双工的同异步链路上进行点到点的数据传输。

![](https://i-blog.csdnimg.cn/blog_migrate/962c53af3c70e21e483992f3ccb2da23.png)

LCP是用来创建二层连接的，是有连接的(以太协议无连接)；NCP是用来实现三层通信的

![](https://i-blog.csdnimg.cn/blog_migrate/8f75ccd405b4b9013cdec19b4efeef63.png) ![](https://i-blog.csdnimg.cn/blog_migrate/25a3deeeb5f41b46dceb1bf110e3b338.png)

![](https://i-blog.csdnimg.cn/blog_migrate/f086de3bcc4326d6c1049dfb2d5ab25f.png)

### 0.1.4 HDLC协议

HDLC(High-level Data Link Control)，高级数据链路控制，简称HDLC，是一种面向比特的链路层协议，思科私有协议，现在几乎不用