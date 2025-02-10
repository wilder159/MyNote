> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_45499326/article/details/114605760)

续上篇，继续在 ILI9341 驱动的 LCD 面板上显示各种内容。  
上一篇《ILI9341 的使用之【七】实体面板案例 -arduino 2.4inch TFT Touch Shield》初步完成 LCD 面板显示最基本的线与矩形图像的实际案例。知道了如何控制最底层显示驱动芯片的指令系统完成色块的显示。这篇进一步深入分析如何在 LCD 面板上显示 ASCII 字符，并进一步分析驱动的实现原理及实现。


一、显示的本质
-------

由于 TFT-LCD 显示面板的最小显示单元就是一个个点（DOT)，对于色彩的显示，采用了 RGB 三色原理，因此真正工作时的最小显示单位是像素（Pixel）。这在本系列的前七篇从原理到 ILI9341 的驱动指令与工作模式等都涉及到了这方面的内容。因此，所有面板内容（文字，图形，影像等）的显示实际都由程序最终对每个像素 Pixel 的控制的结果。这就是 LCD 面板显示的本质。  
![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/ed4593a54fe5ddefef4c60a182ee7eb0_MD5.png]]

因此，在明确 LCD 面板显示本质的前提下，再来分析显示驱动的原理就会有清晰方向。显示驱动所有的操作都是针对显示驱动芯片的，使用的是芯片提供的指令系统。因此下面如非特殊说明代码针对的对象都是驱动芯片。

二、硬件环境的准备及软件环境
--------------

本案例的硬件平台继续使用上一篇《ILI9341 的使用之【七】实体面板案例 - arduino 2.4inch TFT Touch Shield》中的硬件，详细的可以参考上一篇。之后不另行说明的话，使用的的硬件环境不再变化。  
硬件环境：arduino UNO + 2.4 寸 TFT TouchShield （ILI9341 驱动）  
软件环境：arduino IDE 集成环境，ILI9341 驱动的 2.4 寸面板驱动程序库（文后会有源码，附件也可下载）。  
驱动程序库和例程需要提前拷到 arduino 的相关目录里，如下：  
![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/2f2d92ba8b25b07b95ba8e3b667361e4_MD5.png]]

三、芯片指令系统的显示操作
-------------

### 1、显存

LCD 显示系统面板上的每个像素都有一个 18bit 的存储区域与之对应。本案例所用的面板像素为 320*240 个 Pixel。因此实际对应的存储区域为 1382400bit(172800byte)。实际，对 LCD 面板的控制就是对这个 172800 字节的存储区域的读与写。下图为存储区与面板的对应示意。  
![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/1a7be2cd8b8e7bf45ebf5b69c8293075_MD5.png]]

对于 ILI9341 芯片，所有的对像素的操作行为就是，指定一个内存区域（指令码 2AH 和指令码 2BH），然后往这个内存区域按某种顺序写入（指令码 2CH）一个个像素的颜色字节 byte 数据。

因此，对 ILI9341 芯片的显示内存的操作就完成了对显示面板的操作。

### 2、颜色数据填充指令

ILI9341 涉及到设定内存区域及写内存的芯片指令如下：  
![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/20b30e70cfc2e8d23670713bb5a88255_MD5.png]]  
![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/805bb55f65b1ac81473a4f9639b8933c_MD5.png]]  
![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/2445c04b594a748e7c94cd9e0435111b_MD5.png]]

以上为芯片指令体系涉及到点阵显示的最基础的三个指令。以上指令的详细解释请参阅《ILI9341 的使用之【五】命令一》。其中 2Ah 指令设定了显示区域的起止列坐标（即横坐标），2Bh 设定了显示区域 的起止行坐标（即纵坐标）。然后用 2Ch 指令按像素的顺序（正常顺序就是按区域内的第一行从小到大，再第二行从小到大，可以通过指令 36h 设置填入顺序）一个字节一个字节依次写入。如果你写入的数据多于所设定区域的像素数量，那么多余的数据将人被自动丢弃。  
这里的每个像素的色颜数据位长最大为 D[17:0]共 18bit，如果选择的是 16bit 模式，则只需要模块初始化时用指令 3Ah 时，将 DPI[2:0]与 DBI[2:0]都设置成 “101” 即可。  
![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/aefebb0cf96b23c77084088f1ca9041f_MD5.png]]

