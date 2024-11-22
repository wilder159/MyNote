> 📌 原文链接：[https://blog.csdn.net/...](https://blog.csdn.net/qq_25218501/article/details/131546655) 🕘 收藏时间：2024年07月04日 📂 文档目录：**我的云文档/应用/网页收藏** 📑 本文档由[【金山收藏助手】](https://kdocs.cn/l/cpRidR7kBnn3)一键生成

**目录**

[DMA-直接存储器访问控制器](https://blog.csdn.net/qq_25218501/article/details/#t0)

[DMA概览](https://blog.csdn.net/qq_25218501/article/details/#t1)

[DMA的作用](https://blog.csdn.net/qq_25218501/article/details/#t2)

[DMA框图](https://blog.csdn.net/qq_25218501/article/details/#t3)

[DMA外设要点概括](https://blog.csdn.net/qq_25218501/article/details/#t4)

[DMA功能对比](https://blog.csdn.net/qq_25218501/article/details/#t5)

[STMF10x DMA具体内容](https://blog.csdn.net/qq_25218501/article/details/#t6)

[DMA主要特性](https://blog.csdn.net/qq_25218501/article/details/#t7)

[DMA中断](https://blog.csdn.net/qq_25218501/article/details/#t8)

[DMA请求映像](https://blog.csdn.net/qq_25218501/article/details/#t9)

[DMA的使用步骤](https://blog.csdn.net/qq_25218501/article/details/#t10)

[HAL库中的DMA功能实例](https://blog.csdn.net/qq_25218501/article/details/#t11)

[句柄结构体介绍（以DMA为例）](https://blog.csdn.net/qq_25218501/article/details/#%E5%8F%A5%E6%9F%84%E7%BB%93%E6%9E%84%E4%BD%93%E4%BB%8B%E7%BB%8D%EF%BC%88%E4%BB%A5DMA%E4%B8%BA%E4%BE%8B%EF%BC%89)

 [外设初始化结构体介绍](https://blog.csdn.net/qq_25218501/article/details/#%C2%A0%E5%A4%96%E8%AE%BE%E5%88%9D%E5%A7%8B%E5%8C%96%E7%BB%93%E6%9E%84%E4%BD%93%E4%BB%8B%E7%BB%8D)

[具体配置流程](https://blog.csdn.net/qq_25218501/article/details/#%E5%85%B7%E4%BD%93%E9%85%8D%E7%BD%AE%E6%B5%81%E7%A8%8B)

[CubeMX配置DMA过程](https://blog.csdn.net/qq_25218501/article/details/#t12)

[MEMTOMEM](https://blog.csdn.net/qq_25218501/article/details/#MEMTOMEM)

[P TO M](https://blog.csdn.net/qq_25218501/article/details/#P%20TO%20M)

[M TO P](https://blog.csdn.net/qq_25218501/article/details/#M%20TO%20P)

[生成工程讲解](https://blog.csdn.net/qq_25218501/article/details/#%E7%94%9F%E6%88%90%E5%B7%A5%E7%A8%8B%E8%AE%B2%E8%A7%A3)

[DMA具体实现功能讲解](https://blog.csdn.net/qq_25218501/article/details/#t13)

[存储器到存储器](https://blog.csdn.net/qq_25218501/article/details/#%E5%AD%98%E5%82%A8%E5%99%A8%E5%88%B0%E5%AD%98%E5%82%A8%E5%99%A8)

[实验现象](https://blog.csdn.net/qq_25218501/article/details/#%E5%AE%9E%E9%AA%8C%E7%8E%B0%E8%B1%A1)

[存储器到外设](https://blog.csdn.net/qq_25218501/article/details/#%E5%AD%98%E5%82%A8%E5%99%A8%E5%88%B0%E5%A4%96%E8%AE%BE)

[实验现象](https://blog.csdn.net/qq_25218501/article/details/#%E5%AE%9E%E9%AA%8C%E7%8E%B0%E8%B1%A1)

## 0.1 DMA-直接存储器访问控制器

### 0.1.1 DMA概览

#### 0.1.1.1 DMA的作用

- 直接存储器访问 (DMA) 用于在外设与存储器之间以及存储器与存储器之间，提供高速数据传输。可以在无需任何 CPU 操作的情况下通过 DMA 快速移动数据。这样节省的 CPU 资源可供其它操作使用。
- 外设到存储器

- 比如我们将串口接收到的数据，由串口外设的数据寄存器直接将数据通过DMA搬运到FLASH中

- 外设到存储器

- 比如我们将Flash中存储的数据通过DMA搬运到串口外设的数据寄存器，如何直接发送个上位机

- 存储器到存储器，比如将FLASH的数据通过DMA搬运到SRAM

-总结：DMA的就是CPU的助手、数据搬运工。

#### 0.1.1.2 DMA框图

![[../../_resources/DMA/cc1e339025ac9efe31a3bb1ce3a191c3_MD5.png]]

#### 0.1.1.3 DMA外设要点概括

对于DMA这个器件，它的功能就是建立起一个数据传输通道。

![[../../_resources/DMA/d4f50b4b89ed6548f459833e9f221620_MD5.png]]

#### 0.1.1.4 DMA功能对比

![[../../_resources/DMA/cb41838996e8cea8f1f4967f7aeb9b97_MD5.png]]

### 0.1.2 STMF10x DMA具体内容

#### 0.1.2.1 DMA主要特性

● 12个独立的可配置的通道(请求)： DMA1有7个通道， DMA2有5个通道

● 每个通道都直接连接专用的硬件DMA请求，每个通道都同样支持软件触发。这些功能通过软件来配置。

● 在同一个DMA模块上，多个请求间的优先权可以通过软件编程设置(共有四级：很高、高、中等和低)，优先权设置相等时由硬件决定(请求0优先于请求1，依此类推) 。

● 独立数据源和目标数据区的传输宽度(字节、半字、全字)，模拟打包和拆包的过程。源和目标地址必须按数据传输宽度对齐。

● 支持循环的缓冲器管理

● 每个通道都有3个事件标志(DMA半传输、 DMA传输完成和DMA传输出错)，这3个事件标志逻辑或成为一个单独的中断请求。

● 存储器和存储器间的传输

● 外设和存储器、存储器和外设之间的传输

● 闪存、 SRAM、外设的SRAM、 APB1、 APB2和AHB外设均可作为访问的源和目标。

● 可编程的数据传输数目：最大为65535

#### 0.1.2.2 [DMA中断](https://so.csdn.net/so/search?q=DMA%E4%B8%AD%E6%96%AD&spm=1001.2101.3001.7020)

每个DMA通道都可以在DMA传输过半、传输完成和传输错误时产生中断。为应用的灵活性考 虑，通过设置寄存器的不同位来打开这些中断。

![[../../_resources/DMA/c68c557846c85cbe350b0bd6e348c4c7_MD5.png]]

#### 0.1.2.3 DMA请求映像

**DMA1控制器**

从外设(TIMx[x=1、 2、 3、 4]、 ADC1、 SPI1、 SPI/I2S2、 I2Cx[x=1、 2]和USARTx[x=1、 2、 3]) 产生的7个请求，通过逻辑或输入到DMA1控制器，这意味着同时只能有一个请求有效。参见下 图的DMA1请求映像。

外设的DMA请求，可以通过设置相应外设寄存器中的控制位，被独立地开启或关闭。

![[../../_resources/DMA/b03fb8c57c09508a91f0c28459097782_MD5.png]]

**DMA控制器(DMA)**

![[../../_resources/DMA/d69c101ce2f424bde8e9f7603956ef02_MD5.png]]

**DMA2控制器**

从外设(TIMx[5、 6、 7、 8]、 ADC3、 SPI/I2S3、 UART4、 DAC通道1、 2和SDIO)产生的5个请 求，经逻辑或输入到DMA2控制器，这意味着同时只能有一个请求有效。参见下图的DMA2请求 映像。

外设的DMA请求，可以通过设置相应外设寄存器中的DMA控制位，被独立地开启或关闭。

注意： DMA2控制器及相关请求仅存在于大容量产品和互联型产品中。

![[../../_resources/DMA/d356c3a6b7787bda28af5870cc7143f8_MD5.png]]

**DMA控制器(DMA)**

![[../../_resources/DMA/f56a6ef4ad140ac5fd201b541e40df05_MD5.png]]

#### 0.1.2.4 DMA的使用步骤

1、建立（选取）传输通道。

        存储器->存储器

        外设->存储器

        存储器->外设

2、确定传输对象。

     具体的功能，比如：

        UART（源）->内存（目标）

        内存数据（源）->UART（目标）

3、敲定传输细节。

        确定由谁来产生DMA请求,外设的DMA请求对应通道

        通道优先级

        确定传输数据双方的数据格式

        确定数据是否需要一直采集（循环模式是否使能）

        是否需要传输标志\中断

#### 0.1.2.5 HAL库中的DMA功能实例

- DMA句柄结构体：

        -DMA_HandleTypeDef

- DMA初始化结构体：

        -DMA_InitTypeDef

HAL库中外设驱动的实现（任意外设通用）：

1、[句柄](https://so.csdn.net/so/search?q=%E5%8F%A5%E6%9F%84&spm=1001.2101.3001.7020)结构体（DMA_HandleTypeDef）：

Instance：它指向了外设内，一个具体的外设成员。如：ADC里的ADC1、ADC2，UART里的UART1、UART2，DMA的Channel1、Channel2，实际上它用指针指向一个外设基地址。

Init：指向了一个具体外设的初始化结构体，用来配置外设的工作参数。

-大家可以理解为句柄结构体就是HAL库的一种封装形式，也是通过句柄结构体实现对外设的初始化。

2、初始化结构体(xx_InitTypeDef)：

根据外设的各种配置寄存器，组织起来的外设参数配置结构体，内附在xx_HandleTypeDef结构体中。

##### 0.1.2.5.1 句柄结构体介绍（以DMA为例）

![[../../_resources/DMA/ae70b336bbff94fb8db9f2a721eafe5c_MD5.png]]

DMA_HandleTypeDef结构体里有两个最主要的成员

第一个是Instance，它指向了一个具体的外设基地址，当我们使用句柄结构体来进行具体初始化的时候，它就可以通过这个来指向一个具体外设的寄存器来进行我们的外设初始化。

第二个是Init（初始化结构体），它是用于配置特定外设的参数配置的，如串口的波特率、数据位数等等

第三个成员Lock（锁）和第四个成员（State）一般很少用，这里不做讲解。

而后面的几个函数都是外设的回调函数（一般在中断中配合外设标志位来使用）：

第一个函数用于传输完成。由于是函数指针，因此需要我们自己定义函数，然后进行传参。

第二个函数用于传输过半。

第三个参数用于传输出错。

第四个参数用于传输中止。

回调函数可以这样理解：比如我们调用了hal库的一个中断处理函数，这个中断处理函数是在中断中进行的，然后这个中断处理函数就会接受一个参数（句柄结构体），通过结构体里的函数指针就可以回调我们在这个结构体里给它定义的回调函数，其实这个回调就是执行的意思。

![[../../_resources/DMA/a233df6b9a1592453d1880d286df5f48_MD5.png]]

最后的三个成员ErrorCode、DmaBaseAddress、ChannelIndex的具体一成员信息都是和外设相关的，也一般很少使用，具体我们可以查看参考手册来进行了解。

##### 0.1.2.5.2  外设初始化结构体介绍

DMA_InitTypeDef，它是根据DMA外设的各种配置寄存器，组织起来的外设参数配置结构体，是内附在DMA_HandleTypeDef句柄结构体里的

![[../../_resources/DMA/d39564228fa6be49559b41fd2c094728_MD5.png]]

![[../../_resources/DMA/a48834dda36dfab485ac4e95b647ceec_MD5.png]]

##### 0.1.2.5.3 具体配置流程

首先初始化句柄结构体，将Instance成员指向一个具体的外设，它是DMA_Channel_TypeDef类型的，通过搜索可以发现，它对应着DMA1_Channel1到DMA1_Channel7，和DMA2_Channel1到DMA2_Channel5。通过指向一个具体的外设通道，从而建立起对应的外设请求。

![[../../_resources/DMA/8f9f3005d57a59805ffe2c2f8bfbfb87_MD5.png]]

然后配置DMA的工作模式，通过配置Init（初始化结构体）

配置第一个成员Direction，可以通过ctrl+f搜索到具体配置的传输方向

![[../../_resources/DMA/82b4a89a4def5cb282f93e8d05db8157_MD5.png]]

第二个成员PeriphInc和第三个成员MemInc

配置是否使能递增模式（分别对应外设和内存），如果我们要搬运的数据存储在一块连续字节的内存内，我们就需要采用递增才能将数据完整搬运到另一块内存中，它会自动发送完第一个内存地址的数据，然后进行内存地址的偏移，继续进行下一个地址内容的发送。对于寄存器而言我们也可以理解为一块内存。

![[../../_resources/DMA/dfb1b2437595616dff88a47109b4599a_MD5.png]]

第五、第六个成员PeriphDataAlignment和MemDataAlignment分别对应配置外设和内存的传输的数据格式

我们以串口为例，串口的数据寄存器只有8bit，我们要选择DMA_PDATAALIGN_BYTE。而对于内存，如果我们使用了一个int型的话，那么就对应着32bit，我们就要选择DMA_MDATAALIGN_WORD

![[../../_resources/DMA/6d392f8e5b2cdc47564c828788adc2ca_MD5.png]]

第七个成员Mode，我们可以选择循环模式还是单次传输模式。

![[../../_resources/DMA/e48cc25b4c8e79ba32a62a92d7104560_MD5.png]]

最后一个成员Priority，我们可以配置DMA的优先级

![[../../_resources/DMA/9a445a870921f63c13d66c6243f202c9_MD5.png]]

具体我们可以看DMA的仲裁器可以知道

![[../../_resources/DMA/33e523849c0c693264c70e90ec5e2af4_MD5.png]]

总结起来就是hal库正是通过这种句柄结构体+初始化结构体的方式将外设很好地封装起来并且组织好了

![[../../_resources/DMA/44e0566e188dca27a25e99a2a3d85d59_MD5.png]]

配置完所有参数我们要使用HAL_DMA_Init传入句柄结构体，从而完成DMA外设的初始化

#### 0.1.2.6 CubeMX配置DMA过程

##### 0.1.2.6.1 MEMTOMEM

其中有一点需要补充注意的是，是否递增的选择：要看源数据存储是否在一块连续地址上，目标数据是否也是连续的地址字节，即是否大于一个字节。

![[../../_resources/DMA/a2bc702ca40a57ee57486e0e19b363ea_MD5.png]]

数据的位数宽度可参考

![[../../_resources/DMA/343ff563b508d289b7172e09d839d587_MD5.png]]

##### 0.1.2.6.2 P TO M

以串口1为例，USART1_RX用于从外面读取数据，因此DMA请求方向是外设到内存，通道固定是DMA1 Channel5，模式选择Normal，循环模式一般在DAC才选用，优先级选择低，数据位数宽度最好根据串口数据寄存器位数选择为Byte，一般CubeMX会自动根据外设寄存器选择好

![[../../_resources/DMA/cbab38583c13116ab7308d186e1434b6_MD5.png]]

![[../../_resources/DMA/a325c4beef36d2cdaa98a143d347df5c_MD5.png]]

##### 0.1.2.6.3 M TO P

USART1_TX用于向外发送数据，因此DMA请求方向是内存到外设，通道固定是DMA1 Channel4，模式选择Normal，循环模式一般在DAC才选用，优先级选择低，数据位数宽度最好根据串口数据寄存器位数选择为Byte

![[../../_resources/DMA/668fb09cdcf20a60a67e27f4242a78c8_MD5.png]]

最终可以在DMA1和DMA2看见对应通道的配置

![[../../_resources/DMA/63d19204f7f86aaf98b9d086388dce5b_MD5.png]]

##### 0.1.2.6.4 生成工程讲解

生成工程，在dma.c中可以看到从DMA句柄结构体的定义来配置我们的M TO M，以及DMA外设通道4和5在中断向量表的优先级，CubeMX都自动配置好了

![[../../_resources/DMA/7453f0318b234e3eeaa5cd97009f00a8_MD5.png]]

![[../../_resources/DMA/f5fca43f03305ee91076d3cba515debd_MD5.png]]

由于DMA的外设到内存和内存到外设是和我们的外设紧密联系的，所以工程自动将DMA配置的这一部分代码放在了外设文件里。

再看usart.c中串口的配置。首先定义了串口句柄结构体和两个DMA句柄结构体（分别是发送和接收）。接着进行了串口参数的初始化和串口用到的GPIO配置

![[../../_resources/DMA/8d1e86612f9686069fb2675ab74205df_MD5.png]]

![[../../_resources/DMA/559eb1ae705e9743d08a61655de021bd_MD5.png]]

然后初始化了我们串口外设到内存的DMA句柄结构体 并 紧接着调用了__HAL_LINKDMA()函数，将我们的串口外设和DMA建立链接在一起。

![[../../_resources/DMA/b8e0a49f110ffdaa59dbdd9e1ed5ed36_MD5.png]]

以TX为例，__HAL_LINKDMA(uartHandle,hdmatx,hdma_usart1_tx);函数其中第一个参数是串口句柄结构体，第二个参数是串口句柄结构体中的DMA句柄指针成员hdmatx，第三个是我们已经实例化了的DMA句柄结构体。所以__HAL_LINKDMA函数的作用是将串口句柄结构体中的DMA句柄指针hdmatx赋值具体实例化为hdma_usart1_tx，从而关联为实际的DMA功能。

![[../../_resources/DMA/a8aacaa0270e4ba314c49f320a2364ca_MD5.png]]

通过查看stm32fxx_hal_dma.c文件的指导说明，我们了解到DMA的具体用法。如

HAL_DMA_Start()可以开启DMA传输

HAL_DMA_PollForTransfer()可以对DMA的传输结束状态进行轮询

HAL_DMA_Start_IT()可以开启DMA传输中断

HAL_DMA_IRQHandler()为具体的DMA中断函数

最后还有一些DMA使能、检测标志函数的相关说明

![[../../_resources/DMA/d18472a002a621f6588835b7ab3c3fa6_MD5.png]]

#### 0.1.2.7 DMA具体实现功能讲解

##### 0.1.2.7.1 存储器到存储器

首先定义一个无符号整型的常量数组并初始化（存储在FLASH中），如何定义一共普通的无符号数组，两个数组成员个数都是32。

![[../../_resources/DMA/e1b179a56537cd762a3e9349f825defe_MD5.png]]

然后再main函数中进行相关的初始化配置，再使用DMA时，必须调用

HAL_DMA_Start(&DMA_Handle,(uint32_t)aSRC_Const_Buffer,(uint32_t)aDST_Buffer,BUFFER_SIZE);

函数来软件启动DMA（存储器到存储器必须软件开启），四个参数分别为DMA的句柄结构体、源地址、目标地址和传输数据大小

![[../../_resources/DMA/9e593b85458aa1cf2ce4740768077cd1_MD5.png]]

当传输完数据之后，我们可以通过调用__HAL_DMA_GET_FLAG(&DMA_Handle,DMA_FLAG_TC6)函数来判断DMA传输的状态

![[../../_resources/DMA/79b4e22b4e111bfd01f061cb862f66dc_MD5.png]]

![[../../_resources/DMA/517ac95fa1b80f0822edf1d520619a56_MD5.png]]

![[../../_resources/DMA/ea684c0f2e4932086a0e83906cb06845_MD5.png]]

最后烧入程序，通过Debug进行调试

###### 0.1.2.7.1.1 实验现象

![[../../_resources/DMA/170e3446328fcba3f496968c59e6d1c8_MD5.png]]

##### 0.1.2.7.2 存储器到外设

首先初始化串口和DMA，具体注意DMA的配置（与前面CubeMX一致）

![[../../_resources/DMA/b754ac66ddc7d91df84d35f946114396_MD5.png]]

然后填充要发送的数据，接着调用HAL_UART_Transmit_DMA(&UartHandle, (uint8_t *)SendBuff ,SENDBUFF_SIZE);函数，第一个参数为串口句柄结构体、第二个参数为发送缓冲区、第三个参数为发送缓冲区的大小。

通过这个函数来使用串口1的DMA发送功能，这样就可以进行DMA的传输了

![[../../_resources/DMA/3ccea8b56c4869a1abc02caf1b81062b_MD5.png]]

具体进入HAL_UART_Transmit_DMA函数，可以发现其中调用了一些UART DAM句柄结构体中的回调函数，最终通过HAL_DMA_Start_IT()函数传输数据到串口数据寄存器中发送出去

![[../../_resources/DMA/5b2e7d2bf69164931225fae6a58b7dda_MD5.png]]

![[../../_resources/DMA/c8e432391e36907574080fe4b53cd3d0_MD5.png]]

进入HAL_DMA_Start_IT()中可以找到DMA的使能__HAL_DMA_ENABLE(hdma);

![[../../_resources/DMA/2b8c3d0d178dd242764db31368b30d8b_MD5.png]]

###### 0.1.2.7.2.1 实验现象

DMA一般模式

![[../../_resources/DMA/ba2bae41265d2b8e47725b2e03ca5a15_MD5.png]]

DMA循环模式

我们可以发现DAM在一直不停得传输。在传输的过程中，CPU可以做自己的事情，如进行LED灯的闪烁显示等

![[../../_resources/DMA/ba208808cb9d965b2f83ad6027ed0da3_MD5.png]]