---
title: "Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客"
source: "https://blog.csdn.net/qq_41965346/article/details/118760449"
author:
published:
created: 2025-04-09
description: "文章浏览阅读6.2w次，点赞174次，收藏630次。本文详细介绍了Modbus通信协议的基本概念，包括通信模式（并行与串行，异步与同步），单播与广播模式，Modbus基本概念及其在工业应用中的场景。重点解析了Modbus帧格式、寄存器类型与地址分配、功能码分类及部分功能码实例、异常码及其处理流程。此外，还涵盖了Modbus在串行传输（RTU、ASCII）和以太网（TCP/IP）中的应用。"
tags:
  - "clippings"
---
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_41965346/article/details/118760449)

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/6e12024dc0b13162d85323f69e63b481_MD5.png]]  

#### 0.1.1.1 Modbus

*   *   *   [0. 前言](#0__2)
        *   [1. 基本宏观概念（大方面，是什么，干啥用的）](#1__5)
        *   *   [1.1 通信模式分类](#11__19)
            *   *   [1.1.1 并行通信（数据各位同时传送）](#111__20)
                *   [1.1.2 串行通信（数据一位一位顺序依次传送）](#112__28)
                *   *   [1.1.2.1 异步串行通信（最常采用的通信方式）](#1121__37)
                    *   [1.2.2.2 同步串行通信](#1222__52)
            *   [1.2 Modbus 基本概念](#12_Modbus_70)
            *   [1.3 应用场景](#13__83)
        *   [2. 分解模块概念（细节步骤，包括什么东西，怎么定义和运作的）](#2__90)
        *   *   [2.1 单播模式与广播模式](#21__122)
            *   [2.2 Modbus 帧格式](#22_Modbus_127)
            *   [2.3 寄存器（数据的存储和读取）](#23__138)
            *   *   [2.3.1 寄存器种类](#231__140)
                *   [2.3.2 寄存器地址分配](#232__149)
            *   [2.4 功能码（主机发送的命令代码）](#24__162)
            *   *   [2.4.1 功能码分类](#241_163)
                *   [2.4.2 部分功能码举例](#242__194)
            *   [2.5 异常码（服务器或从站返回的异常代号）](#25__208)
            *   *   [2.5.1 通信状况](#251__210)
                *   [2.5.2 响应类型](#252__221)
                *   [2.5.3 异常码表](#253__259)
                *   [2.5.4 事务处理流程](#254__274)
            *   [2.6 数据帧格式](#26__280)
            *   *   [2.6.1 0x01 功能码（读线圈状态）](#261_0x01_282)
                *   [2.6.2 0x03 功能码（读保持寄存器）](#262_0x03_316)
                *   [2.6.3 0x0f 功能码（写多个线圈）](#263_0x0f_343)
                *   [2.6.4 0x10 功能码（写多个保持寄存器）](#264_0x10_379)
            *   [2.7 三种通信模式](#27__408)
            *   *   [2.7.1 串行传输模式（异步串行传输）](#271__410)
                *   [2.7.2 以太网（MODBUS TCP/IP）](#272_MODBUS_TCPIP_483)

#### 0.1.1.2 前言

> 最近学习 Modbus，本人 0 基础，学习之前都不知道 Modbus 是什么，以前从未接触过这个协议；看资料看了 4 天，CSDN 上的博文总是不尽人意、缺枝短叶，看完仍是一头雾水；后来结合着 Modbus 中文协议，终于算是有了比较透彻的理解。在此细心的整理了两天的学习笔记，给出了一个比较完整的学习流程和知识记录。

#### 0.1.1.3 基本宏观概念（大方面，是什么，干啥用的）

**目录**：

*   **1.1 通信模式分类**
    *   并行通信
    *   串行通信
*   **1.2 单播模式与广播模式**
*   **1.3Modbus 基本概念**
*   **1.4 应用场景**

##### 0.1.1.3.1 通信模式分类

###### 0.1.1.3.1.1 并行通信（数据各位同时传送）

> 一般快速设备之间采用并行通信，譬如 CPU 与存储设备、存储器与存储器、主机与打印机等都采用并行通讯。并行通讯，有多少位数据就必须有多少根数据线，如下图是 11 位数据就有 11 根数据线。

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/4edea1d92537898c9ae3f7c505013f67_MD5.png]]

*   **优点**：传输速度快
*   **缺点**：占用引脚资源多

###### 0.1.1.3.1.2 串行通信（数据一位一位顺序依次传送）

> 串行通信最少可以只需一根通信线，只发或只收。因而大大节省了系统资源，降低了系统成本。由于只用一根数据线，所以是以降低传送速度来换取资源的，它常用在传送距离远，速度要求不高的场合。

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/fd85716adfae8bfc0f91af67a6cefea1_MD5.png]]

*   **优点**：通信线路简单，占用引脚资源少，成本低
*   **缺点**：传输速度慢

###### 0.1.1.3.1.3 异步串行通信（最常采用的通信方式）

> 具有固定的通信格式，数据以相同的帧格式传送，每一帧由起始位、数据位、奇偶校验位和停止位组成。

1.  **起始位**：通信线路上没有数据传输的时候处于高电平（逻辑 “1“）状态，当发送设备发送一个字符数据时，先发送一个低电平（逻辑“0“）信号，告诉接收端” 开始发送数据了“，这个低电平就是一个起始位，接收端收到这个信息就准备接收信息。
2.  **数据位**：可以是 5 位、6 位、7 位、或 8 位。数据传送时，低位在前。
3.  **奇偶校验位**：用于数据传输过程的检错
    *   **奇校验**：保证数据位和校验位 “1” 的个数为奇数
    *   **偶校验**：保证数据位和校验位 “1” 的个数为偶数
    *   **无校验**：没有校验位，此时校验位用一个停止位补充，即有两个停止位
4.  **停止位**：停止位可以是 1 位、也可以是 1.5 位或 2 位。接收端收到停止位后，知道上一字符已传送完毕，同时，也为接收下一字符作好准备。若停止位后不是紧接着传送下一个字符，则让线路保持为 “1”。“1” 表示通信线路处于空闲等待状态。存在空闲位是异步通信的特性之一。

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/1f8742551ab6dcb2bb284e8f2a4cc3de_MD5.png]]

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/3b1535d83d87a47751f85f15031faeff_MD5.png]]

###### 0.1.1.3.1.4 同步串行通信

> 1.  通信双方共用一个时钟，这是同步通信区分于异步通信的最显著的特点，为了保证接收正确无误，发送方除了传送数据外，还要传送同步时钟。
> 2.  同步传输是以数据块为传输单位，在数据块传送时，为提高通信速度，去掉了起始位和停止位。数据开始传送前用同步字符来指示（常约定 1～2 个），并由时钟来实现发送端和接收端的同步，即检测到规定的同步字符后，下面就连续按顺序传送数据，直到一块数据传送完毕。
> 3.  数据块与数据块之间的时间间隔是固定的，且不允许有空位，当线路空闲或不发信息时，发送同步字符。
> 4.  同步传输中，发送方发出数据后等接收方发回响应以后才发下一个数据包。

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/94196b5c1f119056ecea54a4c7dbec39_MD5.jpg]]  
![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/282c8449d89cb0aa05d50c05f0925908_MD5.png]] 
**参考文章**

*   [同步串行与异步串行](https://blog.csdn.net/u012160319/article/details/43486995)
*   [串行通信原理及实验仿真](https://blog.csdn.net/gemengxia/article/details/108320356)
*   [UART 串口校验方式（无校验、奇偶校验、固定校验）](https://blog.csdn.net/Rocher_22/article/details/106550351)
*   [[计算机组成原理] 奇偶校验](https://blog.csdn.net/shiawaseli/article/details/98524134)

##### 0.1.1.3.2 Modbus 基本概念

> 1.  **Modbus** 是一种**串行[通信协议](https://baike.baidu.com/item/%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE)**，是 Modicon 公司（现在的[施耐德电气](https://baike.baidu.com/item/%E6%96%BD%E8%80%90%E5%BE%B7%E7%94%B5%E6%B0%94) SchneiderElectric）于 1979 年为使用[可编程逻辑控制器](https://baike.baidu.com/item/%E5%8F%AF%E7%BC%96%E7%A8%8B%E9%80%BB%E8%BE%91%E6%8E%A7%E5%88%B6%E5%99%A8)（PLC）通信而发表。Modbus 已经成为工业领域通信协议的业界标准（Defacto），并且现在是工业[电子设备](https://baike.baidu.com/item/%E7%94%B5%E5%AD%90%E8%AE%BE%E5%A4%87/4393826)之间常用的连接方式，是工业自动化领域里使用最普遍的协议。随着电子技术计算机和通信技术的不断发展, Modbus 也从最古老的 **RS232/Rs485 串行总线**发展到了 **ModbusTCP** 和**透明就绪工业以太网**。
> 2.  Modbus 是 **OSI 模型第 7 层**上的应用层报文传输协议，它在连接至不同类型总线或网络的设备之间提供客户机 / 服务器通信。
> 3.  从 1979 年开始，Modbus 作为工业串行链路的事实标准，Modbus 使成千上万的自动化设备能够通信。目前，对简单而精致的 Modbus 结构的支持仍在增长。互联网用户能够使用 TCP/P 栈上的保留系统端口 502 访问。
> 4.  ModbusModbus 是一个**请求 / 应答协议**，并且提供功能码规定的服务。 Modbus 功能码是 Modbus 请求 / 应答 PDU 的元素。
> 5.  Modbus 是一种**应用层报文传输协议**，用于在通过不同类型的总线或网络连接的设备之间的客户机 / 服务器通信。

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/3c6bbf973d684b7771f875b88f9839ee_MD5.png]] 
**参考：**

*   [【百度百科】](https://baike.baidu.com/item/Modbus%E9%80%9A%E8%AE%AF%E5%8D%8F%E8%AE%AE/5972462?fromtitle=ModBus&fromid=305501&fr=aladdin)

##### 0.1.1.3.3 应用场景

> 1.  工业、建筑、基础设施
> 2.  每种设备 (PC、HMI、控制面板、驱动器、运动控制、I/O 设备……) 都能使用 Modbus 协议来启动远程操作。同样的通信能够在基于串行链路和以太网 TCP/IP 网络上进行。网关能够实现在各种使用 Modbus 协议的总线或网络之间的通信。

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/4dfcee95002d449033d8478ed637b51e_MD5.png]]

#### 0.1.1.4 分解模块概念（细节步骤，包括什么东西，怎么定义和运作的）

> 通信就是交流，协议就是交流的规范或者语言。Modbus 简单来说就是主机和远程设备的交流语言。工业上，比如我想获得工厂温度、湿度信息，那我先通过传感器获得温度湿度数据，保存到记录里，我在办公室想知道这个信息，那么我只要保证三点：和工厂的控制设备连接、发送命令，控制设备返回响应，这三项满足，就能远程获取信息或者控制控制器了。

> 那么这里面包括了几点：数据的存储和读取（线圈和寄存器）、命令的种类（功能码）、数据的传输（通信模式、数据帧格式）、数据正确与否的检验（校验）、反馈或响应（异常码）等模块。

**目录**：

*   **2.1 单播模式与广播模式**
*   **2.2 Modbus 地址规则**
*   **2.3 Modbus 帧格式**
*   **2.4 寄存器**
    *   寄存器种类
    *   寄存器地址分配
*   **2.5 功能码**
    *   功能码分类
    *   部分功能码举例
*   **2.6 异常码**
    *   通信状况
    *   响应类型
    *   异常码表
    *   事务处理流程
*   **2.7 数据帧格式**
    *   0x01 功能码
    *   0x03 功能码
    *   0x0f 功能码
    *   0x10 功能码
*   **2.8 三种通信模式**
    *   串行传输模式：RTU、ASCII
    *   以太网（TCP/IP）

##### 0.1.1.4.1 单播模式与广播模式

> **单播模式**：“**一对一**” 通讯。主站只寻找某一确定的从站，从站接收到命令后处理，并返回一个应答报文。主站需要发出一个命令报名并处理从站返回的报文，从站需要接收主站的命令报文并发出一个应答报文。每个子节点必须有唯一的地址（1-247）。

> **广播模式：**“**一对所有**”。主站向所有从站发送请求，对于主站发送的广播请求没有应答返回，广播请求必须是写命令。所有从站必须接收写功能的广播。地址 0 用来广播通信。

##### 0.1.1.4.2 Modbus 帧格式

> Modbus 应用协议定义了一个独立于通信层的**协议数据单元** PDU（Protocol DataUnit），在不同的总线或者网络的 Modbus 协议同过在 PDU 上添加对应的附加域，构造出能用于当前通信的 ADU。

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/14d332c047258d846efa9b8bb48d9a25_MD5.png]] 
![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/8da486ea1562d4b3961e9d84bda21ec8_MD5.png]]

> Modbus 合法字节地址为十进制 0-247，0 为广播地址，每个子设备地址为 1-247。主节点将子节点地址填入地址域来寻找子节点，子节点将自己的地址填入来告知主节点是哪个子节点在应答。

1.  **功能码**表明了主节点要求子节点执行的操作。
2.  **数据部分**包括了请求时的具体细节或者返回时的数据细节。
3.  **差错校验**是串行传输中对地址域和 PDU 的冗余校验值，不同的传输模式采用不同的校验方式

##### 0.1.1.4.3 寄存器（数据的存储和读取）

###### 0.1.1.4.3.1 寄存器种类

<table><thead><tr><th align="center">寄存器种类</th><th align="center">说明</th><th align="center">PLC 类比</th><th align="center">举例</th></tr></thead><tbody><tr><td align="center">线圈状态</td><td align="center">输出端口。可设定端口的输出状态，也可以读取该位的输出状态。可分为两种不同的执行状态，例如保持型或边沿触发型。可读可写。</td><td align="center">DO 数字量输出</td><td align="center">电磁阀输出，MOSFET 输出，LED 显示等。</td></tr><tr><td align="center">离散输入状态</td><td align="center">输入端口。通过外部设定改变输入状态，可读但不可写。</td><td align="center">DI 数字量输入</td><td align="center">拨码开关，接近开关等。</td></tr><tr><td align="center">保持寄存器</td><td align="center">输出参数或保持参数，控制器运行时被设定的某些参数。可读可写。</td><td align="center">AO 模拟量输出</td><td align="center">模拟量输出设定值，PID 运行参数，变量阀输出大小，传感器报警上限下限。</td></tr><tr><td align="center">输入寄存器</td><td align="center">输入参数。控制器运行时从外部设备获得的参数。可读但不可写。</td><td align="center">AI 模拟量输入</td><td align="center">模拟量输入</td></tr></tbody></table>

###### 0.1.1.4.3.2 寄存器地址分配

> 寄存器 PLC 地址指存放于控制器中的地址，这些控制器可以是 PLC，也可以使触摸屏，或是文本显示器。PLC 地址一般采用 10 进制描述，共有 5 位，其中第一位代表寄存器类型。

> 寄存器协议地址指指通信时使用的寄存器地址，例如 PLC 地址 40001 对应寻址地址 0x0000，40002 对应寻址地址 0x0001，寄存器寻址地址一般使用 16 进制描述。例如，PLC 寄存器地址 40003 对应协议地址 0002，PLC 寄存器地址 30003 对应协议地址 0002，虽然两个 PLC 寄存器寄存器通信时使用相同的地址，但是需要使用不同的功能码访问，不同的功能码对应操作对应确定的 PLC 地址，所以访问时不存在冲突。

<table><thead><tr><th align="center">寄存器 PLC 地址</th><th align="center">寄存器协议地址</th><th align="center">适用的功能码（指令代号）</th><th align="center">寄存器种类</th><th align="center">读写状态</th></tr></thead><tbody><tr><td align="center">00001-09999</td><td align="center">0000-FFFF</td><td align="center">01H、05H、0FH</td><td align="center">线圈状态</td><td align="center">可读可写</td></tr><tr><td align="center">10001-19999</td><td align="center">0000-FFFF</td><td align="center">02H</td><td align="center">离散输入状态</td><td align="center">可读</td></tr><tr><td align="center">30001-39999</td><td align="center">0000-FFFF</td><td align="center">04H</td><td align="center">输入寄存器</td><td align="center">可读</td></tr><tr><td align="center">40001-49999</td><td align="center">0000-FFFF</td><td align="center">03H、06H、0FH</td><td align="center">保持寄存器</td><td align="center">可读可写</td></tr></tbody></table>

##### 0.1.1.4.4 功能码（主机发送的命令代码）

###### 0.1.1.4.4.1 功能码分类

> 功能码按操作对象可以分为两种: 位操作功能码（最小单位是 bit）和字操作功能码（最小单位为 2 字节）

​ **功能码按使用范围可以分为三种：**  
![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/b6a57ff8ac5371fd90d49b575a1fccac_MD5.png]]

**公共功能码：**

*   是较好地被定义的功能码
*   保证是唯一的
*   MODBUS 组织可改变的
*   公开证明的
*   具有可用的一致性调试
*   由 Modbus-IDA.org 确认的

**用户定义功能码**：

*   有两个用户定义功能码的定义范围，即 65-72 和十进制 100-110
*   用户没有 MODBUS 组织的任何批准就可以选择和实现一个功能码
*   不能保证被选功能码的使用是唯一的
*   如果用户要重新设置功能作为一个公共功能码，那么用户必须启动 RFC，以便将改变引入公共分类中，并且指配一个新的公共功能码。

**保留功能码**：

*   一些公司对传统产品通常使用的功能码，并且对公共使用是无效的功能码。

###### 0.1.1.4.4.2 部分功能码举例

<table><thead><tr><th align="center">功能码</th><th align="center">异常功能码 (+ 0×80)</th><th align="center">中文名称</th><th align="center">寄存器 PLC 地址</th><th align="center">位操作 / 字操作</th><th align="center">操作数量</th></tr></thead><tbody><tr><td align="center"><mark>0×01</mark></td><td align="center"><mark>0×81</mark></td><td align="center">读线圈状态</td><td align="center">00001-09999</td><td align="center">位操作</td><td align="center">单个或多个</td></tr><tr><td align="center">0×02</td><td align="center">0×82</td><td align="center">读离散输入状态</td><td align="center">10001-19999</td><td align="center">位操作</td><td align="center">单个或多个</td></tr><tr><td align="center"><mark>0×03</mark></td><td align="center"><mark>0×83</mark></td><td align="center">读保持寄存器</td><td align="center">40001-49999</td><td align="center">字操作</td><td align="center">单个或多个</td></tr><tr><td align="center">0×04</td><td align="center">0×84</td><td align="center">读输入寄存器</td><td align="center">30001-39999</td><td align="center">字操作</td><td align="center">单个或多个</td></tr><tr><td align="center">0×05</td><td align="center">0×85</td><td align="center">写单个线圈</td><td align="center">00001-09999</td><td align="center">位操作</td><td align="center">单个</td></tr><tr><td align="center">0×06</td><td align="center">0×86</td><td align="center">写单个保持寄存器</td><td align="center">40001-49999</td><td align="center">字操作</td><td align="center">单个</td></tr><tr><td align="center"><mark>0×0F</mark></td><td align="center"><mark>0×8F</mark></td><td align="center">写多个线圈</td><td align="center">00001-09999</td><td align="center">位操作</td><td align="center">多个</td></tr><tr><td align="center"><mark>0×10</mark></td><td align="center"><mark>0×90</mark></td><td align="center">写多个保持寄存器</td><td align="center">40001-49999</td><td align="center">字操作</td><td align="center">多个</td></tr></tbody></table>

##### 0.1.1.4.5 异常码（服务器或从站返回的异常代号）

###### 0.1.1.4.5.1 通信状况

​ **当主机向设备发送命令后，可能会出现以 4 下种情况：**

*   请求正确的到达服务器，并且请求的内容服务器可以处理，那么服务器返回一个正常响应
*   请求正确的到达服务器，但是请求服务器无法处理（例如请求读一个不存在的寄存器），此时服务器将返回一个异常响应，通知主机错误和错误的类型。
*   请求到达服务器，但是不正确，检测到了通信错误（奇偶校验、LRC、CRC 等等），那么不返回响应，主机将最终成为超时状态
*   请求就没到达服务器，服务器没收到也就更不会响应，主机也会成为超时状态

###### 0.1.1.4.5.2 响应类型

​ **综上，根据服务器处理结果，可以建立两种类型的响应：**

*   **一个正常 MODBUS 应答帧**：
    
    *   **功能码域**：响应功能码 = 请求功能码
        
    *   **数据域**：请求中要求的任何数据
        
    *   **校验码**：响应帧自身计算
        
        <table><thead><tr><th align="center">从站地址</th><th align="center">功能码</th><th align="center">数据</th><th align="center">校验码（低位）</th><th align="center">校验码（高位）</th></tr></thead><tbody><tr><td align="center">从站自身地址</td><td align="center">与请求功能码保持一致（范围：0x00-0x7f)</td><td align="center">请求中要求的任何数据</td><td align="center">XX</td><td align="center">XX</td></tr></tbody></table>
    
    ![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/5cdf065738b8d1a88f0c4defe4e2f586_MD5.png]]
    
*   **一个异常 MODBUS 应答帧**：
    
    *   用来为客户机提供处理过程中与被发现的差错相关的信息
        
    *   **功能码域**：响应功能码 = 请求功能码 + 0x80（见 2.4.2）
        
    *   **数据域**：提供一个异常码来指示差错原因
        
    *   **校验码**：响应帧自身计算
        
        <table><thead><tr><th align="center">从站地址</th><th align="center">功能码</th><th align="center">数据</th><th align="center">校验码（低位）</th><th align="center">校验码（高位）</th></tr></thead><tbody><tr><td align="center">从站自身地址</td><td align="center">请求功能码 + <mark>0x80</mark></td><td align="center">异常码</td><td align="center">XX</td><td align="center">XX</td></tr></tbody></table>
    
    ![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/50dd1283ffdf3c7dec911aeadcb19d9a_MD5.png]]
    

###### 0.1.1.4.5.3 异常码表

<table><thead><tr><th align="center">代码</th><th align="center">名称</th><th align="center">含义</th></tr></thead><tbody><tr><td align="center">01</td><td align="center">非法功能</td><td align="center">对于服务器 (或从站) 来说，询问中接收到的功能码是不可允许的操作。这也许是因为功能码仅仅适用于新设备而在被选单元中是不可实现的。同时，还指出服务器 (或从站) 在错误状态中处理这种请求，例如：因为它是未配置的，并且要求返回寄存器值。</td></tr><tr><td align="center">02</td><td align="center">非法数据地址</td><td align="center">对于服务器 (或从站) 来说，询问中接收到的数据地址是不可允许的地址。特别是，参考号和传输长度的组合是无效的。对于带有 100 个寄存器的控制器来说，带有偏移量 96 和长度 4 的请求会成功，带有偏移量 96 和长度 5 的请求将产生异常码 02。</td></tr><tr><td align="center">03</td><td align="center">非法数据值</td><td align="center">对于服务器 (或从站) 来说，询问中包括的值是不可允许的值。这个值指示了组合请求剩余结构中的故障，例如：隐含长度是不正确的。并不意味着，因为 MODBUS 协议不知道任何特殊寄存器的任何特殊值的重要意义，寄存器中被提交存储的数据项有一个应用程序期望之外的值。</td></tr><tr><td align="center">04</td><td align="center">从站设备故障</td><td align="center">当服务器 (或从站) 正在设法执行请求的操作时，产生不可重新获得的差错。</td></tr><tr><td align="center">05</td><td align="center">确认</td><td align="center">与编程命令一起使用。服务器 (或从站) 已经接受请求，并且正在处理这个请求，但是需要长的持续时间进行这些操作。返回这个响应防止在客户机 (或主站) 中发生超时错误。客户机 (或主站) 可以继续发送轮询程序完成报文来确定是否完成处理。</td></tr><tr><td align="center">06</td><td align="center">从属设备忙</td><td align="center">与编程命令一起使用。服务器 (或从站) 正在处理长持续时间的程序命令。当服务器 (或从站) 空闲时，用户 (或主站) 应该稍后重新传输报文。</td></tr><tr><td align="center">0A</td><td align="center">不可用网关路径</td><td align="center">与网关一起使用，指示网关不能为处理请求分配输入端口至输出端口的内部通信路径。通常意味着网关是错误配置的或过载的。</td></tr><tr><td align="center">0B</td><td align="center">网关目标设备响应失败</td><td align="center">与网关一起使用，指示没有从目标设备中获得响应。通常意味着设备未在网络中。</td></tr></tbody></table>

###### 0.1.1.4.5.4 事务处理流程

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/b89934c9979400a85a9f3ee5ca6a8a62_MD5.png]]

##### 0.1.1.4.6 数据帧格式

###### 0.1.1.4.6.1 0x01 功能码（读线圈状态）

*   **请求帧格式**
    
    <table><thead><tr><th align="center">从站地址</th><th align="center">功能码</th><th align="center">起始地址（高位）</th><th align="center">起始地址（低位）</th><th align="center">数量（高位）</th><th align="center">数量（低位）</th><th align="center">校验码（低位）</th><th align="center">校验码（高位）</th></tr></thead><tbody><tr><td align="center">0x03</td><td align="center">0x01</td><td align="center">0x00</td><td align="center">0x13</td><td align="center">0x00</td><td align="center">0x1B</td><td align="center">XX</td><td align="center">XX</td></tr></tbody></table>
    
    **含义：**
    
    **目标所在从站**：0x03（3 号）  
    **命令**：0x01（读线圈状态）  
    **寄存器种类**：线圈状态  
    **目标起始索引地址**：0x0013（索引地址 = 19）  
    **目标起始 PLC 地址**： 00001 + 19 = 00020（线圈状态 PLC 地址范围：00001-09999）  
    **读取线圈数量**：0x1B（1B = 27 个，即 27bit 数据）  
    **目标线圈范围**：00020 - 00046（从 00020 开始 27 个线圈）  
    **校验码**：XXXX
    
*   **正常应答帧格式**
    
    <table><thead><tr><th align="center">从站地址</th><th align="center">功能码</th><th align="center">字节计数</th><th align="center">字节一</th><th align="center">字节二</th><th align="center">字节三</th><th align="center">字节四</th><th align="center">校验码（低位）</th><th align="center">校验码（高位）</th></tr></thead><tbody><tr><td align="center">0x03</td><td align="center">0x01</td><td align="center">0x04</td><td align="center">0xCD</td><td align="center">0x6B</td><td align="center">0xB2</td><td align="center">0x05</td><td align="center">YY</td><td align="center">YY</td></tr></tbody></table>
    
    **含义：**
    
    返回从 3 号从站读取的共 4 个字节的数据，数据为：0xCD、0x6B、0xB2、0x05（为了举例子暂时编的），校验码为 YYYY。
    
    **补充：**
    
    由于读取的是线圈（bit)，若读取的个数不为 8 的倍数，比如这次读 27bit，则取整数字节 4 字节 32bit 返回，剩余 5bit 用 0 补全。
    

###### 0.1.1.4.6.2 0x03 功能码（读保持寄存器）

*   **请求帧格式**
    
    <table><thead><tr><th align="center">从站地址</th><th align="center">功能码</th><th align="center">起始地址（高位）</th><th align="center">起始地址（低位）</th><th align="center">数量（高位）</th><th align="center">数量（低位）</th><th align="center">校验码（低位）</th><th align="center">校验码（高位）</th></tr></thead><tbody><tr><td align="center">0x03</td><td align="center">0x03</td><td align="center">0x00</td><td align="center">0x06</td><td align="center">0x00</td><td align="center">0x02</td><td align="center">XX</td><td align="center">XX</td></tr></tbody></table>
    
    **含义：**
    
    **目标所在从站**：0x03（3 号）  
    **命令**：0x03（读保持寄存器）  
    **寄存器种类**：保持寄存器  
    **目标起始索引地址**：0x0006（索引地址 = 6）  
    **目标起始 PLC 地址**： 40001 + 6 = 40007（保持寄存器 PLC 地址范围：40001-49999）  
    **读取寄存器数量**：0x02（02 = 2 个，即 2×2byte = 4byte 数据）（一个寄存器为 2 字节）  
    **目标线圈范围**：40007-40008（从 40007 开始 2 个寄存器）  
    **校验码**：XXXX
    
*   **正常应答帧格式**
    
    <table><thead><tr><th align="center">从站地址</th><th align="center">功能码</th><th align="center">字节计数</th><th align="center">字节一（高位）</th><th align="center">字节一（低位）</th><th align="center">字节二（高位）</th><th align="center">字节二（低位）</th><th align="center">校验码（低位）</th><th align="center">校验码（高位）</th></tr></thead><tbody><tr><td align="center">0x03</td><td align="center">0x03</td><td align="center">0x02</td><td align="center">0xA1</td><td align="center">0x05</td><td align="center">0x04</td><td align="center">0xCD</td><td align="center">YY</td><td align="center">YY</td></tr></tbody></table>
    
    **含义：**
    
    返回从 3 号从站读取的共 2 个寄存器的数据，数据为：0xA105（40007 上的数据）、0x04CD（40008 上的数据）（为了举例子暂时编的），校验码为 YYYY。
    

###### 0.1.1.4.6.3 0x0f 功能码（写多个线圈）

*   **请求帧格式**
    
    <table><thead><tr><th align="center">从站地址</th><th align="center">功能码</th><th align="center">起始地址（高位）</th><th align="center">起始地址（低位）</th><th align="center">数量（高位）</th><th align="center">数量（低位）</th><th align="center">字节数</th><th align="center">字节一</th><th align="center">字节二</th><th align="center">字节三</th><th align="center">校验码（低位）</th><th align="center">校验码（高位）</th></tr></thead><tbody><tr><td align="center">0x01</td><td align="center">0x0f</td><td align="center">0x00</td><td align="center">0x13</td><td align="center">0x00</td><td align="center">0x15</td><td align="center">0x03</td><td align="center">0x12</td><td align="center">0x1A</td><td align="center">0x04</td><td align="center">XX</td><td align="center">XX</td></tr></tbody></table>
    
    **含义：**
    
    **目标所在从站**：0x01（1 号）  
    **命令**：0x0f（写多个线圈）  
    **寄存器种类**：线圈  
    **目标起始索引地址**：0x0013（索引地址 = 19）  
    **目标起始 PLC 地址**： 00001 + 19 = 00020（线圈状态 PLC 地址范围：00001-09999）  
    **写入线圈数量**：0x15（0x15 = 21bit 数据）  
    **实际写入线圈数量**：21bit + 3bit = 24bit = 3byte（只能  
    **目标线圈范围**：00020-00040（从 00020 开始 21 个线圈）  
    **字节一的值**：0x12（0x12 = 18 = 0001 0010）  
    **字节二的值**：0x1A（0x1A = 26 = 0001 1010）  
    **字节三的值**：0xAC（0x04 = 4 = 0000 0100）（最高三位为补 0）  
    **校验码**：XXXX
    
    **补充：**  
    若写入的线圈个数不为 8 的倍数，则高位补 0 使其字节数为整数。
    
*   **正常应答帧格式**（在原报文基础上除去字节数和具体字节并加上当前校验码）
    
    <table><thead><tr><th align="center">从站地址</th><th align="center">功能码</th><th align="center">起始地址（高位）</th><th align="center">起始地址（低位）</th><th align="center">数量（高位）</th><th align="center">数量（低位）</th><th align="center">校验码（低位）</th><th align="center">校验码（高位）</th></tr></thead><tbody><tr><td align="center">0x01</td><td align="center">0x0f</td><td align="center">0x00</td><td align="center">0x13</td><td align="center">0x00</td><td align="center">0x15</td><td align="center">YY</td><td align="center">YY</td></tr></tbody></table>
    
    **含义：**  
    向 1 号从站起始地址为 00020 处写入 21 个线圈的值成功，校验码为 YYYY。
    

###### 0.1.1.4.6.4 0x10 功能码（写多个保持寄存器）

*   **请求帧格式**
    
    <table><thead><tr><th align="center">从站地址</th><th align="center">功能码</th><th align="center">起始地址（高位）</th><th align="center">起始地址（低位）</th><th align="center">数量（高位）</th><th align="center">数量（低位）</th><th align="center">字节数</th><th align="center">字节</th><th align="center">校验码（低位）</th><th align="center">校验码（高位）</th></tr></thead><tbody><tr><td align="center">0x01</td><td align="center">0x10</td><td align="center">0x00</td><td align="center">0x53</td><td align="center">0x00</td><td align="center">0x02</td><td align="center">0x04</td><td align="center">0x13141A1B</td><td align="center">XX</td><td align="center">XX</td></tr></tbody></table>
    
    **含义：**
    
    **目标所在从站**：0x01（1 号）  
    **命令**：0x10（写多个保持寄存器）  
    **寄存器种类**：保持寄存器  
    **目标起始索引地址**：0x0053（索引地址 = 83）  
    **目标起始 PLC 地址**： 40001 + 83 = 40084（线圈状态 PLC 地址范围：40001-49999）  
    **写入寄存器数量**：0x02（02 = 2 个，即 2×2byte = 4byte 数据）（一个寄存器为 2 字节）  
    **目标寄存器范围**：40084-40085（从 40084 开始 2 个保持寄存器）  
    **写入字节数**：0x04（4 个）  
    **写入字节内容**：0x13141A1B（40084 保持寄存器写入 0x1314 = 0001 0011 0001 0100 ，40085 保持寄存器写入 0x1A1B = 0001 1010 0001 1011）  
    **校验码**：XXXX
    
*   **正常应答帧格式**（在原报文基础上除去字节数和具体字节并加上当前校验码）
    
    <table><thead><tr><th align="center">从站地址</th><th align="center">功能码</th><th align="center">起始地址（高位）</th><th align="center">起始地址（低位）</th><th align="center">数量（高位）</th><th align="center">数量（低位）</th><th align="center">校验码（低位）</th><th align="center">校验码（高位）</th></tr></thead><tbody><tr><td align="center">0x01</td><td align="center">0x10</td><td align="center">0x00</td><td align="center">0x53</td><td align="center">0x00</td><td align="center">0x02</td><td align="center">YY</td><td align="center">YY</td></tr></tbody></table>
    
    **含义：**  
    向 1 号从站的 40084 位置开始写入两个保持寄存器成功，校验码为 YYYY。
    

##### 0.1.1.4.7 三种通信模式

###### 0.1.1.4.7.1 串行传输模式（异步串行传输）

> 主从协议，位于 OSI 模型的第二层。由于没有冲突检测，为了防止混乱，只有一个主站，向 “从站” 发送命令并处理从节点的响应，从站接收主站的命令并做出响应；主站只能启动一个 Modbus 事务处理，从站没有收到主站的请求时不主动传输数据，也不与其他从站通信。

> 物理层上，最常用 **RS-485** 2 线制接口或 4 线制接口，当需要短距离的点到点通信时，也可以使用 **RS-232** 串行接口。

> 串行总线作为客户机，从站作为服务器。在串行链路上，所有设备的传输模式（及串行口参数）必须相同。所有设备必须实现 RTU 模式，ASCII 模式只是一个选项，默认模式必须是 RTU 模式。

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/0912dd485ff53e80e766ea6931f6b171_MD5.png]]

*   **RTU（远程终端单元）（采用 8bit 异步串行通信方式）（使用 CRC 循环冗余校验）**
    *   **编码系统**：8 位二进制，每个 8 位字节含有两个 4 位十六进制字符
        
    *   **字节格式（11 位）**
        
        *   **有校验**：1 个起始位，8 个数据位，1 个奇偶校验位，1 个停止位
        *   **无校验**：1 个起始位，8 个数据位，2 个停止位
    *   **串行发送字符**：从左到右：最低有效位（LSB）… 最高有效位（MSB）
        
    *   **校验（奇校验 ODD、偶校验 EVEN 和无校验 NONE）**
        
        *   默认校验模式必须是偶校验，为了保证和其他产品的兼容性，建议使用无校验。
        *   如果使用无校验，那么多附加一个停止位来满足定长 11 位异步字符。  
            ![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/4b73be742288c283019b7c4ebf0285a1_MD5.png]] 
            ![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/b157da61b7ca4e4813866e13a78eb10b_MD5.png]]
    *   **帧格式（最大 256 字节）**
        
        <table><thead><tr><th align="center"><mark>地址</mark></th><th align="center"><mark>功能码</mark></th><th align="center">数据</th><th align="center"><mark>CRC 校验</mark></th></tr></thead><tbody><tr><td align="center">1byte = 8bit</td><td align="center">1byte = 8bit</td><td align="center">Nbyte = N×8bit</td><td align="center">2byte = 16bit</td></tr></tbody></table>

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/f379ce470079c0edb54553076ac5b7ac_MD5.png]]

```
从站地址：1字节 = 8位（1byte = 8bit）
功能码：1字节 = 8位（1byte = 8bit）
数据：0-252字节
CRC循环冗余校验：2字节 = 16位（2byte = 16bit）
```

*   **帧判断**  
    **结束判断**：用时长至少为 3.5 个字符时间的空闲间隔将报文帧区分开，如果 3.5 个字符时间未接收到字符，则视为该帧结束。  
    ![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/33f6380dc1f98f84c15c63123ecb72aa_MD5.png]] 
    ![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/1452b9a1b58ef4565f4096af2f335ab4_MD5.png]] 
    **完整性判断**：用 1.5 个字符时间判断帧的完整性，如果两个字符之间的空闲间隔大于 1.5 个字符时间，那么认为报文帧不完整，接收站丢弃这个报文帧。  
    ![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/8bbd849cd489f481b068a984e19c2b3f_MD5.png]]
    
*   **帧校验（CRC 循环冗余校验，2 字节，检验整个报文内容）**  
    发送方计算 CRC 的值并附加到帧尾，接收报文的过程中，接收设备重新计算 CRC 的值，并将计算的结果和接收到的 CRC 比较；若不相等，则产生了错误。
    
*   **ASCII（使用 LRC 纵向冗余校验）**  
    用两个 ASCII 码字符发送报文中的一个 8 位字节，当通信链路或者设备不能满足 RTU 模式的定时管理要求时使用。
    
    *   **编码系统**：十六进制，ASCII 字符 0-9，A-F，报文中每个 ASCII 字符表示一个十六进制字符
    *   **字节格式（10 位）**
        *   有校验：1 个起始位，7 个数据位，1 个奇偶校验位，1 个停止位
        *   无校验：1 个起始位，7 个数据位，2 个停止位
    *   **串行发送字符**：从左到右：最低有效位（LSB）… 最高有效位（MSB）
    *   **校验（奇校验 ODD、偶校验 EVEN 和无校验 NONE）**
        *   默认校验模式必须是偶校验，为了保证和其他产品的兼容性，建议使用无校验。
        *   如果使用无校验，那么多附加一个停止位来满足定长 **10 位**异步字符。  
            ![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/a234ce94a9c7264cab611e2eae330692_MD5.png]] 
            ![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/4b420bf2f432aa3d817dbf089aecfc1b_MD5.png]]
    *   **帧格式（最大 513 字节）**<table><thead><tr><th align="center">起始</th><th align="center">地址</th><th align="center">功能码</th><th align="center">数据</th><th align="center">LRC 校验</th><th align="center">结束</th></tr></thead><tbody><tr><td align="center">1 字符’:'冒号</td><td align="center">2 字符</td><td align="center">2 字符</td><td align="center">0-2×252 字符</td><td align="center">2 字符</td><td align="center">2 字符（回车换行 CR，LF）</td></tr></tbody></table>

> 1.  一个报文必须以一个冒号（：）字符（十六进制 ASCII 3A）作为起始，以回车换行（CRLF）（十六进制 ASCII 0D 和 0A）作为结束。
> 2.  报文中字符间时间间隔不能超过 1s，除非配置了更长时间的超时，例如广域网应用可以要求 4-5s 超时，若超时，则表示已经出现错误。

*   **帧校验（LRC 纵向冗余校验）**
    *   校验内容不包括起始符和结束符
    *   对报文中除起始符和结束符外的所有连续的 8 位字节相加，忽略任何进位，然后求其二进制补码得到 LRC
    *   LRC 的结果也被编码为 2 个 ASCII 码字符。

###### 0.1.1.4.7.2 以太网（MODBUS TCP/IP）

**MODBUS TCP/IP 通信结构：**

> 串行链路上一个主站多个从站的模式演变为多个客户机和多个服务器的模式，IANA（Internet Assigned NumbersAuthority，互联网编号分配管理机构）给 Modbus 协议赋予 TCP 端口号为 502，ModbusTCP/IP 服务器端通常该端口作为接收报文的端口, 这是目前在仪表与自动化行业中唯一分配到的端口号。

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/293eec848c60df00fc57018f2a417380_MD5.png]]