### 3、ILI9341 显示流程及代码

![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/181bf2886fcf48831cb7cf684a2c5371_MD5.png]]

根据上面的流程图，具体的代码如下：

#### 设置显示区块函数 setAddrWindow(int x1, int y1, int x2, int y2)

setAddrWindow() 函数，显存中设定了一个填色区域。该函数在 Adafruit_TFTLCD 类中定义。类的关联源文件在 Adafruit_TFTLCD.h 和 Adafruit_TFTLCD.cpp 中

```c
void Adafruit_TFTLCD::setAddrWindow(int x1, int y1, int x2, int y2) {       
 //arduino 环境int为16bit
  CS_ACTIVE;     //宏，把CS片选引脚设为有效，使显示芯片可以访问
uint32_t t;
//把x1,x2合并到t中
    t = x1;
    t <<= 16;
    t |= x2;
writeRegister32(ILI9341_COLADDRSET, t);    
//设置显示区块的起止列号（横坐标），ILI9341_COLADDRSET为宏定义，在registers.h中定义的宏，值为02A
//发送完命令码后，马上就发送命令参数t。
    t = y1;
    t <<= 16;
    t |= y2;
writeRegister32(ILI9341_PAGEADDRSET, t); 
//设置显示区块的起止行号（纵坐标），ILI9341_PAGEADDRSET为宏定义，在registers.h中定义的宏，值为02B
//发送完命令码后，马上就发送命令参数t。
  CS_IDLE;      //宏，把CS片选引脚设为空闲，显示芯片不接受数据
}

/其中writeRegister32()代码如下：
void Adafruit_TFTLCD::writeRegister32(uint8_t r, uint32_t d) {
  CS_ACTIVE;               // 宏：片选信号处理有效状态
  CD_COMMAND;              //宏：总线处理于命令模式
  write8(r);               //将指令 字节r写入芯片
  CD_DATA;                 //宏：总线切换到数据模式
  delayMicroseconds(10);   //以下把颜色数据d(32bit)写入到芯片中。
  write8(d >> 24);
  delayMicroseconds(10);
  write8(d >> 16);
  delayMicroseconds(10);
  write8(d >> 8);
  delayMicroseconds(10);
  write8(d);
  CS_IDLE;
}
```

#### 单个像素的填充函数 drawPixel(int16_t x, int16_t y, uint16_t color)

该函数出现在 Adafruit_TFTLCD 类中。一次只填充一个像素。关联文件为 Adafruit_TFTLCD.h 和 Adafruit_TFTLCD.cpp

```c
void Adafruit_TFTLCD::drawPixel(int16_t x, int16_t y, uint16_t color) {

   if((x < 0) || (y < 0) || (x >= _width) || (y >= _height)) return; //如果坐标超范围，就不动作直接返回
   CS_ACTIVE;
    setAddrWindow(x, y, _width-1, _height-1); //把显示区块定义得很大，但下面只写一个pixel
    CS_ACTIVE;
    CD_COMMAND; 
    write8(0x2C);     //写2Ch指令
    CD_DATA; 
    write8(color >> 8); write8(color);  //在显示我块里填入一个像素颜色数据。  
  CS_IDLE;
}
```

### 矩形填充函数 fillRect() - 用于放大字体

该函数定义在文件 Adafruit_TFTLCD.cpp 中的 Adafruit_TFTLCD 类里。函数声明如下：  
void Adafruit_TFTLCD::fillRect(int16_t x1, int16_t y1, int16_t w, int16_t h, uint16_t fillcolor)  
由名字可知，该函数用于填充一个矩形块。该函数通过调用下面的 flood（）函数实现矩形区域 色块的填充。

