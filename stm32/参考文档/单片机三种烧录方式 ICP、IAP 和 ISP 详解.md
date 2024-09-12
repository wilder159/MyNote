> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/69237591)

单片机的烧录方式主要可以分为三种，分别为 ICP(在[电路编程](https://zhida.zhihu.com/search?q=%E7%94%B5%E8%B7%AF%E7%BC%96%E7%A8%8B&zhida_source=entity&is_preview=1))、IAP(在应用编程) 以及 ISP([在系统编程](https://zhida.zhihu.com/search?q=%E5%9C%A8%E7%B3%BB%E7%BB%9F%E7%BC%96%E7%A8%8B&zhida_source=entity&is_preview=1))。玩单片机的同学都应该听所说 IAP、ICP 和 ISP 这几个词，在此小编在帮你们 “巩固” 一下知识。首先先来介绍这几个小伙伴的名字。  

1.  ICP(In Circuit Programing) 在电路编程  
    
2.  ISP(In System Programing) 在系统编程  
    
3.  IAP(In applicating Programing) 在应用编程

**1、ICP（In Circuit Programing）**  

-----------------------------------

使用过新唐单片机的一定知道，[新唐单片机](https://zhida.zhihu.com/search?q=%E6%96%B0%E5%94%90%E5%8D%95%E7%89%87%E6%9C%BA&zhida_source=entity&is_preview=1)配套了一套编程工具，那就是 NuMicro_ICP_Programming_Tool。就像 ST 芯片配置的有 J-Flash 工具一样。

使用对应厂家的软件以及[仿真器](https://zhida.zhihu.com/search?q=%E4%BB%BF%E7%9C%9F%E5%99%A8&zhida_source=entity&is_preview=1)都可以烧录程序，目前主流的有 Jtag（Joint Test Action Group）以及 SWD（Serial Wire Debug）接口。而 **ICP 编程就是以 SWD 接口**进行的。

下图为 Jtag 接口和 SWD 接口的区别：

![[../../_resources/未命名/60044d1cfca3fa782a8008add8434420_MD5.jpg]]

执行 ICP 功能，仅需要 3 个引脚 RESET、ICPDA 及 ICPCK。RESET 用于进入或退出 ICP 模式，ICPDA 为数据输入输出脚，ICPCK 为[编程时钟](https://zhida.zhihu.com/search?q=%E7%BC%96%E7%A8%8B%E6%97%B6%E9%92%9F&zhida_source=entity&is_preview=1)输入脚。用户需要在系统板上预留 VDD、GND 以及这三个脚。

![[../../_resources/未命名/42022513510ba1ffd7f00279b45cd08e_MD5.jpg]]

新唐官方给了详细的描述，ICP 是指 “在电路编程”，PC 上运行的软件“NuMicro ICP 编程工具” 透过 SWD 的端口更新晶片内部 APROM、LDROM、数据闪存 (DataFlash) 和目标[用户配置](https://zhida.zhihu.com/search?q=%E7%94%A8%E6%88%B7%E9%85%8D%E7%BD%AE&zhida_source=entity&is_preview=1)字 (Config) 芯片。

![[../../_resources/未命名/dd8543cf88269d8f9df2d8ee5f3f0e25_MD5.jpg]]

**2、ISP（In System Programing）**  

----------------------------------

ISP 是指 “在系统上编程”，目标芯片使用 USB/UART/SPI/I²C/RS-485/CAN **周边接口的 LDROM 引导代码**去更新晶片内部 APROM、数据闪存 (DataFlash) 和用户配置字(Config)。

![[../../_resources/未命名/535ab3c0d7b762e9873b75b100e02902_MD5.jpg]]

**3、IAP（In applicating Programing）**  

---------------------------------------

**IAP 就是通过软件实现[在线电擦除](https://zhida.zhihu.com/search?q=%E5%9C%A8%E7%BA%BF%E7%94%B5%E6%93%A6%E9%99%A4&zhida_source=entity&is_preview=1)和编程的方法**。IAP 技术是从结构上将 Flash 存储器映射为两个存储体，当运行一个存储体上的用户程序时，可对另一个存储体重新编程，之后将程序从一个存储体转向另一个。

### 最后我们以[烧录](https://zhida.zhihu.com/search?q=%E7%83%A7%E5%BD%95&zhida_source=entity&is_preview=1)过程中使用的烧录工具以及具体案例来详细介绍这三种烧录方式。  

1、**ICP 使用 SWD 接口进行烧录程序**。常用的烧录工具为 J-Link、ST-Link、Nu-Link。与之配套的烧录软件为 J-Flash、NuMicro_ICP_Programming_Tool、[st-link utility](https://zhida.zhihu.com/search?q=st-link+utility&zhida_source=entity&is_preview=1)。

![[../../_resources/未命名/f88574ff17f28945a21f86417575ea34_MD5.jpg]]

2、ISP 是使用**引导程序通过 USB/UART 等接口进行烧录**的，首先就是需要有 BoodLoad 程序。最常见的烧录方式就是学习 [8051 单片机](https://zhida.zhihu.com/search?q=8051%E5%8D%95%E7%89%87%E6%9C%BA&zhida_source=entity&is_preview=1)时使用的 STC-ISP 烧录工具了。

![[../../_resources/未命名/9ef90db25cf795b9e0032c31d873cdff_MD5.jpg]]

3、**IAP 就是通过软件实现在线电擦除和编程的方法，没有使用任何工具**，仅仅是通过软件的方法来更新 Flash 中的数据。

讲述一个案例，那就是通过 4G 模块来远程更新程序。将 Flash 分成两块区域，第一块为 Boodload 程序，第二块区域存放的是应用程序 APP。4G 模块和目标板通讯，通讯中包含是否更新的位，如果主板接收到需要更新的位，就往 Flash 中写入一个标志位，比如'P'，之后程序跳到第一段程序 Boodload 程序中执行，首先判断 Flash 中的是否有更新程序的标志位'P'，如果有则通过规定的协议进行更新应用程序中的程序，更新完毕后清除 Flash 中的更新标志位，跳转到应用程序中去执行。如果没有更新程序标志位‘P’，跳到应用程序执行。

**总结：**

1.  **ICP：使用 SWD 接口进行烧录，如 J-Link 烧录器和 J-Flash 软件配合使用。**
2.  **ISP：使用引导程序（Bootload）加上外围 UART/USB 等接口进行烧录。**
3.  **IAP：软件自身实现在线电擦除和编程的方法，不使用任何工具。程序通常分成两块，分别为[引导程序](https://zhida.zhihu.com/search?q=%E5%BC%95%E5%AF%BC%E7%A8%8B%E5%BA%8F&zhida_source=entity&is_preview=1)和应用程序。**

**​​​​​​​**最后我问大家一个问题，在使用 IAP 编程时候，可否将引导程序和应用程序合成只有一个[代码区](https://zhida.zhihu.com/search?q=%E4%BB%A3%E7%A0%81%E5%8C%BA&zhida_source=entity&is_preview=1)的应用程序，这样还能实现软件更新吗？具体又怎么操作？大家可以思考一下，答案是可行的。

![[../../_resources/未命名/f741b78213148815db0bfced27c96c4c_MD5.jpg]]