**MODBUS TCP/IP 帧：**

> MODBUS 协议定义了一个与基础通信层无关的简单协议数据单元（PDU）。特定总线或网络上 的 MODBUS 协议映射能够在应用数据单元（ADU）上引入一些附加域。

> 在 TCP/IP 上使用一种专用报文头来识别 Modbus 应用数据单元 ADU，即 **MBAP 报文头**。

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/827d765700c1d656d5b5e2447eee016c_MD5.png]]

*   用 MBAP 报文头中的**单个字节单元标识符**取代 MODBUS 串行链路上通常使用的 MODBUS  
    **从站地址域**。这个单元标识符用于设备的通信，这些设备使用单个 IP 地址支持多个独立  
    MODBUS 终端单元，例如：网桥、路由器和网关。
*   用接收者可以验证完成报文的方式设计所有 MODBUS 请求和响应。对于 MODBUS PDU  
    有固定长度的功能码来说，仅功能码就足够了。对于在请求或响应中携带一个可变数据的功能码来说，数据域包括字节数。
*   当在 TCP 上携带 MODBUS 时，即使将报文分成多个信息包来传输，办事在 MBAP 报文头上携带**附加长度信息**，以便接收者能识别报文边界。显式和隐式长度规则的存在以及  
    **CRC-32 差错校验码**的使用（在以太网上）将对请求或响应报文产生极小的未检出干扰。