```c
void Adafruit_TFTLCD::fillRect(int16_t x1, int16_t y1, int16_t w, int16_t h, 
  uint16_t fillcolor) {
  int16_t  x2, y2;

  // 如果矩形位置在面板之外，或数据不合理，则不动作，直接返回
  if( (w            <= 0     ) ||  (h             <= 0      ) ||
      (x1           >= _width) ||  (y1            >= _height) ||
     ((x2 = x1+w-1) <  0     ) || ((y2  = y1+h-1) <  0      )) return;

  //矩形与面板有交集，则只在交集面积部分显示。
  if(x1 < 0) { // Clip left
    w += x1;
    x1 = 0;
  }
  if(y1 < 0) { // Clip top
    h += y1;
    y1 = 0;
  }
  if(x2 >= _width) { // Clip right
    x2 = _width - 1;
    w  = x2 - x1 + 1;
  }
  if(y2 >= _height) { // Clip bottom
    y2 = _height - 1;
    h  = y2 - y1 + 1;
  }

  setAddrWindow(x1, y1, x2, y2);  //设置显示的面板区块地址
  flood(fillcolor, (uint32_t)w * (uint32_t)h);  //快速填充  
}
```

#### 多像素颜色填充函数 flood(uint16_t color , uint32_t len)

设置完显示区域后，接下来是往该区块内一个像素一个像素地填入颜色数据，这里以 16bit 的方式编码颜色值。flood 函数是在 Adafruit_TFTLCD 类中定义的方法。关联文件为 Adafruit_TFTLCD.h 和 Adafruit_TFTLCD.cpp  
该函数实现的是块状像素的快速填充。在实际字符显示时，用于对放大的字体进行显示操作。  
/ _为了提高写入的速度，函数对像素数量进行了切块，每块 64 像素，同时通过减少语句的切换，在每个循环里一次写 4 个 pixels。同时，如果每个颜色数据的高 8 位与低 8 位字节相等，那就不再重写总线数据。直接只发送写脉冲，进一步提高效率。_/

```c
void Adafruit_TFTLCD::flood(uint16_t color, uint32_t len) {
  uint16_t blocks;
  uint8_t  i, hi = color >> 8,
              lo = color;

  CS_ACTIVE;       //片选引脚有效信号，使芯片可以访问
  CD_COMMAND;      //总线进入命令模式，总线上的数据被芯片解读为命令
  if (driver == ID_9341) {
    write8(0x2C);    //把8位数据写入总线，并发送
  } else if (driver == ID_932X) {
    write8(0x00); // High byte of GRAM register...
    write8(0x22); // Write data to GRAM
  } else if (driver == ID_HX8357D) {
    write8(HX8357_RAMWR);
  } else {
    write8(0x22); // Write data to GRAM
  }

  // Write first pixel normally, decrement counter by 1
  CD_DATA;     //总线进入数据模式，总线上的数据被芯片解读为数据
  write8(hi);   //写入一个像素的颜色数据的高8位。
  write8(lo);    //再写入该像素的颜色数据的低8位。
  len--;

  blocks = (uint16_t)(len / 64); // 为提高写入的速度，进行分块，每个块64个像素。
  if(hi == lo) {      //如果出现16bits颜色值中高8位等于低8位，则不进行总线的重复写入，而直接发送写脉冲，提高填充速度

    while(blocks--) {
      i = 16; // 64 pixels/block / 4 pixels/pass
      do {
        //一个循环写入4个像素，减少不必要的程序指令时间浪费，提高填充速度
        WR_STROBE; WR_STROBE; WR_STROBE; WR_STROBE; // 2 bytes/pixel
        WR_STROBE; WR_STROBE; WR_STROBE; WR_STROBE; // x 4 pixels
      } while(--i);
    }
    // Fill any remaining pixels (1 to 64)
    for(i = (uint8_t)len & 63; i--; ) {  //len截取最低的8位后，与63求余，得出余下未填充像素个数
      WR_STROBE;
      WR_STROBE;
    }
  } else {    //如果出现的16bits颜色值中高8位不等于低8位，则需要在传送中每次更新总线数据
    while(blocks--) {
      i = 16; // 64 pixels/block / 4 pixels/pass
      do {
        write8(hi); write8(lo); write8(hi); write8(lo);  //一次写入4个像素数据。
        write8(hi); write8(lo); write8(hi); write8(lo);
      } while(--i);
    }
    for(i = (uint8_t)len & 63; i--; ) {  //写入剩余的像素数据。
      write8(hi);  
      write8(lo);
    }
  }
  CS_IDLE;
}


//里面重要的关联函数为 write8()代码如下。
void Adafruit_TFTLCD::write8(uint8_t value) {
  write8inline(value);
}

//Write8inline()为在pin_magic.h中定义的宏：如下
#define write8inline(d) {                          \
    PORTD = (PORTD & B00000011) | ((d) & B11111100); \
    PORTB = (PORTB & B11111100) | ((d) & B00000011); \
    WR_STROBE; }

//WR_STROBE 为pin_magic.h中定义的宏
#define WR_STROBE { WR_ACTIVE; WR_IDLE; }
```

