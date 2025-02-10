> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_45499326/article/details/113992657)

前面共用了六个篇幅，把 TFT-LCD 原和驱动芯片 ILI9341 的接口系统与命令系统做了一个比较详细的介绍与解释。上面几篇的很多细节由于需要在使用过程中去验证。因此有此专有名语的解释以及功能的介绍相可能不十分准备与完整，在接下来的应用实例 中如果有发现此类情况会一一做出更正或完善。以希望读者能在过程中指证以期共同完善。也因此上面七篇完全可以做为指南手册性质的资料，在应用过程中随时查阅参考。

[《ILI9341 的使用之【一】TFT-LCD 原理（转载）》](https://blog.csdn.net/weixin_45499326/article/details/113567280)  
[《ILI9341 的使用之【二】ILI9341 介绍》](https://blog.csdn.net/weixin_45499326/article/details/113571104)  
[《ILI9341 的使用之【三】ILI9341 系统通信接口模式操作详解》](https://blog.csdn.net/weixin_45499326/article/details/113620791)  
[《ILI9341 的使用之【四】RGB 接口操作详解》](https://blog.csdn.net/weixin_45499326/article/details/113678283)  
[《ILI9341 的使用之【五】命令一》](https://blog.csdn.net/weixin_45499326/article/details/113718511)  
[《ILI9341 的使用之【六】命令二》](https://blog.csdn.net/weixin_45499326/article/details/113837122)  
[《ILI9341 的使用之【七】实体面板案例 - arduino 2.4inch TFT Touch Shield》](https://blog.csdn.net/weixin_45499326/article/details/113992657)  
[《ILI9341 的使用之【八】ASCII 字符显示及驱动分析》](https://blog.csdn.net/weixin_45499326/article/details/114605760)  
[《ILI9341 的使用之【九】BG2312 字库》](https://blog.csdn.net/weixin_45499326/article/details/114776453)

接下来的几篇是围绕 ILI9341 芯片的相关应用案例的介绍以及分析。内容会涉及到具体的面板，例程分板，接口分析。通过更直观的方式实现基于 ILI9341 芯片的 LCD 面板的应用。

### 0.1.1 一、arduino 2.4inch TFT Touch Shield

该板是我在网上淘到到的一块基于 ILI9341 的 2.4 英寸，带触控功能的面板。这块板的设计以 Shield 的形态很好地与目前流行的 arduino UNO 开发板契合。使我们不需要纠结于硬件底层，而只需要关注应用实现。  
![[../../../../_resources/ILI9341的使用之【七】实体面板案例-arduino 2.4inch TFT Touch Shield/7b2fd2ae11010d7fb0ad235f1299c198_MD5.png]]  
插入 arduino UNO 后是这样的  
![[../../../../_resources/ILI9341的使用之【七】实体面板案例-arduino 2.4inch TFT Touch Shield/90959f3b890d9b85a3ba9eb21763099c_MD5.png]]

### 0.1.2 二、面板的使用参数

![[../../../../_resources/ILI9341的使用之【七】实体面板案例-arduino 2.4inch TFT Touch Shield/c20122f66965803edbeed116879a3ff7_MD5.png]]

![[../../../../_resources/ILI9341的使用之【七】实体面板案例-arduino 2.4inch TFT Touch Shield/c5c962c706eef61f5eac1ceee52c825c_MD5.png]]

![[../../../../_resources/ILI9341的使用之【七】实体面板案例-arduino 2.4inch TFT Touch Shield/b192c95df5d2fb41543414031a0a32dd_MD5.png]]

![[../../../../_resources/ILI9341的使用之【七】实体面板案例-arduino 2.4inch TFT Touch Shield/4861204ad2a705429355b523eaa56889_MD5.png]]

### 0.1.3 三、重点说明

根据前面六篇对 ILI9341 的介绍，该 2.4 英寸面板，从硬件搭建上可以看出以下几点：

#### 0.1.3.1 1、通讯接口

MCU 通讯接口模式上使用的是并行 8 位接口模式。涉及到的相关硬件设置如下：  
![[../../../../_resources/ILI9341的使用之【七】实体面板案例-arduino 2.4inch TFT Touch Shield/6ab1a1ca15d9598df079968df32d4201_MD5.png]]  
具体可 以参见[《ILI9341 的使用之【二】ILI9341 介绍》](https://blog.csdn.net/weixin_45499326/article/details/113571104)  
该面板已经在硬件上实现了 ILI9341 芯片的 IM0,IM1,IM2,IM3 四个引脚的低电平连接。使 ILI9341 芯片工作在 8080 MCU 8-bit 总线接口的通信模式下。如上表的红框的定义描述。因此在整个 2.4 寸的面板对外引脚定义上，也是遵循 8-bit 接口模式的。具本内容见上面 “Arduono 使用接线说明”。

#### 0.1.3.2 2、显示接口

根据面板模块参数说明里的对应描述，在下面的例程中在 Lcd_init() 函数实现显示接口的相应设置：  
![[../../../../_resources/ILI9341的使用之【七】实体面板案例-arduino 2.4inch TFT Touch Shield/1242c051681883ce29d8d247832c4bd0_MD5.png]]  
可以知道，这里的 Pixel Format Set (3Ah) 指令里的像素定义为 16bit。即 DBI[2:0] 与 DPI[2:0] 参数的最值都为”B101“对应如下格式  
![[../../../../_resources/ILI9341的使用之【七】实体面板案例-arduino 2.4inch TFT Touch Shield/ab8b9abbe43189cd5eb56cea74ce401d_MD5.png]]

由于通信接口是 8 位的接品，因此总线一次是传送 8bit 数据的。所以在实际传送数据到 RAM 时是，一共传两次，一次 8Bit，共 16bit，先传高 8 位再传低 8 位。  
例如：红色的像素数据为 D[15:0]=11111 000 000 00000=f800h (前 5 位为红色编码区，中间 6 位为绿色编码区，后 5 位为兰色编码区）

<table><thead><tr><th>----</th><th>D15</th><th>D14</th><th>D13</th><th>D12</th><th>D11</th><th>D10</th><th>D9</th><th>D8</th><th>D7</th><th>D6</th><th>D5</th><th>D4</th><th>D3</th><th>D2</th><th>D1</th><th>D0</th></tr></thead><tbody><tr><td>定义</td><td>红</td><td>色</td><td>编</td><td>码</td><td>区</td><td>绿</td><td>色</td><td>编</td><td>码</td><td>区</td><td></td><td>兰</td><td>色</td><td>编</td><td>码</td><td>区</td></tr><tr><td>纯红</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr><tr><td>纯绿</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr><tr><td>纯兰</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr></tbody></table>

####0.1.1 一、arduino 2.4inch TFT Touch Shield

该板是我在网上淘到到的一块基于 ILI9341 的 2.4 英寸，带触控功能的面板。这块板的设计以 Shield 的形态很好地与目前流行的 arduino UNO 开发板契合。使我们不需要纠结于硬件底层，而只需要关注应用实现。  
![](https://i-blog.csdnimg.cn/blog_migrate/d85aba22dac7259dcbd35ec8a8adeb69.png)  
插入 arduino UNO 后是这样的  
![](https://i-blog.csdnimg.cn/blog_migrate/469b67feb386b1f491a64c711e96266f.png)

### 0.1.4 二、面板的使用参数

![](https://i-blog.csdnimg.cn/blog_migrate/84e2fe3e558a4fe9caf4a3ac71f17b9b.png)

![](https://i-blog.csdnimg.cn/blog_migrate/2885beab707f2ef459450faf8b35a73f.png)

![](https://i-blog.csdnimg.cn/blog_migrate/306932215d639c03688e6148c5440a3a.png)

![](https://i-blog.csdnimg.cn/blog_migrate/94761ec94ec827ee557653818f18d4c1.png)

### 0.1.5 三、重点说明

根据前面六篇对 ILI9341 的介绍，该 2.4 英寸面板，从硬件搭建上可以看出以下几点：

#### 0.1.5.1 1、通讯接口

MCU 通讯接口模式上使用的是并行 8 位接口模式。涉及到的相关硬件设置如下：  
![](https://i-blog.csdnimg.cn/blog_migrate/2ba04e89ae568dd2a8206f9ab1573376.png)  
具体可 以参见[《ILI9341 的使用之【二】ILI9341 介绍》](https://blog.csdn.net/weixin_45499326/article/details/113571104)  
该面板已经在硬件上实现了 ILI9341 芯片的 IM0,IM1,IM2,IM3 四个引脚的低电平连接。使 ILI9341 芯片工作在 8080 MCU 8-bit 总线接口的通信模式下。如上表的红框的定义描述。因此在整个 2.4 寸的面板对外引脚定义上，也是遵循 8-bit 接口模式的。具本内容见上面 “Arduono 使用接线说明”。

#### 0.1.5.2 2、显示接口

根据面板模块参数说明里的对应描述，在下面的例程中在 Lcd_init() 函数实现显示接口的相应设置：  
![](https://i-blog.csdnimg.cn/blog_migrate/17168204da229caa6e048d56f8361197.png)  
可以知道，这里的 Pixel Format Set (3Ah) 指令里的像素定义为 16bit。即 DBI[2:0] 与 DPI[2:0] 参数的最值都为”B101“对应如下格式  
![](https://i-blog.csdnimg.cn/blog_migrate/e11b8f160e898009e235cef3bf0ac1e1.png)

由于通信接口是 8 位的接品，因此总线一次是传送 8bit 数据的。所以在实际传送数据到 RAM 时是，一共传两次，一次 8Bit，共 16bit，先传高 8 位再传低 8 位。  
例如：红色的像素数据为 D[15:0]=11111 000 000 00000=f800h (前 5 位为红色编码区，中间 6 位为绿色编码区，后 5 位为兰色编码区）

<table><thead><tr><th>----</th><th>D15</th><th>D14</th><th>D13</th><th>D12</th><th>D11</th><th>D10</th><th>D9</th><th>D8</th><th>D7</th><th>D6</th><th>D5</th><th>D4</th><th>D3</th><th>D2</th><th>D1</th><th>D0</th></tr></thead><tbody><tr><td>定义</td><td>红</td><td>色</td><td>编</td><td>码</td><td>区</td><td>绿</td><td>色</td><td>编</td><td>码</td><td>区</td><td></td><td>兰</td><td>色</td><td>编</td><td>码</td><td>区</td></tr><tr><td>纯红</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr><tr><td>纯绿</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr><tr><td>纯兰</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr></tbody></table>

#### 0.1.5.3 3、指令

涉及到的主要指令逻辑为列地址设置 2Ah, 行地址设置 2Bh, 内存写入 2Ch：  
![](https://i-blog.csdnimg.cn/blog_migrate/24611e0cdb08afef70ac34688a7c211a.png)  
![](https://i-blog.csdnimg.cn/blog_migrate/fda81f2c91c8b724ef8a6ab4b4486c8d.png)  
涉及到的物理总线写写入时序为，在下面的例程中由 Lcd_Write_Bus() 实现该写入时序。

![](https://i-blog.csdnimg.cn/blog_migrate/8c81bc0037827ea32656290b94c42c82.png)

### 0.1.6 3、指令

涉及到的主要指令逻辑为列地址设置 2Ah, 行地址设置 2Bh, 内存写入 2Ch：  
![[../../../../_resources/ILI9341的使用之【七】实体面板案例-arduino 2.4inch TFT Touch Shield/19c37784235b49cf855d3b5806f65957_MD5.png]]  
![[../../../../_resources/ILI9341的使用之【七】实体面板案例-arduino 2.4inch TFT Touch Shield/fd0d0a481ad7eb2b406aa640d95319b8_MD5.png]]  
涉及到的物理总线写写入时序为，在下面的例程中由 Lcd_Write_Bus() 实现该写入时序。

![[../../../../_resources/ILI9341的使用之【七】实体面板案例-arduino 2.4inch TFT Touch Shield/6e44bee966ed3c36655cc4ad1ab69494_MD5.png]]

### 0.1.7 四、驱动应用案例一

第一个完整例程如下，在程序里已经对所有功能做了相应注释，请结合这前的工作原理描述和指令详细说明去理解。该例程在 arduino UNO 上运行通过。

```c
//本例程做为基于ILI9341芯片驱动的2.4寸面板的第一个案例。采用了最基本的显示模式。以ILI9341指令系统中的
//MCU to memory write / read direction基础模式访问面板。
//下面是引脚定义，需结合面板的引脚图定义理解。
// Breakout/Arduino UNO pin usage:
// LCD Data Bit :   7   6   5   4   3   2   1   0
// Uno dig. pin :   7   6   5   4   3   2   9   8
// Uno port/pin : PD7 PD6 PD5 PD4 PD3 PD2 PB1 PB0  //对应Port的标识。实际为PROTD或PORTB的宏定义所操作的端口寄存器
// Mega dig. pin:  29  28  27  26  25  24  23  22
#define LCD_RD   A0
#define LCD_WR   A1     
#define LCD_RS   A2        
#define LCD_CS   A3       
#define LCD_REST A4

//实现总线写入的最底层操作。在该函数调用前需设定好REST，RD,RS,CS提前置高。在SETUP主体中可以看到。
void Lcd_Writ_Bus(unsigned char d)
{
 PORTD = (PORTD & B00000011) | ((d) & B11111100);  //将d的2-7位写入PORTD的2-7位
 PORTB = (PORTB & B11111100) | ((d) & B00000011); //将d的0-1位写入PROTB的0-1位
 //*(portOutputRegister(digitalPinToPort(LCD_WR))) &=  ~digitalPinToBitMask(LCD_WR);
 //*(portOutputRegister(digitalPinToPort(LCD_WR)))|=  digitalPinToBitMask(LCD_WR);
 digitalWrite(LCD_WR,LOW);
 digitalWrite(LCD_WR,HIGH);
}


//在总线上写入命令。写入前要置RS引脚为低电平。
void Lcd_Write_Com(unsigned char VH)  
{   
  *(portOutputRegister(digitalPinToPort(LCD_RS))) &=  ~digitalPinToBitMask(LCD_RS);//LCD_RS=0;
  Lcd_Writ_Bus(VH);
}


//在总线上写入数据。写入前要置RS引脚为高电平。
void Lcd_Write_Data(unsigned char VH)
{
  *(portOutputRegister(digitalPinToPort(LCD_RS)))|=  digitalPinToBitMask(LCD_RS);//LCD_RS=1;
  Lcd_Writ_Bus(VH);
}

//写完一条指令后再写一8位的数据
void Lcd_Write_Com_Data(unsigned char com,unsigned char dat)
{
  Lcd_Write_Com(com);
  Lcd_Write_Data(dat);
}


//地址区域设置。涉及指令2Ah、2Bh
void Address_set(unsigned int x1,unsigned int y1,unsigned int x2,unsigned int y2)
{
        Lcd_Write_Com(0x2a);
	Lcd_Write_Data(x1>>8);   //设定屏幕数据操作区域的列首地址数据，，先写入16bit数据位的高位
	Lcd_Write_Data(x1);      //写入16bit数据位的低位
	Lcd_Write_Data(x2>>8);   //设定屏幕数据操作区域的列尾地址数据，，先写入16bit数据位的高位
	Lcd_Write_Data(x2);      //写入16bit数据位的低位
        Lcd_Write_Com(0x2b);
	Lcd_Write_Data(y1>>8);     //设定屏幕数据操作区域的行首地址数据，，先写入16bit数据位的高位
	Lcd_Write_Data(y1);         //写入16bit数据位的低位
	Lcd_Write_Data(y2>>8);      //设定屏幕数据操作区域的行尾地址数据，，先写入16bit数据位的高位
	Lcd_Write_Data(y2);         //写入16bit数据位的低位
	      Lcd_Write_Com(0x2c); 	//开启RAM数据持续写入状态。				 
}


//面板初始化
void Lcd_Init(void)
{
  digitalWrite(LCD_REST,HIGH);  
  delay(5); 
  digitalWrite(LCD_REST,LOW);
  delay(15);
  digitalWrite(LCD_REST,HIGH);
  delay(15);                   //到此为硬重置

  digitalWrite(LCD_CS,HIGH);  //根据MCU 8080-I 8bit通信接口操作规范，设置引脚的对应状态。
  digitalWrite(LCD_WR,HIGH);
  digitalWrite(LCD_CS,LOW);  //片选有效

    Lcd_Write_Com(0xCB);    //  指令Power Control A
    Lcd_Write_Data(0x39); 
    Lcd_Write_Data(0x2C); 
    Lcd_Write_Data(0x00); 
    Lcd_Write_Data(0x34);   //设置 Vcore=1.6V
    Lcd_Write_Data(0x02);   //设置DDVDH=5.6V

    Lcd_Write_Com(0xCF);   // 指令Power Control B
    Lcd_Write_Data(0x00); 
    Lcd_Write_Data(0XC1); 
    Lcd_Write_Data(0X30); 

    Lcd_Write_Com(0xE8);   //指令 Driver timing Congrol A 
    Lcd_Write_Data(0x85); 
    Lcd_Write_Data(0x00); 
    Lcd_Write_Data(0x78); 

    Lcd_Write_Com(0xEA);  //指令 Driver timing Congrol B
    Lcd_Write_Data(0x00); 
    Lcd_Write_Data(0x00); 
 
    Lcd_Write_Com(0xED);  //指令 Power on sequence control 
    Lcd_Write_Data(0x64); 
    Lcd_Write_Data(0x03); 
    Lcd_Write_Data(0X12); 
    Lcd_Write_Data(0X81); 

    Lcd_Write_Com(0xF7);  //指令Pump ratio control
    Lcd_Write_Data(0x20); //DDVDH=2*VCL
  
    Lcd_Write_Com(0xC0);    //Power control 
    Lcd_Write_Data(0x23);   //VRH[5:0]  GVDD=4.6V
 
    Lcd_Write_Com(0xC1);    //Power control 
    Lcd_Write_Data(0x10);   //SAP[2:0];BT[3:0] 

    Lcd_Write_Com(0xC5);    //VCM control  1  
    Lcd_Write_Data(0x3e);   //Contrast  VCOMH=3.45V VCOML=-1.5V
    Lcd_Write_Data(0x28); 
 
    Lcd_Write_Com(0xC7);    //VCM control2 
    Lcd_Write_Data(0x86);   //--
 
    Lcd_Write_Com(0x36);    // Memory Access Control 
    //Lcd_Write_Data(0x48);  // MX=1 Column Address Order ; BGR=1 RGB(IC)-->BGR(LCD Panel)
     Lcd_Write_Data(0x08);  // MX=0,BGR =1

    Lcd_Write_Com(0x3A);    //指令Pixel Format Set 
    Lcd_Write_Data(0x55);   //RGB 接口和MCU接口模式的像素数据格式为16bit/pixel  

    Lcd_Write_Com(0xB1);    //Frame Rate Control (B1h)（In Normal Mode /Full colors ）
    Lcd_Write_Data(0x00);  
    Lcd_Write_Data(0x18);  //79HZ(frame rate)
 
    Lcd_Write_Com(0xB6);    // Display Function Control 
    Lcd_Write_Data(0x08);   // Interval Scan 
    Lcd_Write_Data(0x82);   //底背景为白屏， 5 frams Scan Cycle
    Lcd_Write_Data(0x27);   //320 line

    Lcd_Write_Com(0x11);    //Exit Sleep 
    delay(120);             //必须120ms的延迟
				
    Lcd_Write_Com(0x29);    //Display on 
    Lcd_Write_Com(0x2c);    //Memory Write Start(2C) 或 Memory Write Continue(3ch)
}


//画水平线。。设定需填色的行列起止地址范围后往里填色
void H_line(unsigned int x, unsigned int y, unsigned int l, unsigned int c)                   
{	

  //x,y 为水平线的起始坐标 ，，l为水平线长度单位为像素，c为颜色参数
  unsigned int i,j;

  
  Lcd_Write_Com(0x02c);       //write_memory_start
  digitalWrite(LCD_RS,HIGH);
  digitalWrite(LCD_CS,LOW);

  l=l+x;  //转换成终止列的X坐标  
  Address_set(x,y,l,y);   //框出要填色的区域

  j=l;                //确定要填入的像素个数
  for(i=1;i<=j;i++)
  {
    Lcd_Write_Data(c>>8);   //写入颜色数据的高8位
    Lcd_Write_Data(c);      //写入颜色数据的低8位
  }
  digitalWrite(LCD_CS,HIGH);   
}


//画垂直线。。设定需填色的行列起止地址范围后往里填色
void V_line(unsigned int x, unsigned int y, unsigned int l, unsigned int c)                   
{	
  unsigned int i,j;
  
  Lcd_Write_Com(0x02c); //write_memory_start
  digitalWrite(LCD_RS,HIGH);
  digitalWrite(LCD_CS,LOW);
  
  l=l+y;  //转换成终止行的Y坐标
  Address_set(x,y,x,l); //框出要填色的区域
  j=l;             //确定要填入的像素个数
  for(i=1;i<=j;i++)
  { 
    Lcd_Write_Data(c>>8);    //写入颜色数据的高8位
    Lcd_Write_Data(c);       //写入颜色数据的低8位
  }
  digitalWrite(LCD_CS,HIGH);   
}


//画空心矩形
void Rect(unsigned int x,unsigned int y,unsigned int w,unsigned int h,unsigned int c)
{
  H_line(x  , y  , w, c);
  H_line(x  , y+h, w, c);
  V_line(x  , y  , h, c);
  V_line(x+w, y  , h, c);
}


//画实心矩形
void Rectf(unsigned int x,unsigned int y,unsigned int w,unsigned int h,unsigned int c)
{
  unsigned int i;
  for(i=0;i<h;i++)
  {
    H_line(x  , y+i, w, c);
  }
}


int RGB(int r,int g,int b)
{return r << 16 | g << 8 | b;
}




//满屏填充
void LCD_Clear(unsigned int j)                   
{	
  unsigned int i,m;

    digitalWrite(LCD_CS,LOW);
 Address_set(0,0,239,319);


  for(i=0;i<320;i++)  //320个行
    for(m=0;m<240;m++) //240列
    {
      Lcd_Write_Data(j>>8);
      Lcd_Write_Data(j);

    }
  digitalWrite(LCD_CS,HIGH);   
}

void setup()
{
  Serial.begin(9600);

  //以下为定义各个PIN脚的输入输出状态
  for(int p=0;p<10;p++)
  {
    pinMode(p,OUTPUT);
  }
  pinMode(A0,OUTPUT);
  pinMode(A1,OUTPUT);
  pinMode(A2,OUTPUT);
  pinMode(A3,OUTPUT);
  pinMode(A4,OUTPUT);
  digitalWrite(A0, HIGH);
  digitalWrite(A1, HIGH);
  digitalWrite(A2, HIGH);
  digitalWrite(A3, HIGH);
  digitalWrite(A4, HIGH);
  
  Lcd_Init();//初始化面板

}

void loop()
{  
   
   LCD_Clear(0xf800); //红色填满屏
   LCD_Clear(0x07E0); //绿色填满屏
   LCD_Clear(0x001F); //兰色填满屏
   
   //空心矩形 
  for(int i=0;i<500;i++)
  {
    Rect(random(300),random(300),random(300),random(300),random(65535)); // rectangle at x, y, with, hight, color
  Serial.println("完成第"+String(i)+"个矩形");
  }

//实心矩形
   for(int i=0;i<10;i++)
  {
    Rectf(random(300),random(300),random(300),random(300),random(65535)); // rectangle at x, y, with, hight, color
  Serial.println("完成第"+String(i)+"个实心矩形");
  }

  

}
```