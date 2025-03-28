---
title: "【STM32学习笔记】USART 硬件流控"
source: "https://zhuanlan.zhihu.com/p/89440015"
author:
  - "[[知乎专栏]]"
published:
created: 2025-03-17
description: "流控的概念源于 RS232 这个标准，在 RS232 标准里面包含了串口、流控的定义。大家一定了解，RS232 中的“RS”是Recommend Standard 的缩写，即”推荐标准“之意，它并不像 IEEE-1284、IEEE-1394 等标准，是由“委…"
tags:
  - "clippings"
---
**流控的概念源于 [RS232](https://zhida.zhihu.com/search?content_id=108004760&content_type=Article&match_order=1&q=RS232&zhida_source=entity) 这个标准，在 RS232 标准里面包含了串口、流控的定义。**大家一定了解，RS232 中的“RS”是Recommend Standard 的缩写，即”推荐标准“之意，它并不像 [IEEE-1284](https://zhida.zhihu.com/search?content_id=108004760&content_type=Article&match_order=1&q=IEEE-1284&zhida_source=entity)、[IEEE-1394](https://zhida.zhihu.com/search?content_id=108004760&content_type=Article&match_order=1&q=IEEE-1394&zhida_source=entity) 等标准，是由“委员会定制”。因而，不同的厂商在做 RS232 时，多少会有不同，流控也都会存在差异。以下我们与大家一起探讨流控的作用、搭建及如何操作。

**本文着重探讨硬件流控。**

  

**为什么需要流控？**

  

数据在两个串口之间进行通讯的时候常常会出现丢失数据的现象，比如两台计算机或者是一台计算机和一个单片机之间进行通讯，当接收端的数据缓冲区已经满了，这个时候如果还有数据发送过来，因为接收端没有时间进行处理，那这样的数据就有可能会丢失。在工业现场或者其他领域，经常会遇到这种问题，本质原因是速度不匹配、处理能力不匹配。比如单片机的主频只有20M或30M，ARM的处理能力可能是200M，PC机的处理能力是几个G，这种处理能力的不匹配造成了传输的时候数据容易丢失。

**硬件流控就是来解决这个速度匹配的问题。**它的基本含义非常简单，当接收端接收到的数据处理不过来时，就向发送端发送不再接收的信号，发送端接收到这个信号之后就会停止发送，直到收到可以继续发送的信号再继续发送。因此**流控本身是可以控制数据传输的进度，进而防止数据丢失。**

  

**一般常用的流控方式有两种：硬件流控和软件流控。**本文主要探讨硬件流控。

  

**如何在STM32上搭建硬件流控？**

![](https://pic3.zhimg.com/v2-5a70619cf0be732e9d32a5e5a7bfb63c_1440w.jpg)

▲　图1，硬件流控的连接原理图

  

图1中，以前用到的 TX 和 RX，也就是简单的三线串口的通讯方式，如果使能了硬件流控，在这个基础上需要增加两根控制线，一根叫 [CTS](https://zhida.zhihu.com/search?content_id=108004760&content_type=Article&match_order=1&q=CTS&zhida_source=entity)（Clear To Send 为输入信号，一根叫 [RTS](https://zhida.zhihu.com/search?content_id=108004760&content_type=Article&match_order=1&q=RTS&zhida_source=entity)（Require To Send 为输出信号）。其实从名字上也可以看到，一个是接收控制，一个是发送控制。

从硬件连接原理图中我们可以看到，如果从 [USART](https://zhida.zhihu.com/search?content_id=108004760&content_type=Article&match_order=1&q=USART&zhida_source=entity) 1 向 USART 2 发送的话，USART 1 的 TX 和 USART 2 的 RX 相连，USART 1 的 CTS 和 USART 2 的 RTS 相连，数据的方向是从 TX 到 RX，从串口1到串口2，流控是从 RTS 到 CTS 也就是从串口2到串口1。

  

**数据线方向与流控线数据方向相反**

  

从图1 - 硬件流控的连接原理图中，大家可以发现数据线方向与流控线数据方向是相反的，为什么呢？文章前面提到了流控的主要概念是指接收端没有时间处理这样的数据或者是处理能力比较弱，所以需要让发送端等待，接收端发出来的信号叫 RTS 信号，发送端检测管脚叫 CTS。因此，硬件连接原理图的下半部分和上半部分正好相反，接收端和串口2的TX相连，RTS和串口2的CTS相连。

  

**数据在接收的时候具体如何操作？**

![](https://picx.zhimg.com/v2-0ecdead04ed257fe2fba69629fe6c407_1440w.jpg)

▲　图2，接收与RTS信号原理图

  

从图2 - 接收与 RTS 信号原理图中，我们可以看到，RTS 信号在数据没有被读取之前都是保持在高电平状态，我们可以看到在 Start 之前都是高电平，这也就是告诉发送端，数据还没有被拿走，请发送端等待，一旦数据被 DMA 或者 CPU 从 DR 寄存器读取之后，RTS 就释放高电平，变为低电平，这时候发送端如果想发送数据的话就可以直接发送了。

一句话概括，就是 **RTS 表示了 USART 是否已经准备好接收新的数据了**。

另外，我们需要注意，当 USART 的 FIFO 模式也就是缓冲模式开启的时候，在 FIFO 满的时候才会去拉高 RTS 信号。

![](https://picx.zhimg.com/v2-1e705e5a441ad915dea5e2027fd25fc7_1440w.jpg)

▲　图3，发送与CTS信号原理图

  

图3 - 发送与 CTS 信号原理图中，TDR 是 USART 的发送寄存器，在这个寄存器中写入数据，如果这时候在移位寄存器中没有数据正在发送，硬件就会把 TDR 中的内容搬移到移位寄存器中，之后按照设置好的波特率、数据位等数据格式开始直接发送数据。这就是一个正常的数据发送的流程。

**如果使能了硬件流控的功能，就会增加一个实时检测的步骤。**在图3中，当没有收到CTS信号的时候，TX 发送线上数据是连续发送的，表现形式为：在 STOP 位后紧跟着就是下一个数据的 Start bit。

当 Data 2 还在 TX 线上进行发送的时候，如果此时在 CTS 信号上检测到了高电平，即使在 Data 2 的 STOP 位发送完之前写入了 Data 3，在当前的字节发送完之后是不会马上发送新写入的数据的，而是要等待，直到在 CTS 管脚上检测为低电平后，TX脚上才会开启 Data 3 的 Start 信号。

这里其实我们可以简单理解一下，**在发送的时候要实时监测 CTS 的电平状态，如果发现是高电平，就不会再发送新的数据**，直到 CTS 检测发现已经没有高电平信号了。

需要注意的是在当前字节发送完之前的三个时钟周期，CTS 需要提前置位上，也就是在Data 2 结尾的地方如果只差一个 STOP bit，那有可能把 Data 3 连续发送出去。

有人可能会有疑问，CTS 不是马上就置位了吗，而且 Data 2 还没有完全发送出去。其实它是去检查 CTS 的标志位，设置这个标志位至少需要两个时钟周期，设置好了 CTS 的标志位之后，硬件才会去检查进而不去发送 Data 3 的 Start bit。但如果设置的 CTS 或者是检查到的 CTS 已经是非常晚了，那后面的一个字节就已经发送过去了，因为在发送 Data 3 的时候没看到有 CTS 的标志位，所以就要求我们至少提前三个时钟周期把 RTS 信号释放出来，让 CTS 把这个信号检测到进而让后面的数据不再发送。RTS 是只要在接收缓冲区非空的时候就会被提前置位，也就是结果寄存器里面只要有一个东西就会把它置位，都会放在当前的移位缓冲寄存器里。

在原则上是不会出现由于 RTS 置位比较晚，导致 CTS比较慢的现象。但是不排除一种情况，就是 CTS 和 RTS 之间的延迟特别大，或者说串口的波特率特别快，这个时候就容易出现由于 RTS 置位比较晚使得 CTS 比较慢的现象。

  

**软件配置**

![](https://pic3.zhimg.com/v2-fcd558d5029ed426d8c75d300f108f1e_1440w.jpg)

▲　图4，软件配置

  

在 [CubeMX](https://zhida.zhihu.com/search?content_id=108004760&content_type=Article&match_order=1&q=CubeMX&zhida_source=entity) 里可以选择一个串口模式为异步模式，之后在它下面的硬件流控 RS232 中选择 CTS/RTS。这里要注意一下，CTS 和 RTS 是可以单独使能的，可以根据速度来选择使能 CTS 还是 RTS，如果我的速度比较慢的话就使能 RTS，因为 RTS 是给对方的信号，不需要考虑对方的处理能力。

另外，在 CubeMX 里也可以使能 [RS485](https://zhida.zhihu.com/search?content_id=108004760&content_type=Article&match_order=1&q=RS485&zhida_source=entity) 的硬件流控，这里的流控实际上流控的是数据的方向，因为 RS485 是一个半双工的通讯模式，它的数据收的时候就不能发，发的时候不能收。STM32 上有一个 DE 管脚和 RS485 的接收器芯片直接相连，控制数据的收发，所以我们要知道在 STM32 的硬件流控中其实包含两方面的内容，一方面是关于速度的，也就是 RS232 的 CTS、RTS；另一方面是关于数据的方向的控制，它是基于 RS485 的，在软件中只需要设置它的功能，其他使用功能和串口都是一样的。

  

**硬件流控和软件流控的区别**

  

**软件流控是以特殊的字符来代表从机已经不能再接收新的数据了**，基本的流程就是从机在接收数据很多的时候或主动给发送端发送一个特殊字符，当发送端接收到这个特殊字符后就不能再发送数据了。

软件流控很方便，不需要增加新的硬件，还是以前的TX、RX，但是使用了软件流控，它本身的字符也是数据，这个数据只不过是说在软件里把它设置了一个特殊的含义。如果它是一个全双工的通讯，在给另一个串口发送数据的时候如果也包含了这样一个特殊字符，对方就会误以为我让它不要再发送数据了，会有一定的概率出现错误，而硬件流控就不需要考虑这方面，只需要使用 CTS 和 RTS，所有的数据都是由硬件来操作的。

在实际的应用开发中，大家需要根据自己的实际情况来选择使用硬件流控还是软件流控。

![](https://pic3.zhimg.com/v2-7579ef727f7dc97619c4d3e358bc2c78_1440w.jpg)

**点击链接观看更多相关课程**

[电堂科技​c.51diantang.com/](https://link.zhihu.com/?target=https%3A//c.51diantang.com/)

发布于 2019-10-31 11:33

### 0.1.1 内容所属专栏

## 0.2 [

STM32学习分享

](https://www.zhihu.com/column/c_1139497340078583808)

工程师的随身学堂

[

单片机

](https://www.zhihu.com/topic/19577755)

[

STM32

](https://www.zhihu.com/topic/19855229)

[

嵌入式系统

](https://www.zhihu.com/topic/19565752)