#### 驱动中多次出现的各种宏的定义说明

在 pin_magic.h 中定义的宏

```c
#define RD_ACTIVE        *rdPort &=  rdPinUnset    //RD引脚置0，读有效
 #define RD_IDLE            *rdPort |=  rdPinSet      //RD引脚置1，读空闲
 #define WR_ACTIVE        *wrPort &=  wrPinUnset   //WR引脚置0，写有效
 #define WR_IDLE           *wrPort |=  wrPinSet      //WR引脚置1，写空闲
 #define CD_COMMAND    *cdPort &=  cdPinUnset    //CD引脚置0，总线进入命令模式
 #define CD_DATA           *cdPort |=  cdPinSet       //CD引脚置1，总线进入数据模式
 #define CS_ACTIVE         *csPort &=  csPinUnset    //CS引脚置0，片选有效，芯片可访问
 #define CS_IDLE            *csPort |=  csPinSet       //CS引脚置1，片选空闲，芯片不可访问
```

以上宏涉及到的变量定义在 Adafruit_TFTLCD.h 文件中，具体如下：

```c
volatile uint8_t *csPort    , *cdPort    , *wrPort    , *rdPort;
  uint8_t           csPinSet  ,  cdPinSet  ,  wrPinSet  ,  rdPinSet  ,   csPinUnset,  cdPinUnset,  wrPinUnset,  rdPinUnset,     _reset;
```

以上宏涉及的变量赋值，出现在 Adafruit_TFTLCD.cpp

```c
csPort     = portOutputRegister(digitalPinToPort(cs));
    cdPort     = portOutputRegister(digitalPinToPort(cd));
    wrPort     = portOutputRegister(digitalPinToPort(wr));
    rdPort     = portOutputRegister(digitalPinToPort(rd));
  csPinSet   = digitalPinToBitMask(cs);
  cdPinSet   = digitalPinToBitMask(cd);
  wrPinSet   = digitalPinToBitMask(wr);
  rdPinSet   = digitalPinToBitMask(rd);
  csPinUnset = ~csPinSet;
  cdPinUnset = ~cdPinSet;
  wrPinUnset = ~wrPinSet;
  rdPinUnset = ~rdPinSet;
```

如果不想深究这些变量的含义。那么上方的这些宏的定义其实就是把对应的引脚置 1 或置 0。为了提高效率，这里用到了 arduino 底层的端口 / 引脚操作宏 digitalPinToPort， portOutputRegister，digitalPinToBitMask。在这里简单列出这三个宏的定义。详细的说明，读者可以自行百度，或有需要的话再另起一篇文单进行说明：  
/* 关于系统的几个重要宏定义，如下：

```c
#define digitalPinToPort(P) ( pgm_read_byte( digital_pin_to_port_PGM + (P) ) )  
//返回的是引脚P对应的PORT号。
#define portOutputRegister(P) ( (volatile uint8_t *)( pgm_read_word( port_to_output_PGM + (P))) )
// 返回的是PORT号对应的输出寄存器地址（把16位地址指针转成了8位地址指针）
#define portInputRegister(P) ( (volatile uint8_t *)( pgm_read_word( port_to_input_PGM + (P))) )
//返回的是PORT号对应的输入寄存器地址（把16位地址指针转成了8位地址指针）
#define portModeRegister(P) ( (volatile uint8_t *)( pgm_read_word( port_to_mode_PGM + (P))) )
//返回的是PORT号对应的模式寄存器地址（把16位地址指针转成了8位地址指针）
#define _BV(n) (1<<(n))
#define digitalPinToBitMask(P) ( pgm_read_byte( digital_pin_to_bit_mask_PGM + (P) ) )
//返回的是 _BV(n) 宏，即1<<n
```

*/

