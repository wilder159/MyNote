> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_45499326/article/details/113678283)
### 0.1.1 一、RGB 接口的选择

ILI9341 有两种 RGB 接口，分别是 **DE 模式**和 **SYNC 模式**。通过设置 RCM[1:0]bits 位（此为指令 B0h-RGB Interface Signal Control 的参数），实现接口类型的选择。当 RCM[1:0]=‘10’时，包含 VSYNC、HSYNC、DOTCLK、DE、D[17:0] 些引脚的 DE 模式被选择。当 RCM[1:0]='11’时，包含 VSYNC、HSYNC、DOTCLK，D[17:0] 引脚的 [SYNC](https://so.csdn.net/so/search?q=SYNC&spm=1001.2101.3001.7020) 模式被选择。（见下表）  
**说明： DE（Data Enable）；SYNC（SYNChronization)**  
**注意：RGB 模式是工作在串行接口模式下的。**  
**象素格式：** ILI9341 支持多种的象素格式，这可以通过设置 “Pixel Format set (3Ah)” 指令中的 **DPI[2:0]** 位和 “Interface Control（F6h）” 指令中的 **RIM** 位进行选择。**RCM[1:0]** 是在 "RGB Interface Signal Control(B0h)" 指令中进行设置。  
![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/be381c853e2c67ab7deca197c941205f_MD5.png]]  
下图为 SYNC 模式下的像素颜色格式：  
![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/13b82aac59a3e3159050e8974cc7e73e_MD5.png]]  
像素时钟（DOTCLK) 会一直不停运行。当 DOTCLK 上升沿时会引入 VSYNC，HSYNC，DE 和 D[17:0] 状态值。  
VSYNC(垂直同步) 引脚，被用于标识什么时候收到一个新的显示帧数据。该引脚是**低电平有效**。并且该 状态在 DOTCLK 上升沿时被读入显示模块。

HSYNC(水平同步）引脚，被用于标识什么时候收到一个帧的新行。该引脚是**低电平有效**。并且该 状态在 DOTCLK 上升沿时被读入显示模块。

在 DE 模式中，DE（Data Enable) 信号用来标识什么时候接收到了将被显示的 RGB 信息。该信号是**高电平有效**，它的状态在 DOTCLK 上升沿时被显示模块读取。

D[17:0] 被用来传送用来显示的图像信息。当 DE=0 并且在 DOTCLK 上升沿时被读入。

在 SYNC 模式中，有效的显示数据是经由 D[17:0] 以像素数据输入的。该数据的格式是根据 HSYNC 信号的 HFP/HBP 设置的和 VSYNC 信号的 VFP/VBP 设置的。（vFP/VBP，HFP/HBP 的含义如下图所述，设置的指令为 B5h-Blanking Porch Control）  
在这两种 RGB 模式中，输出的显示数据首先被写入 GRAM 中，然后再从 GRAM 中根据灰度数据输出相应的源极电压。  
![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/692c32a1ec868ce7f0b43f665763d1dd_MD5.png]]  
![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/eb0b05bab066d54c76ac28d90a2ea77a_MD5.png]]  
一般 240*320 分辨率的面板所使用的标准参数值为，6.35MHZ 时钟频率和 70HZ 的帧刷新率。  
注意：  
1、Vertical 垂直的周期 (一帧 one frame）的值 为：Vsync + VBP + VAdr + VFP  
2、Horizontal 水平周期（一线 one line）的值为： Hsync + HBP + HAdr + HFP.  
3、在所有主控制器与显示模块之间传送像素值的时间里，控制信号 PCLK 和 Hsync 将按规定传送

同时也要确保  
(Number of PCLK per 1 line) ≥ (Number of RTN clock) x Division ratio (DIV) x PCDIV

RGB 接口操作显示控制时钟设置示例：  
使用 DPI 的寄存器显示操作与内部时钟 PCLKD 同步，PCLKD 是通过对 DOTCLK 分频产生的。  
PCDIV[5:0]: 在内部时钟 PCLKD 的一个高 / 低周期间的 DOTCLK 时钟的个数。PCDIV 是指令 B6h（Display Function Control) 定义的参数。  
![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/651558f7305035d0f43fd1f90f98bc13_MD5.png]]

PCDIV 指定了 DOTCLK 脉冲的分频率，它决定于 PCLKD 的频率与内部 615KHZ 振荡时钟有多少差异。根据下面的约束设置 PCDIV：  
(Number of PCLK in 1H) ≥ (Number of RTN clock) x Division ratio (DIV) x PCDIV

**设置实例:** 设置帧刷新率为 70Hz:  
**Internal Clock**  
Internal Oscillation Clock: 615KHz  
DIV[1:0] = 00 (1 倍的时钟 615KHZ)  
RTN[4:0] = 1b (27 clocks)  
FP = 7’h2 (2 lines), BP = 7’h2 (2 lines), NL = 6’h27 (320 lines)  
（说明：VFP，VBP 指令 B5h Blanking Porch Control 的参数。NL 为指令 B6h Display Function Control 的参数）  
Frame Rate ： 70.30Hz

**DOTCLK**  
HSYNC = 10 CLK  
HBP = 20 CLK  
HFP=10 CLK  
（说明：HFP，HBP 指令 B5h Blanking Porch Control 的参数。）  
70Hz x (2 + 320 + 2) lines x (10 + 20 + 240 + 10) clocks = 6.35MHz  
DOTCLK frequency = 6.35MHz  
6.35 MHz / 615KHz = 10.32 I Set PCDIV so that PCLK is divided by 10.  
external fosc = 6.35 MHz / 10 = 635KHz  
PCDIV = [6.35MHz / 635KHz) / 2 ] - 1 = 4  
PCDIV[5:0] = 6’h04 (10 DOTCLK)

