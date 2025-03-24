---
title: "以太网技术详解-CSDN博客"
source: "https://blog.csdn.net/q1594/article/details/126584325"
author:
published:
created: 2025-02-24
description: "文章浏览阅读5.2k次，点赞2次，收藏34次。以太网（Ethernet）是当今局域网采用的最通用的通信协议标准，该标准定义了在局域网中采用的电缆类型和信号处理方法。以太网凭借其成本低、通信速率高、抗干扰性强等优点被广泛运用。以太网分为标准以太网（10Mbit/s）,快速以太网（100Mbit/s）和千兆以太网。随着以太网技术的发展，也产生了万兆以太网（10Gbit/s）。..._千兆网口协议"
tags:
  - "clippings"
---
## 0.1 千兆以太网协议简介

最新推荐文章于 2024-12-11 09:44:15 发布

![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[momo5234](https://blog.csdn.net/q1594 "momo5234") ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCurrentTime2.png) 于 2022-08-29 14:48:52 发布

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

## 0.2 1.以太网概述

  以太网（Ethernet）是当今局域网采用的最通用的通信协议标准，该标准定义了在局域网中采用的电缆类型和信号处理方法。以太网凭借其成本低、通信速率高、抗干扰性强等优点被广泛运用。  
  以太网分为标准以太网（10Mbit/s）,快速以太网（100Mbit/s）和千兆以太网。随着以太网技术的发展，也产生了万兆以太网（10Gbit/s）。

## 0.3 2.以太网接口

  以太网接口有RJ45、RJ11接口（电话线接口）、SC光纤接口等。RJ45是最常见的接口。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/01cbc1ac6aa9057ca11cd8ad6d8644b9.png)  
  在不同的速率下接口的作用也不同，10M/100M速率下RJ45接口只用了四线，1000M下用了八线。千兆以太网接口向下兼容。具体如下图  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6492f551396275c0895f94a943447955.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2a04257d01a3b5ee70307aa10aad43a9.png)  
  以太网接口电路主要由MAC（Media Access Control）控制器和物理层接口PHY（Physical Layer，PHY）两大部分构成。这两部分既可整合到一颗芯片内，也可独立分开。对于本次设计来说，MAC控制器由FPGA实现，PHY芯片开发板板载。  
  VSC8601设备包含一个IEEE802.3兼容的串行管理接口（SMI）,其中MDC和MDIO可对芯片进行控制。SMI提供了访问权限设备控制和状态寄存器。控制SMI的寄存器由32组成16位寄存器，包括所有需要的IEEE指定寄存器。此外，有通过设备寄存器31可以访问的寄存器的附加寄存器。  
  PHY芯片指开发板板载的以太网芯片。PHY接收MAC发送过来的数据，把并行数据转化为串行数据，按照物理层的编码规则把数据编码转化为模拟信号发送出去，接收数据时流程反之。  
  PHY还提供了和对端设备连接的重要功能，并通过LED灯显示连接状态和工作状态。还可互相协商连接速度、双工或者半双工、是否采用流控等。通常结果是采用双方能同时支持的最大速度和最好的双工。这种技术被称为自协商。

### 0.3.1 MDIO接口

  MAC和PHY芯片有一个配置接口，即MDIO接口。可以配置PHY芯片的工作模式以及获取PHY芯片的状态信息。PHY芯片内部有一系列寄存器。用户通过配置寄存器来配置PHY芯片的工作模式。  
  FPGA通过MDIO接口对PHY芯片的内部寄存器进行配置。通常情况下芯片在默认情况下也可以工作，即配置芯片不是必须的。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/78456f91244a7582f4740441b188f57f.png)

## 0.4 [RGMII](https://so.csdn.net/so/search?q=RGMII&spm=1001.2101.3001.7020)和RMII接口

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/88b1807672f5a6ce61df2f9e4ab2ffe3.png)

  RGMII接口是双边沿发送数据，一个[时钟周期](https://so.csdn.net/so/search?q=%E6%97%B6%E9%92%9F%E5%91%A8%E6%9C%9F&spm=1001.2101.3001.7020)可以发送8bit，但是在FPGA内部一般采用上升沿处理数据，所以要将双边沿信号RGMII转化成单边沿信号GMII。