### 4、颜色编码

通过上方两个对像方法 setAddrWindow 和 flood。就可以实现像素的填充。填充涉及到的颜色编码的方法如下：  
由于面板模块参数说明里的对应描述  
![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/b5fc40b1af602167f16e9470c049f190_MD5.png]]

可以知道，这里的 Pixel Format Set (3Ah) 指令里的像素定义为 16bit。即 DBI[2:0] 与 DPI[2:0] 参数的最值都为”B101“对应如下格式  
![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/291259a8cd8c30a64e73a028a5040236_MD5.png]]

由于通信接口是 8 位的接品，因此总线一次是传送 8bit 数据的。所以在实际传送数据到 RAM 时是，一共传两次，一次 8Bit，共 16bit，先传高 8 位再传低 8 位。  
例如：红色的像素数据为 D[15:0]=11111 000 000 00000=f800h (前 5 位为红色编码区，中间 6 位为绿色编码区，后 5 位为兰色编码区）  
![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/e4599c17297f855eedc7e21b3f7fdb3f_MD5.png]]

理解了上面的底层显示功能后，接下来就可以进入字符显示的实质性功能实现了。

四、ASCII 编码到点阵字库的显示
------------------

```
有了上面这么多的内容铺垫，到这编终于要进入本篇的核心内容，字符的显示了。本章节又会涉及到几个重要文件：
```

C:\Users\loneve\Documents\Arduino\libraries\Adafruit_TFTLCD\glcdfont.c ASCII 5*7 字库  
C:\Program Files (x86)\Arduino\hardware\arduino\avr\cores\arduino\Print.h  
C:\Program Files (x86)\Arduino\hardware\arduino\avr\cores\arduino\Print.cpp

由于底层的 LCD 显示驱动类 class Adafruit_GFX : public Print 直接继承自 Print 类的。又由于 Printr 类的定义中，对写字符的最基础方法 write() 是定义为纯虚函数，因此整个驱动通过在 Adafruit_GFX 类中对 write() 方法进行重写，使得驱动充分地利用了 arduino 编译环境中的 Print 类的字符处理能力，令驱动变得逻辑层次更清晰。

### 1、ASCII 码表

相信 ASCII 码表大家都非常熟悉了，这里不再多说，直接上一张表，方便接下来查询使用。  
![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/2b394534ff74958cd8b914fee59d1efb_MD5.png]]

ASCII 码表中每个字符的二进制编码，恰恰也是字库数组的索引码。因此在具体函数中 drawChar() 函数是通过该二进制码实现对字库的索引的。这里选知道一下这个规则，具体在后面的代码解释中会涉及到。

### 2、字库操作

字库文件是在驱动软件包里的 glcdfont.c。具体在下载该包时可以看到。其结构如下：

```c
static const unsigned char font[] PROGMEM = {
    0x00, 0x00, 0x00, 0x00, 0x00,
    0x3E, 0x5B, 0x4F, 0x5B, 0x3E,
    0x3E, 0x6B, 0x4F, 0x6B, 0x3E,
    0x1C, 0x3E, 0x7C, 0x3E, 0x1C,
    0x18, 0x3C, 0x7E, 0x3C, 0x18,
   ................................................
0x7F, 0x08, 0x08, 0x08, 0x7F,
.......................................
   0x30, 0x40, 0xFF, 0x01, 0x01,
    0x00, 0x1F, 0x01, 0x01, 0x1E,
    0x00, 0x19, 0x1D, 0x17, 0x12,
    0x00, 0x3C, 0x3C, 0x3C, 0x3C,
    0x00, 0x00, 0x00, 0x00, 0x00    // #255 NBSP
};
```