### 0.1.2 二、RGB 接口时序

**以下为 18/16bit RGB 接口模式的时序图解：**  
![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/1d370ffafdcff602c280678954b47a3d_MD5.png]]  
![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/46df5dbe37b457a4f12dd6bbbded7996_MD5.png]]  
注意 1：在 RGB 接口的 SYNC 模式下，不需要 DE 信号  
注意 2：”interface Mode Control(B0h)“命令的这几个参数取值：VSPL，HSPL，DPL，EPL 都 为 0。

**下面为 6-bit RGB 接口模式的进序图解：**  
![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/10021459545d27ab02f73f8579422556_MD5.png]]  
![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/9c4d64f4897cbc6f4793fcea4ed6bb37_MD5.png]]  
注意 1：在 RGB 接口的 SYNC 模式下，不需要 DE 信号  
注意 2：”interface Mode Control(B0h)“命令的这几个参数取值：VSPL，HSPL，DPL，EPL 都 为 0。  
注意 3：在 6-bit RGB 接口模式下，一个像素的每个点（R,G,B）是与 DOTCLK 同步传送  
注意 4：: 在 6 位 RGB 接口模式下，将 VSYNC、HSYNC、DE 的周期设置为 DOTCLK 的 3 倍。

### 0.1.3 三、VSYNC 接口

ILI9341 支持 VSYNC 接口模式，该模式是由引脚 VSYNC 提供帧同步信号。该模式是通过 8080-I/8080-II 系列 MCU 接口来显示动图。当 VSYNC 接口被选择用来显示一个动图时，GRAM 的最小更新速度是有限制的，并且 VSYNC 接口是由 DM[1:0]=‘10’ 和 RM='0’两个参数设置使能的。  
以上提到的两个参数由指令 F6h(Interface Control) 中的 DM[1:0]（Display Operation Mode）参数和 RM(Inerface for RAM Access) 参数设置，  
![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/98ef99b55cd715d8c19728630b7bb62f_MD5.png]]  
在 VSYNC 模式下，显示操作是与内部时钟和 VSYNC 引脚的输入相同步的。帧的刷新率决定于 VSYNC 信息号的脉冲速率。所有的显示数据都存储在 GRAM 使动图显示的传送需要的总数据最小化。  
![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/8f26c2911cbb38d2ddea0c46742a3601_MD5.png]]  
![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/895902c074f2ac9b38808b1370f2ee3b_MD5.png]]  
VSYNC 接口对经由系统接口向内部 GRAM 写入数据的速度是有最小速度限制的。通过下面的公式进行计算：  
Internal clock frequency (fosc.) [Hz] = FrameFrequency x (DisplayLine (NL) + FrontPorch (VFP) + BackPorch (VBP)) x ClockCyclePerLines (RTN) x FrequencyFluctuation.

![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/be8f934b691e893a4fe3451ab6ca6511_MD5.png]]  
注意：当 RAM 写操作不是从 VSYNC 的下降沿开始的，则从 VSYNC 的下降沿开始到 RAM 写操作开始的这段时间也必须计算在内。

以下是一个 GRAM 最小写速度和 VSYNC 接口模式下内部时钟频率的实例：  
![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/e4324e2c369b19746d3be9995c3afd77_MD5.png]]  
在计算内部时钟频率时，必须要考虑到振荡器的变化。在上面的例子中，考虑到计算的内部时钟频率有 ±10% 的裕度变化并且要确保在一个垂 VSYNC 周期内完成显示操作。频率变化的原因主要有大规模集成电路的制造工艺、室温、外部电阻和 VCI 电压的变化。

Minimum speed for RAM writing [Hz] > 240 x 320 x 748K / [ (2 + 320 – 2)lines x 27clocks] ≒ 6.65 MHz

计算上述理论值的前提是 ILI9341 在 VSYNC 下降沿开始写入内部 GRAM。物理显示行和执行数据写入操作的 GRAM 地址之间必须至少有 2 行空白。GRAM 以 6.35MHZ 或更高的写速度将保证 GRAM 的写入操作能在 ILI9341 开始在屏幕上显示 GRAM 数据之前完成，并且能在不出现闪烁的前提下重定整个屏幕。

**使用 VSYNC 接口时注意如下：**  
1、最小的 GRAM 写速率必须被满足。并且必须要考虑频率的变化。  
2、帧的显示速率决定于 VSYNC 信号，以及 VSYNC 的周期必须长于完全显示的扫描周期。  
3、当从内部时钟操作模式（DM[1:0]=‘00’) 切换到 VSYNC 接口模式，或反相。这个切换应从下一个 VSYNC 周期 ，也即，在一个帧完全显示 后。  
4、在垂直同步界面模式下，不支持部分显示、垂直滚动和隔行扫描功能。

![[../../../../_resources/ILI9341的使用之【四】RGB接口操作详解/84d9c36ee4d100417f9dd6daa860a4e9_MD5.png]]

### 0.1.4 四、色深转换查询表 (省略）

### 0.1.5 五、DDRAM（Display Data RAM）

ILI9341 集成了 240x320x18 位图形型静态 RAM。这 172,800 字节的内存允许存储一幅 240xRGBx320 图像，18 位分辨率 (262K-color)。当对帧存储器的同一位置同时进行面板显示读和接口读 / 写时，显示器上不会有异常可见效果。