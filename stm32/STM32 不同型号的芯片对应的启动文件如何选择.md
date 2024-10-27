> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_44788542/article/details/111645556)

*   startup_stm32f10x_ld_vl.s: for STM32 Low density Value line devices
    
*   startup_stm32f10x_ld.s: for STM32 Low density devices
    
*   startup_stm32f10x_md_vl.s: for STM32 Medium density Value line devices
    
*   startup_stm32f10x_md.s: for STM32 Medium density devices
    
*   startup_stm32f10x_hd.s: for STM32 High density devices
    
*   startup_stm32f10x_xl.s: for STM32 XL density devices
    
*   startup_stm32f10x_cl.s: for STM32 Connectivity line devices
    

cl：互联型产品，stm32f105/107 系列

vl：超值型产品，stm32f100 系列

xl：超高密度产品，stm32f101/103 系列

ld：低密度产品，FLASH 小于 64K

md：中等密度产品，FLASH=64 or 128

hd：高密度产品，FLASH 大于 128

```
/*  Tip: To avoid modifying this file each time you need to switch between these
        devices, you can define the device in your toolchain compiler preprocessor.

 - Low-density devices are STM32F101xx, STM32F102xx and STM32F103xx microcontrollers
   where the Flash memory density ranges between 16 and 32 Kbytes.
 - Low-density value line devices are STM32F100xx microcontrollers where the Flash
   memory density ranges between 16 and 32 Kbytes.
 - Medium-density devices are STM32F101xx, STM32F102xx and STM32F103xx microcontrollers
   where the Flash memory density ranges between 64 and 128 Kbytes.
 - Medium-density value line devices are STM32F100xx microcontrollers where the 
   Flash memory density ranges between 64 and 128 Kbytes.   
 - High-density devices are STM32F101xx and STM32F103xx microcontrollers where
   the Flash memory density ranges between 256 and 512 Kbytes.
 - High-density value line devices are STM32F100xx microcontrollers where the 
   Flash memory density ranges between 256 and 512 Kbytes.   
 - XL-density devices are STM32F101xx and STM32F103xx microcontrollers where
   the Flash memory density ranges between 512 and 1024 Kbytes.
 - Connectivity line devices are STM32F105xx and STM32F107xx microcontrollers.
  */
```

以上摘自 <stm32f10x.h>  
提示：为了避免每次需要在这些设备之间切换时都修改此文件，可以在工具链编译器预处理器中定义设备。

- 低密度器件是 STM32F101xx、STM32F102xx 和 STM32F103xx 微控制器  
其中 flash 范围在 16-32Kbytes。对应的启动文件为：startup_stm32f10x_ld.s

- 低密度超值型器件是 STM32F100xx 微控制器  
其中 flash 范围在 16-32Kbytes 之间。对应的启动文件为：startup_stm32f10x_ld_vl.s

- 中等密度设备为 STM32F101xx、STM32F102xx 和 STM32F103xx 微控制器  
其中 flash 范围在 64-128Kbytes 之间。对应的启动文件为：startup_stm32f10x_md.s

- 中等密度值线设备是 STM32F100xx 微控制器  
其中 flash 范围在 64-128Kbytes 之间。对应的启动文件为：startup_stm32f10x_md_vl.s

- 高密度设备是 STM32F101xx 和 STM32F103xx 微控制器  
其中 flash 范围在 256-512Kbytes 之间。对应的启动文件为：startup_stm32f10x_hd.s

- 高密度值线设备是 STM32F100xx 微控制器  
其中 flash 范围在 256-512 Kbytes 之间。对应的启动文件为：startup_stm32f10x_ld_vl.s

- 超高密度型器件是 STM32F101xx 和 STM32F103xx 微控制器  
其中 flash 范围在 512-1024 Kbytes 之间。对应的启动文件为：startup_stm32f10x_xl.s

- 互联网型设备是 STM32F105xx 和 STM32F107xx 微控制器。对应的启动文件为：startup_stm32f10x_cl.s