实际字库为一个数据结构。为了方便阅读，才写成 5 个字节一行的格式。一共 255 行，对应 255 个 ASCII 码。每一行 5 个字节，对应每个字符的 5 列点阵。

拿显示 “H” 来展示整个显示的过程：首先‘H’的 ASCII 码十进制为 72。则在 font 数组里从第一行开始读到第 72 行（每行 5 个字节即 72_5=360），在计算数组里的 H 字符的第一列地址时就是 font+(72_5)。在程序分析里会有该语句。上述过程如下图示意，就比较直观了。

![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/e6e98039a42e8247f1e4c4fcef51ee5a_MD5.png]]

### 3、字符显示代码分析

![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/768630e45341035f3d8362014b142427_MD5.png]]

上面核心功能就是 write 函数和 drawChar 函数。Write 函数有多个重载函数形态但最后都调用了 write(uint8_t c) 这个函数，完成单个字符的显示功能，实现了系统 Print 类与 LCD 显示系统的接口，drawChar 函数完成从 ASCII 码到字库点阵数据的检索，然后读出点阵数据依据一定顺序去调用最终的像素填充程序实现字符显示。填充程序就是第三章第 3 点讨论的 drawPixel() 函数及 fillRect() 函数等。  
字符点阵字节的读出与显示的顺序为：先发送 X1-D0，X1-D1…X1-D7 ，然后是 X2-D0，X2-D1…X2-D7。然后是 X3-D0，X3-D1…X3-D7。依此类推 X5-D0，X5-D1…X5-D7。如下图示：

![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/7de0827cfcdf29aa7127d4861fa21ea6_MD5.png]]

五、显示字符例程 DisplayString.ino
--------------------------

以下内容是这篇字符显示的入口程序。相信在看完前述部分的内容，对下面这个例程应该是很容易理解的。这里要注意一点的是，本案例的面板对应的型号类别为 BREAKOUT BOARD 。由于在 Adafruit_GFX.h 中有一个关于这个开关的宏定义 #define USE_ADAFRUIT_SHIELD_PINOUT 。直接把这个宏屏蔽。

displayString.ino