**MBAP 报文头格式（长 7 字节）：**

![[../_resources/Modbus通讯协议从一窍不通到原来如此_modbus从站主动上报-CSDN博客/ecb39e5545921efa39cfd76fab0a61b0_MD5.png]]

*   **事务元处理标识符**：用于事务处理配对。在响应中，MODBUS 服务器复制请求的事务处理标识符。
*   **协议标识符**：用于系统内的多路复用。通过值 0 识别 MODBUS 协议。
*   **长度**：长度域是下一个域的字节数，包括单元标识符和数据域。
*   **单元标识符**：为了系统内路由，使用这个域。专门用于通过以太网 TCP-IP 网络和 MODBUS 串 行链路之间的网关对 MODBUS 或 MODBUS + 串行链路从站的通信。MODBUS 客户机在请求中设置 这个域，在响应中服务器必须利用相同的值返回这个域。

**MODBUS TCP/IP 帧格式：**

> 协议数据单元前加 MBAP 报文头，没有了校验码，其他数据格式相同 请求帧格式：

<table><thead><tr><th align="center">事务元处理标识符（高位）</th><th align="center">事务元处理标识符（低位）</th><th align="center">协议标识符（高位）</th><th align="center">协议标识符（低位）</th><th align="center">长度（高位）</th><th align="center">长度（低位）</th><th align="center">单元标识符</th><th align="center">功能码</th><th align="center">起始地址（高位）</th><th align="center">起始地址（低位）</th><th align="center">寄存器数量（高位）</th><th align="center">寄存器数量（低位）</th></tr></thead><tbody><tr><td align="center">0x15</td><td align="center">0x01</td><td align="center">0x00</td><td align="center">0x00</td><td align="center">0x00</td><td align="center">0x06</td><td align="center">0xFF</td><td align="center">0x03</td><td align="center">0x00</td><td align="center">0x06</td><td align="center">0x00</td><td align="center">0x02</td></tr></tbody></table>

**正常应答帧格式**：

<table><thead><tr><th align="center">事务元处理标识符（高位）</th><th align="center">事务元处理标识符（低位）</th><th align="center">协议标识符（高位）</th><th align="center">协议标识符（低位）</th><th align="center">长度（高位）</th><th align="center">长度（低位）</th><th align="center">单元标识符</th><th align="center">功能码</th><th align="center">字节计数</th><th align="center">字节一（高位）</th><th align="center">字节一（低位）</th><th align="center">字节二（高位）</th><th align="center">字节二（低位）</th></tr></thead><tbody><tr><td align="center">0x15</td><td align="center">0x01</td><td align="center">0x00</td><td align="center">0x00</td><td align="center">0x00</td><td align="center">0x06</td><td align="center">0xFF</td><td align="center">0x03</td><td align="center">0x02</td><td align="center">0xA1</td><td align="center">0x05</td><td align="center">0x04</td><td align="center">0xCD</td></tr></tbody></table>

**参考文章：**

*   [MODBUS 协议整理——功能码简述](https://blog.csdn.net/iteye_3759/article/details/82548784)
*   [ModbusTcp 和 ModbusRtu](https://blog.csdn.net/weixin_42077793/article/details/118048327)
*   Modbus 协议中文版