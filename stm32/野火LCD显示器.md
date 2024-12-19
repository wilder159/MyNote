# 1 显示器介绍
## 1.1 显示器基本参数
1. 像素。是组成图像的基本单位
2. 分辨率。“行像素值x列像素值”表示像素的基本单位
3. 色彩深度。表示显示器的每个像素点能够表示多少种颜色，一般用bit表示
4. 显示器尺寸。一般用英寸表示，指屏幕对角线的长度
5. 点距。指两个像素点之间的距离。相同尺寸屏幕点距越小，分辨率越大，屏幕越细腻
## 1.2 液晶控制原理
[[../_resources/显示器/d32e943a0bbfbbba4b45d004673bcab0_MD5.jpeg|两种液晶显示屏控制框图]]

![[../_resources/显示器/d32e943a0bbfbbba4b45d004673bcab0_MD5.jpeg]]
## 1.3 液晶面板的控制信号（RGB接口）
[[../_resources/显示器/88ed9f9edc386b2a7075dabcd3335208_MD5.jpeg|液晶面板的信号线]]
![[../_resources/显示器/88ed9f9edc386b2a7075dabcd3335208_MD5.jpeg]]
1. RGB信号线。RGB信号线各有8根，分别用于表示液晶屏一个像素点的红、绿、蓝颜色分量。常见的颜色表示会在“RGB”后面附带各个颜色分量值的数据位数，如RGB565表示红绿蓝的数据线数分别为5、6、5根， 一共为16个数据位，可表示216种颜色；而这个液晶屏的种颜色分量的数据线都有8根， 所以它支持RGB888格式，一共24位数据线，可表示的颜色为224种。
2. 同步时钟信号CLK。每个时钟传输一个像素点
3. 水平同步信号HSYNC。每传输完一行数据后，HSYNC会发生一次电平跳变
4. 垂直同步信号VSYNC。每传输完一帧数据，VSYNC会发生一次电平跳变。 
5. 数据使能信号。表示信号的有效性。为高数据有效。
“帧”是图像的单位，一幅图像称为一帧，一帧指一个完整屏液晶像素点。人们常常用“帧/秒”来表示液晶屏的刷新特性，如液晶屏以60帧/秒的速率运行时，VSYNC每秒钟电平会跳变60次。
## 1.4 液晶数据传输时序

[[../_resources/显示器/275e3964071aebc52326a97e35404251_MD5.jpeg| 液晶时序图]]
![[../_resources/显示器/275e3964071aebc52326a97e35404251_MD5.jpeg]]
水平与垂直同步信号每次发生跳变的时候，RGB信号线的数据是无效的。同时，每一行以及每一帧数据传输的前后，各自也有一段数据是无效的。所以要注意根据器件特性，忽略掉无效数据。
[[../_resources/显示器/c802e7866fba9b619622e96dc781e83b_MD5.jpeg| 数据传输图解]]
![[../_resources/显示器/c802e7866fba9b619622e96dc781e83b_MD5.jpeg]]

[[../_resources/显示器/d5193b426efb65922f9a5f257989c619_MD5.jpeg|时序中时间参数]]
![[../_resources/显示器/d5193b426efb65922f9a5f257989c619_MD5.jpeg]]
## 1.5 显存

液晶屏中的每个像素点都是数据，在实际应用中需要把每个像素点的数据缓存起来，再传输给液晶屏，一般会使用SRAM或SDRAM性质的存储器， 而这些专门用于存储显示数据的存储器，则被称为显存。显存一般至少要能存储液晶屏的一帧显示数据，如分辨率为800x480的液晶屏， 使用RGB888格式显示，它的一帧显示数据大小为：3x800x480=1152000字节；若使用RGB565格式显示，一帧显示数据大小为：2x800x480=768000字节。

一般来说，外置的液晶控制器会自带显存，而像STM32F429等集成液晶控制器的芯片可使用内部SRAM或外扩SDRAM用于显存空间。