```c
#include <Adafruit_GFX.h>    // Core graphics library
#include <Adafruit_TFTLCD.h> // Hardware-specific library

// The control pins for the LCD can be assigned to any digital or
// analog pins...but we'll use the analog pins as this allows us to
// double up the pins with the touch screen (see the TFT paint example).

#define LCD_CS A3 // Chip Select goes to Analog 3
#define LCD_CD A2 // Command/Data goes to Analog 2
#define LCD_WR A1 // LCD Write goes to Analog 1
#define LCD_RD A0 // LCD Read goes to Analog 0

#define LCD_RESET A4 // Can alternately just connect to Arduino's reset pin

// When using the BREAKOUT BOARD only, use these 8 data lines to the LCD:
// For the Arduino Uno, Duemilanove, Diecimila, etc.:
//   D0 connects to digital pin 8  (Notice these are
//   D1 connects to digital pin 9   NOT in order!)
//   D2 connects to digital pin 2
//   D3 connects to digital pin 3
//   D4 connects to digital pin 4
//   D5 connects to digital pin 5
//   D6 connects to digital pin 6
//   D7 connects to digital pin 7
// For the Arduino Mega, use digital pins 22 through 29
// (on the 2-row header at the end of the board).

// Assign human-readable names to some common 16-bit color values:
#define	BLACK   0x0000
#define	BLUE    0x001F
#define	RED     0xF800
#define	GREEN   0x07E0
#define CYAN    0x07FF
#define MAGENTA 0xF81F
#define YELLOW  0xFFE0
#define WHITE   0xFFFF

Adafruit_TFTLCD tft(LCD_CS, LCD_CD, LCD_WR, LCD_RD, LCD_RESET);
// If using the shield, all control and data lines are fixed, and
// a simpler declaration can optionally be used:
// Adafruit_TFTLCD tft;

Adafruit_GFX_Button buttons[15];
void setup(void) {
  Serial.begin(9600);
  Serial.println(F("TFT LCD test"));

#ifdef USE_ADAFRUIT_SHIELD_PINOUT
  Serial.println(F("Using Adafruit 2.4\" TFT Arduino Shield Pinout"));
#else
  Serial.println(F("Using Adafruit 2.4\" TFT Breakout Board Pinout"));
#endif

  Serial.print("TFT size is "); Serial.print(tft.width()); Serial.print("x"); Serial.println(tft.height());

  tft.reset();

  uint16_t identifier = tft.readID();
  if(identifier==0x0101)
      identifier=0x9341;
  
  if(identifier == 0x9325) {
    Serial.println(F("Found ILI9325 LCD driver"));
  } else if(identifier == 0x4535) {
    Serial.println(F("Found LGDP4535 LCD driver"));
  }else if(identifier == 0x9328) {
    Serial.println(F("Found ILI9328 LCD driver"));
  } else if(identifier == 0x7575) {
    Serial.println(F("Found HX8347G LCD driver"));
  } else if(identifier == 0x9341) {
    Serial.println(F("Found ILI9341 LCD driver"));
  } else if(identifier == 0x8357) {
    Serial.println(F("Found HX8357D LCD driver"));
  } else {
    Serial.print(F("Unknown LCD driver chip: "));
    Serial.println(identifier, HEX);
    Serial.println(F("If using the Adafruit 2.4\" TFT Arduino shield, the line:"));
    Serial.println(F("  #define USE_ADAFRUIT_SHIELD_PINOUT"));
    Serial.println(F("should appear in the library header (Adafruit_TFT.h)."));
    Serial.println(F("If using the breakout board, it should NOT be #defined!"));
    Serial.println(F("Also if using the breakout, double-check that all wiring"));
    Serial.println(F("matches the tutorial."));
    return;
  }

  tft.begin(identifier);

 
}

void loop(void) {
   tft.fillScreen(BLACK);

  unsigned long start = micros();
  tft.setCursor(0, 0);
  
  tft.setTextColor(RED);  tft.setTextSize(1);
  tft.println("Hello World!");
  tft.println(01234.56789);
  tft.println(0xDEADBEEF, HEX);
  tft.println();
  tft.println();
  tft.setTextColor(GREEN); tft.setTextSize(2);
   tft.println("Hello World!");
  tft.println(01234.56789);
  tft.println(0xDEADBEEF, HEX);
  tft.println();
  tft.println();
  
  tft.setTextColor(BLUE);    tft.setTextSize(3);
   tft.println("Hello World!");
  tft.println(01234.56789);
  tft.println(0xDEADBEEF, HEX);
  
  tft.setTextColor(WHITE);    tft.setTextSize(4);
   tft.println("Hello!");
   tft.setTextColor(YELLOW);    tft.setTextSize(5);
   tft.println("Hello!");
   tft.setTextColor(RED);    tft.setTextSize(6);
   tft.println("Hello!");
  tft.println();
  tft.println();
 
  delay(1000);delay(1000);delay(1000);delay(1000);delay(1000);
}
```

正确运行后的显示效果：  
![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/799e45601e16e4452574d33b5bd44fd4_MD5.png]]

六、关于驱动的源程序
----------

驱动已压缩，用时解开后，把整个文件夹拷贝到 【文档】目录下的【arduino/librarys】目录就可以了。不同的系统可能文档目录的叫法不同，windows 下叫【文档】，MacOS 叫【文稿】，但安装完 arduino 后都 会在【文档】或【文稿】下有【arduino/librarys】目录：

![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/de8972cb66ca1b7bd110b5d09e95f0df_MD5.png]]

然后在 arduino UNO 里就可以看到示例：  
![[../../../../_resources/ILI9341的使用之【八】ASCII字符显示及驱动分析/142e687d20ecf05ac01745ebfa0316a4_MD5.png]]  
压缩文件请到博客的资源里去下载。[点这里下载 Adafruit_TFTLCD.zip](https://download.csdn.net/download/weixin_45499326/15690315)