> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Z_H_Z_0/article/details/106574292)

1、[ARM](https://so.csdn.net/so/search?q=ARM&spm=1001.2101.3001.7020) 寄存器组介绍

ARM 处理器一般共有 37 个[寄存器](https://so.csdn.net/so/search?q=%E5%AF%84%E5%AD%98%E5%99%A8&spm=1001.2101.3001.7020)，其中包括：

（1） 31 个通用寄存器，包括 PC（[程序计数器](https://so.csdn.net/so/search?q=%E7%A8%8B%E5%BA%8F%E8%AE%A1%E6%95%B0%E5%99%A8&spm=1001.2101.3001.7020)）在内，都是 32 位的寄存器。  
（2） 6 个状态寄存器，都是 32 位的寄存器。  
ARM 处理器共有 7 种不同的处理器模式：  
用户模式（User），快速[中断](https://so.csdn.net/so/search?q=%E4%B8%AD%E6%96%AD&spm=1001.2101.3001.7020)模式（FIQ），普通中断模式（IRQ），管理模式（Svc），数据访问中止模式（Abort），未定义指令中止模式（Und），系统模式（Sys），  
在每一种处理器模式中有一组相应的寄存器。在任意一种处理器模式下，可见的寄存器包括 15 个通用寄存器（R0~R14）、一个或者二个状态寄存器以及程序计数器（PC）。在所有的寄存器中，有些是各模式共用同一个物理寄存器，有些寄存器是各个模式自己拥有独立的物理寄存器![[../_resources/RM 寄存器及功能介绍  R0-R15 寄存器_arm 中的 scratch 寄存器是什么意思/2023b07aa3b8fd15f5508495249dd3f4_MD5.png]]

其中 r0~r3 主要用于子程序间传递参数， r4~r11 主要用于保存局部变量，但在 Thumb 程序中，通常只能使用 r4~r7 来保存局部变量； r12 用作子程序间 scratch 寄存器，即 ip 寄存器； r13 通常用做栈指针，即 sp； r14 寄存器又被称为连接寄存器（lr），用于保存子程序以及中断的返回地址； r15 用作程序计数器（pc），由于 ARM 采用了流水线机制，当正确读取了 PC 的值后，该值为当前指令地址加 8 个字节，即 PC 指向当前指令的下两条指令地址。

CPSR 和 SPSR 都是程序状态寄存器，其中 SPSR 是用来保存中断前的 CPSR 中的值，以便在中断返回之后恢复处理器程序状态。  
2.CPSR 寄存器详解

所有处理器模式下都可访问当前程序状态寄存器 CPSR。CPSR 中包含条件码标志、中断禁止位、当前处理器模式以及其他状态和控制信息。在每种异常模式下都有一个对用的程序状态寄存器 SPSR。当异常出现时，SPSR 用于保存 CPSR 的状态，以便异常返回后恢复异常发生时的工作状态。

(1) 条件码标志

N、Z、C、V，最高 4 位称为条件码标志。ARM 的大多数指令可以条件执行的，即通过检测这些条件码标志来决定程序指令如何执行。

各个条件码的含义如下：

N：在结果是有符号的二进制补码情况下，如果结果为负数，则 N=1；如果结果为非负数，则 N=0。

Z：如果结果为 0，则 Z=1; 如果结果为非零，则 Z=0。

C：其设置分一下几种情况：

```
对于加法指令（包含比较指令CMN），如果产生进位，则C=1;否则C=0。

           对于减法指令（包括比较指令CMP），如果产生借位，则C=0;否则C=1。

           对于有移位操作的非法指令，C为移位操作中最后移出位的值。

           对于其他指令，C通常不变。
```

V：对于加减法指令，在操作数和结果是有符号的整数时，如果发生溢出，则 V=1; 如果无溢出发生，则 V=0; 对于其他指令，V 通常不发生变化。

(2) 控制位的作用在图 1 中可以看出，在这里就不阐述了。

二：CPSR 与 CPSR_c 的区别

```
CPSR_c指的是CPSR的低8位控制位

  CPSR有4个8位区域：标志域（F）、状态域（S）、扩展域（X）、控制域（C）

  MSR - Load specified fields of the CPSR or SPSR with an immediate constant, or from the contents of a general-purpose register.

 Syntax:

 MSR{cond} <psr>_<fields>, #immed_8r MSR{cond} <psr>_<fields>, Rm where: cond is an optional condition code. <psr> is either CPSR or SPSR. <fields> specifies the field or fields to be moved. <fields> can be one or more of:
```

c control field mask byte (PSR[7:0]) x extension field mask byte (PSR[15:8]) s status field mask byte (PSR[23:16) f flags field mask byte (PSR[31:24]). immed_8r is an expression evaluating to a numeric constant. The constant must correspond to an 8-bit pattern rotated by an even number of bits within a 32-bit word. Rm is the source register.

```
C 控制域屏蔽字节(psr[7:0])
  X 扩展域屏蔽字节(psr[15:8])
  S 状态域屏蔽字节(psr[23:16])
  F 标志域屏蔽字节(psr[31:24])
```

常用于 MRS 或 MSR 指令, 用于 psr 中的值转移到寄存器或把寄存器的内容加载到 psr 中.  
如:

MSR CPSR_c，＃0xd3

**三、R0-R15 寄存器 根据 “ARM-thumb 过程调用标准”：**  
**R0-R3** _**用作传入函数参数，传出函数返回值**_。在子程序调用之间，可以将 r0-r3 用于任何用途。  
被调用函数在返回之前不必恢复 r0-r3。如果调用函数需要再次使用 r0-r3 的内容，则它必须保留这些内容。  
**R4-R11** **被用来存放函数的局部变量**。如果被调用函数使用了这些寄存器，它在返回之前必须恢复这些寄存器的值。  
**R12** **是内部调用暂时寄存器 ip**。它在过程链接胶合代码（例如，交互操作胶合代码）中用于此角色。  
在过程调用之间，可以将它用于任何用途。被调用函数在返回之前不必恢复 r12。  
**R13** **是栈指针 sp**。它不能用于任何其它用途。sp 中存放的值在退出被调用函数时必须与进入时的值相同。  
**R14** **是链接寄存器 lr**。如果您保存了返回地址，则可以在调用之间将 r14 用于其它用途，程序返回时要恢复  
**R15** **是程序计数器 PC**。它不能用于任何其它用途。  
注意：在中断程序中，所有的寄存器都必须保护，编译器会自动保护 R4～R11