# 2 液晶屏简介
## 2.1 ILI9341液晶控制器简介
本液晶屏内部包含有一个液晶控制芯片ILI9341，它的内部结构非常复杂，见图 [ILI9341控制器内部框图](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id19)。 该芯片最主核心部分是位于中间的GRAM(Graphics RAM)，它就是显存。GRAM中每个存储单元都对应着液晶面板的一个像素点。 它右侧的各种模块共同作用把GRAM存储单元的数据转化成液晶面板的控制信号，使像素点呈现特定的颜色，而像素点组合起来则成为一幅完整的图像。

框图的左上角为ILI9341的主要控制信号线和配置引脚，根据其不同状态设置可以使芯片工作在不同的模式， 如每个像素点的位数是6、16还是18位；可配置使用SPI接口、8080接口还是RGB接口与MCU进行通讯。 MCU通过SPI、8080接口或RGB接口与ILI9341进行通讯，从而访问它的控制寄存器(CR)、地址计数器(AC)、及GRAM。

在GRAM的左侧还有一个LED控制器(LED Controller)。LCD为非发光性的显示装置， 它需要借助背光源才能达到显示功能，LED控制器就是用来控制液晶屏中的LED背光源。
[[../_resources/显示器/f063f63838e11e08b71daeeef64c7963_MD5.jpeg|Open: Pasted image 20241211150827.png]]
![[../_resources/显示器/f063f63838e11e08b71daeeef64c7963_MD5.jpeg]]这些信号线的说明见表 [液晶屏引出的信号线说明](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id22)。
[[../_resources/显示器/6609d8e76fa3217e85e72e39734021cc_MD5.jpeg|Open: Pasted image 20241211151403.png]]
![[../_resources/显示器/6609d8e76fa3217e85e72e39734021cc_MD5.jpeg]]
这些信号线即8080通讯接口，带X的表示低电平有效，STM32通过该接口与ILI9341芯片进行通讯，实现对液晶屏的控制。 通讯的内容主要包括命令和显存数据，显存数据即各个像素点的RGB565内容；命令是指对ILI9341的控制指令， MCU可通过8080接口发送命令编码控制ILI9341的工作方式，例如复位指令、设置光标指令、睡眠模式指令等等， 具体的指令在《ILI9341.pdf》数据手册均有详细说明。 写命令时序图见图 [使用18条数据线的8080接口写命令时序](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id23)。
[[../_resources/显示器/dcc4c0706ea258f56e5840727fe3bca4_MD5.jpeg|Open: Pasted image 20241211153808.png]]
![[../_resources/显示器/dcc4c0706ea258f56e5840727fe3bca4_MD5.jpeg]]
由图可知，写命令时序由片选信号CSX拉低开始，对数据/命令选择信号线D/CX也置低电平表示写入的是命令地址(可理解为命令编码，如软件复位命令：0x01)， 以写信号WRX为低，读信号RDX为高表示数据传输方向为写入，同时，在数据线D[17:0](或D[15:0])输出命令地址， 在第二个传输阶段传送的是命令的参数，所以D/CX要置高电平，表示写入的是命令数据，命令数据是某些指令带有的参数，如复位指令编码为0x01，它后面可以带一个参数， 该参数表示多少秒后复位(实际的复位命令不含参数，此处只是为了讲解指令编码与参数的区别)。

当需要把像素数据写入GRAM时，过程很类似，把片选信号CSX拉低后，再把数据/命令选择信号线D/CX置为高电平， 这时由D[17:0]传输的数据则会被ILI9341保存至它的GRAM中。

## 2.2  使用STM32的FSMC模拟8080接口时序

ILI9341的8080通讯接口时序可以由STM32使用普通I/O接口进行模拟，但这样效率太低，STM32提供了一种特别的控制方法——使用FSMC接口实现8080时序。

### 2.2.1 28.4.1. FSMC简介

在前面的《FSMC—扩展外部SRAM》章节中了解到STM32的FSMC外设可以用于控制扩展的外部存储器， 而MCU对液晶屏的操作实际上就是把显示数据写入到显存中，与控制存储器非常类似， 且8080接口的通讯时序完全可以使用FSMC外设产生，因而非常适合使用FSMC控制液晶屏。

FSMC外设的结构见图 [FSMC结构图](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id24)。

![[../_resources/显示器/748a11e35c6140c8460a0076eeb5700e_MD5.jpg]]

控制LCD时，是使用FSMC的NORPSRAM模式的，且与前面使用FSMC控制SRAM的稍有不同， 控制SRAM时使用的是模式A，而控制LCD时使用的是与NOR FLASH一样的模式B， 所以我们重点分析框图中NOR FLASH控制信号线部分。 控制NOR FLASH主要使用到表 [FSMC控制NOR_FLASH的信号线](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#fsmcnor-flash) 的信号线：

![[../_resources/显示器/aac6088a757f5d491e0ca792a9e8166b_MD5.png]]

在控制LCD时，使用的是类似异步、地址与数据线独立的NOR FLASH控制方式， 所以实际上CLK、NWAIT、NADV引脚并没有使用到。

FSMC NOR/PSRAM中的模式B的写时序见图 [FSMC写NOR_FLASH的时序图](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id25)。

![[../_resources/显示器/53232f7fa3a5ceed3fa22772ca1a3c59_MD5.jpg]]

根据STM32对寻址空间的地址映射，地址0x6000 0000 ~0x9FFF FFFF是映射到外部存储器的， 而其中的0x6000 0000 ~0x6FFF FFFF则是分配给NOR FLASH、 PSRAM这类可直接寻址的器件。

当FSMC外设被配置成正常工作，并且外部接了NOR FLASH时， 若向0x60000000地址写入数据如0xABCD，FSMC会自动在各信号线上产生相应的电平信号， 写入数据。FSMC会控制片选信号NE1选择相应的NOR芯片，然后使用地址线A[25:0]输出0x60000000，在NWE写使能信号线上发出低电平的写使能信号， 而要写入的数据信号0xABCD则从数据线D[15:0]输出，然后数据就被保存到NOR FLASH中了。

## 2.3 用FSMC模拟8080时序

![[../_resources/显示器/2522316d21454996aa5f1c0984397480_MD5.jpg]]

见表 [FSMC的NOR与8080信号线对比](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#fsmcnor8080) ，对比FSMC NOR/PSRAM中的模式B时序与ILI9341液晶控制器芯片使用的8080时序可发现， 这两个时序是十分相似的(除了FSMC的地址线A和8080的D/CX线，可以说是完全一样)。

![[../_resources/显示器/92b41e96cba95ecd000de7d45b1f8acc_MD5.png]]

对于FSMC和8080接口，前四种信号线都是完全一样的，仅仅是FSMC的地址信号线A[25:0]与8080的数据/命令选择线D/CX有区别。而对于D/CX线， 它为高电平的时候表示数值，为低电平的时候表示命令，如果能使用FSMC的A地址线根据不同的情况产生对应的电平， 那么就完全可以使用FSMC来产生8080接口需要的时序了。

为了模拟出8080时序，我们可以把FSMC的A0地址线(也可以使用其它A1/A2等地址线)与ILI9341芯片8080接口的D/CX信号线连接， 那么当A0为高电平时(即D/CX为高电平)，数据线D[15:0]的信号会被ILI9341理解为数值，若A0为低电平时(即D/CX为低电平)， 传输的信号则会被理解为命令。

由于FSMC会自动产生地址信号，当使用FSMC向0x6xxx xxx1、0x6xxx xxx3、 0x6xxx xxx5…这些奇数地址写入数据时，地址最低位的值均为1， 所以它会控制地址线A0(D/CX)输出高电平，那么这时通过数据线传输的信号会被理解为数值； 若向0x6xxx xxx0 、0x6xxx xxx2、0x6xxx xxx4…这些偶数地址写入数据时， 地址最低位的值均为0，所以它会控制地址线A0(D/CX)输出低电平， 因此这时通过数据线传输的信号会被理解为命令，见表 [使用FSMC输出地址示例](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id26)。

![[../_resources/显示器/07cdd21c2001dba29a6ec3f33b553b52_MD5.png]]

有了这个基础，只要配置好FSMC外设，然后在代码中利用指针变量，向不同的地址单元写入数据，就能够由FSMC模拟出的8080接口向ILI9341写入控制命令或GRAM的数据了。

注意：在实际控制时，以上地址计算方式还不完整，还需要注意HADDR内部地址与FSMC地址信号线的转换，关于这部分内容在代码讲解时再详细举例说明。

## 2.4 NOR FLASH时序结构体

在讲解程序前，再来了解一下与FSMC NOR FLASH控制相关的结构体。

与控制SRAM时一样，控制FSMC使用NOR FLASH存储器时主要是配置时序寄存器以及控制寄存器， 利用ST标准库的时序结构体以及初始化结构体可以很方便地写入参数。

NOR FLASH时序结构体的成员见 [代码清单:液晶显示-2](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id27)。

代码清单:液晶显示-2 NOR FLASH时序结构体FSMC_NORSRAMTimingInitTypeDef[](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id27 "永久链接至代码")

|   |   |
|---|---|
|1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10|typedef struct<br>{<br>uint32_t FSMC_AddressSetupTime;       /*地址建立时间，0-0xF个HCLK周期*/<br>uint32_t FSMC_AddressHoldTime;        /*地址保持时间，0-0xF个HCLK周期*/<br>uint32_t FSMC_DataSetupTime;           /*地址建立时间，0-0xF个HCLK周期*/<br>uint32_t FSMC_BusTurnAroundDuration;/*总线转换周期,0-0xF个HCLK周期，在NOR FLASH */<br>uint32_t FSMC_CLKDivision;/*时钟分频因子,1-0xF，若控制异步存储器，本参数无效 */<br>uint32_t FSMC_DataLatency;    /*数据延迟时间，若控制异步存储器，本参数无效 */<br>uint32_t FSMC_AccessMode;             /*设置访问模式 */<br>}FSMC_NORSRAMTimingInitTypeDef;|

这个结构体与SRAM中的时序结构体完全一样，以下仅列出控制NOR FLASH时使用模式B用到的结构体成员说明：

(1) FSMC_AddressSetupTime

> 本成员设置地址建立时间，即FSMC读写时序图 [FSMC写NOR_FLASH的时序图](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id25) 中的ADDSET值，它可以被设置为0-0xF个HCLK周期数， 按STM32标准库的默认配置，HCLK的时钟频率为72MHz，即一个HCLK周期为1/72微秒。

(2) FSMC_DataSetupTime

> 本成员设置数据建立时间，即FSMC读写时序图 [FSMC写NOR_FLASH的时序图](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id25) 中的DATAST值，它可以被设置为0-0xF个HCLK周期数。

(3) FSMC_AccessMode ..

> 本成员设置存储器访问模式，不同的模式下FSMC访问存储器地址时引脚输出的时序不一样，可选FSMC_AccessMode_A/B/C/D模式。 控制异步NOR FLASH时使用B模式。

这个FSMC_NORSRAMTimingInitTypeDef时序结构体配置的延时参数，将作为下一节的FSMC NOR FLASH初始化结构体的一个成员。

## 2.5 28.7. FSMC初始化结构体
FSMC控制NOR FLASH相关的结构体，初始化结构体见 [代码清单:液晶显示-3](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id29)。

代码清单:液晶显示-3 NOR FLASH初始化结构体FSMC_NORSRAMInitTypeDef[](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id29 "永久链接至代码")
```c
/**
* @brief  FSMC NOR/SRAM Init structure definition
*/
typedef struct
{
uint32_t FSMC_Bank;                /*设置要控制的Bank区域 */
uint32_t FSMC_DataAddressMux;      /*设置地址总线与数据总线是否复用 */
uint32_t FSMC_MemoryType;          /*设置存储器的类型 */
uint32_t FSMC_MemoryDataWidth;     /*设置存储器的数据宽度*/
uint32_t FSMC_BurstAccessMode; /*设置是否支持突发访问模式，只支持同步类型的存储器 */
uint32_t FSMC_AsynchronousWait;     /*设置是否使能在同步传输时的等待信号，*/
uint32_t FSMC_WaitSignalPolarity;  /*设置等待信号的极性*/
uint32_t FSMC_WrapMode;            /*设置是否支持对齐的突发模式 */
uint32_t FSMC_WaitSignalActive; /*配置等待信号在等待前有效还是等待期间有效 */
uint32_t FSMC_WriteOperation;      /*设置是否写使能 */
uint32_t FSMC_WaitSignal;          /*设置是否使能等待状态插入 */
uint32_t FSMC_ExtendedMode;        /*设置是否使能扩展模式 */
uint32_t FSMC_WriteBurst;          /*设置是否使能写突发操作*/
/*当不使用扩展模式时，本参数用于配置读写时序，否则用于配置读时序*/
FSMC_NORSRAMTimingInitTypeDef* FSMC_ReadWriteTimingStruct;
/*当使用扩展模式时，本参数用于配置写时序*/
FSMC_NORSRAMTimingInitTypeDef* FSMC_WriteTimingStruct;
}FSMC_NORSRAMInitTypeDef;
```
这部分内容与FSMC控制SRAM时完全一致，此处仅列出与NOR FLASH模式B相关的一些结构体成员的说明：
(1) FSMC_Bank
本成员用于选择FSMC映射的存储区域，它的可选参数以及相应的内核地址映射范围见表 [可以选择的存储器区域及区域对应的地址范围](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id30)。

![[../_resources/显示器/a922f1ae160907a966438ca014e5c8be_MD5.png]]
(2) FSMC_MemoryType
 本成员用于设置要控制的存储器类型，它支持控制的存储器类型为SRAM、 PSRAM以及NOR FLASH(FSMC_MemoryType_SRAM/PSRAM/NOR)。
(3) FSMC_MemoryDataWidth
 本成员用于设置要控制的存储器的数据宽度，可选择设置成8或16位(FSMC_MemoryDataWidth_8b /16b)。
(4) FSMC_WriteOperation
这个成员用于设置是否写使能(FSMC_WriteOperation_ Enable /Disable)， 禁止写使能的话FSMC只能从存储器中读取数据，不能写入。
(5) FSMC_ExtendedMode
本成员用于设置是否使用扩展模式(FSMC_ExtendedMode_Enable/Disable)，在非扩展模式下，对存储器读写的时序都只使用FSMC_BCR寄存器中的配置， 即下面的FSMC_ReadWriteTimingStruct结构体成员；在扩展模式下，对存储器的读写时序可以分开配置， 读时序使用FSMC_BCR寄存器，写时序使用FSMC_BWTR寄存器的配置，即后面的FSMC_WriteTimingStruct结构体成员。
(6) FSMC_ReadWriteTimingStruct
本成员是一个指针，赋值时使用上一小节中讲解的时序结构体FSMC_NORSRAMInitTypeDef设置，当不使用扩展模式时，读写时序都使用本成员的参数配置。
(7) FSMC_WriteTimingStruct
同样地，本成员也是一个时序结构体的指针，只有当使用扩展模式时，本配置才有效，它是写操作使用的时序。
对本结构体赋值完成后，调用FSMC_NORSRAMInit库函数即可把配置参数写入到FSMC_BCR及FSMC_BTR/BWTR寄存器中。

**初始化FSMC时钟**
函数开头使用库函数RCC_AHBPeriphClockCmd使能FSMC外设的时钟。
(2) 时序结构体赋值
接下来对时序结构体FSMC_NORSRAMTimingInitTypeDef赋值。在这个时序结构体配置中， 由于我们要使用异步NOR FLASH的方式模拟8080时序， 所以选择FSMC为模式B，在该模式下配置FSMC的控制时序结构体中， 实际上只有地址建立时间FSMC_AddressSetupTime（即ADDSET的值）以及数据建立时间FSMC_DataSetupTime（即DATAST的值）成员的配置值是有效的， 其它异步NOR FLASH没使用到的成员值全配置为0即可。而且，这些成员值使用的单位为：1个HCLK的时钟周期， 而HCLK的时钟频率为72MHz，对应每个时钟周期为1/72微秒。
由图 [ILI9341时序参数说明图](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id43) 及图 [ILI9341的时序参数要求](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id44) 中的ILI9341时序参数说明及要求可大致得知ILI9341的写周期为最小twc = 66ns， 而读周期最小为trdl+trod=45+20=65ns。 （对于读周期表中有参数要一个要求为trcfm和trc分别为450ns及160ns， 但经过测试并不需要遵照它们的指标要求）

![[../_resources/显示器/e860aa78c034ee607677a9000d73a670_MD5.jpg]]![[../_resources/显示器/3a6a455d9905c59611cf6f980494d20a_MD5.jpg]]

在FSMC代码中使用结构体中的FSMC_AddressSetupTime（即ADDSET的值）及FSMC_DataSetupTime（即DATAST的值）成员控制FSMC的读写周期， 见图 [FSMC的读写时序](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id45)。

![[../_resources/显示器/ff6bcbe9a45dbe10857e29d67fce1e68_MD5.jpg]]

结合ILI9341的时序要求和FSMC的配置图，代码中按照读写时序周期均要求至少66ns来计算，配置结果为ADDSET = 1及DATST = 4， 把时间单位1/72微秒(即1000/72纳秒)代入，因此读写周期的时间被配置为：

读周期：trc =((ADDSET+1)+(DATST+1)+2) *1000/72 = ((1+1)+(4+1)+2)*1000/72 = 125ns
写周期：twc =((ADDSET+1)+(DATST+1)) *1000/72 = ((1+1)+(4+1))*1000/72 = 97ns

所以把这两个参数值写入到FSMC后，它控制的读写周期都比ILI9341的最低要求值大。（经测试，这两个参数值也可以适当减小，您可以亲自试一下）
把计算得的参数赋值到时序结构体中的FSMC_AddressSetupTime（即ADDSET的值）FSMC_DataSetupTime（即DATAST的值）中， 然后再把时序结构体作为指针赋值到下面的FSMC初始化结构体中，作为读写的时序参数，最后再调用FSMC_NORSRAMInit函数即可把参数写入到相应的寄存器中。

(3) 配置FSMC初始化结构体

 函数接下来对FSMC SRAM的初始化结构体赋值。主要包括存储映射区域、存储器类型以及数据线宽度等，这些是根据外部电路设置的。
- 设置存储区域FSMC_Bank
 FSMC_Bank成员设置FSMC的NOR FLASH存储区域映射选择为宏 FSMC_Bank1_NORSRAMx (即FSMC_Bank1_NORSRAM4)， 这是由于我们的SRAM硬件连接到FSMC_NE4和NOR/PSRAM相关引脚， 所以对应到存储区域Bank1SRAM4，对应的基地址为0X6C00 0000；
 
- 存储器类型FSMC_MemoryType
 由于使用异步NOR FLASH模式模拟8080时序，所以FSMC_MemoryType成员要选择相应的FSMC_MemoryType_NOR；
 
- 数据线宽度FSMC_MemoryDataWidth
根据硬件的数据线连接，数据线宽度被配置为16位宽FSMC_MemoryDataWidth_16b；

- 写使能FSMC_WriteOperation
 FSMC_WriteOperation用于设置写使能，只有使能了才能正常使用FSMC向外部存储器写入数据；
 
- 扩展模式以及读写时序
在FSMC_ExtendedMode成员中可以配置是否使用扩展模式，当设置扩展模式时，读时序使用FSMC_ReadWriteTimingStruct中的配置，写时序使用FSMC_WriteTimingStruct中的配置，两种配置互相独立，可以赋值为不同的读写时序结构体。在本实例中不使用扩展模式，即 读写时序使用相同的配置，都是赋值为前面的readWriteTiming结构体；

- 其它
 配置FSMC还涉及到其它的结构体成员，但这些结构体成员与异步NOR FLASH控制不相关，都被设置为Disable了；
 
赋值完成后调用库函数FSMC_NORSRAMInit把初始化结构体配置的各种参数写入到FSMC_BCR控制寄存器及FSMC_BTR时序寄存器中。最后调用FSMC_NORSRAMCmd函数使能要控制的存储区域FSMC_Bank1_NORSRAM4。

**计算控制液晶屏时使用的地址**
初始化完FSMC后，即可使用类似扩展外部SRAM中的读取方式：通过访问某个地址，由FSMC产生时序与外部存储器通讯，进行读写。

同样地，当访问特定地址时，FSMC会产生相应的模拟8080时序，控制地址线输出要访问的内存地址，使用数据信号线接收或发送数据， 其它片选信号NE、读使能信号NOE、写使能信号NWE辅助产生完整的时序，而由于控制液晶屏的硬件连接中， 使用如图 [FSMC与8080端口连接简图](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id46) 中的连接来模拟8080时序， 所以FSMC产生的这些信号会被ILI9341接收，并且使用其中一根FSMC_Ax地址控制命令/数据选择引脚RS(即D/CX)，因此， 需要重点理解下当STM32访问什么地址时，对应的FSMC_Ax引脚会输出高电平表示传输的是数据，访问什么地址时， 对应的FSMC_Ax引脚会输出低电平表示传输的是命令。若理解了计算过程，以后就可以根据自己制作的硬件电路来计算访问地址了。

![[../_resources/显示器/13e8d53e65b7157de5069e1a528875a1_MD5.png]]

计算地址的过程如下：

(1) 本工程中使用的是FSMC_NE4作为8080_CS片选信号，所以首先可以确认地址范围， 当访问0X6C00 0000 ~ 0X6FFF FFFF地址时，FSMC均会对外产生片选有效的访问时序；

(2) 本工程中使用FSMC_A23地址线作为命令/数据选择线RS信号，所以在以上地址范围内， 再选择出使得FSMC_A23输出高电平的地址，即可控制表示数据， 选择出使得FSMC_A23输出低电平的地址，即可控制表示命令。

- 要使FSMC_A23地址线为高电平，实质是输出地址信号的第23位为1即可，使用0X6C00 0000~0X6FFF FFFF内的任意地址，作如下运算：
    

> 设置地址的第23位为1： 0X6C00 0000 |= (1<<23) = 0x6C80 0000

- 要使FSMC_A23地址线为低电平，实质是输出地址信号的第23位为0即可，使用0X6C00 0000~0X6FFF FFFF内的任意地址，作如下运算：
    

> 设置地址的第23位为0： 0X6C00 0000 &= ~ (1<<23) = 0x6C00 0000

(3) 但是，以上方法计算的地址还不完全正确，根据《STM32参考手册》对FSMC访问NOR FLASH的说明， 见图 [HADDR与FSMC地址线的说明](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#haddrfsmc) ，STM32内部访问地址时使用的是内部HADDR总线， 它是需要转换到外部存储器的内部AHB地址线，它是字节地址(8位)， 而存储器访问不都是按字节访问，因此接到存储器的地址线依存储器的数据宽度有所不同。

![[../_resources/显示器/dcb5017cd4748b05e956ad14d2d60a74_MD5.png]]

在本工程中使用的是16位的数据访问方式，所以HADDR与FSMC_A的地址线连接关系会左移一位，如HADDR1与FSMC_A0对应、HADDR2与FSMC_A1对应。因此，当FSMC_A0地址线为1时，实际上内部地址的第1位为1，FSMC_A1地址线为1时，实际上内部地址的第2位为1。同样地，当希望 FSMC_A16地址输出高电平或低电平时，需要重新调整计算公式：

- 要使FSMC_A23地址线为高电平，实质是访问内部HADDR地址的第(23+1)位为1即可， 使用0X6C00 0000~0X6FFF FFFF内的任意地址，作如下运算：
> 使FSMC_A23地址线为高电平：0X6C00 0000 |= (1<<(23+1)) = 0x6D00 0000
- 要使FSMC_A23地址线为低电平，实质是访问内部HADDR地址的第(23+1)位为0即可，使用0X6C00 0000~0X6FFF FFFF内的任意地址，作如下运算：
> 使FSMC_A23地址线为低电平： 0X6C00 0000 &= ~ (1<<(23+1)) = 0x6C00 0000

根据最终的计算结果，总结如下：当STM32访问内部的0x6D00 0000地址时，FSMC自动输出时序， 且使得与液晶屏的数据/命令选择线RS(即D/CX)相连的FSMC_A23输出高电平， 使得液晶屏会把传输过程理解为数据传输；类似地，当STM32访问内部的0X6C00 0000地址时， FSMC自动输出时序，且使得与液晶屏的数据/命令选择线RS(即D/CX)相连的FSMC_A23输出低电平， 使得液晶屏会把传输过程理解为命令传输。
[[../_resources/显示器/a2d58fefebf7d415292715a764f52960_MD5.jpeg|Open: Pasted image 20241211220855.png]]
![[../_resources/显示器/a2d58fefebf7d415292715a764f52960_MD5.jpeg]]
当设置了液晶显示窗口，再连续向液晶屏写入像素点时，它会一个点一个点地往液晶屏的X方向填充， 填充完一行X方向的像素点后，向Y方向下移一行，X坐标回到起始位置，再往X方向一个点一个点地填充，如此循环直至填充完整个显示窗口。

而屏幕的坐标原点和XY方向都可以根据实际需要使用0X36命令来配置的， 该命令的说明见 图 [液晶扫描模式命令](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id56)。

![[../_resources/野火显示器/b81145d5174f03a777687ad9f12b889f_MD5.jpg]]

0X36命令参数中的MY、MX、MV这三个数据位用于配置扫描方向，因此一共有23 = 8种模式。 ILI9341_GramScan函数就是根据输入的模式设置这三个数据位，并且根据相应的模式更改XY方向的分辨率LCD_X_LENGTH和LCD_Y_LENGTH， 使得其它函数可以利用这两个全局变量获屏幕实际的XY方向分辨率信息；同时， 函数内还设置了全局变量LCD_SCAN_MODE的值用于记录当前的屏幕扫描模式，这在后面计算触摸屏坐标的时候会使用到。 设置完扫描方向后，代码中还调用设置液晶显示窗口的命令CMD_SetCoordinateX/Y（0X2A/0X2B命令）默认打开一个与屏幕大小一致的显示窗口，方便后续的显示操作。

调用ILI9341_GramScan函数设置0-7模式时，各个模式的坐标原点及XY方向如图 [液晶屏的8种扫描模式](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/LCD.html#id57) 所示。

![[../_resources/野火显示器/6b4ff4900f448214b28488c17fb312d5_MD5.jpg]]

其中模式6最符合我们的阅读习惯，扫描方向与文字方向一致，都是从左到右，从上到下，所以本开发板中的大部分液晶程序都是默认使用模式6。

其实模式0、3、5、6的液晶扫描方向都与文字方向一致，比较适合显示文字，只要适当旋转屏幕即可，使得用屏幕四个边沿作为正面看去都有适合的文字显示模式。 而其它模式由于扫描方向与文字方向不一致，要想实现同样的效果非常麻烦，也没有实现的必要。
## 2.6 ILI9341要设置的配置
1. 设置像素传输格式16位还是18位。: Pixel Format Set (3Ah) DBI[2:0]=101

ILI9341 需要连接的线
片选 CS (NE4)：  PG12
读写线 ：WE ：PD5   OE：PD4
命令/数据线 A23 : PE2
高低地址：PE0  PE1
数据线：
PD14 PD15 PD0   PD1
PE7    PE8    PE9   PE10 
PE11  PE12  PE13 PE14
PE15  PD8   PD9  PD10
LCD 初始化  PG11
LCD 背光     PG6
