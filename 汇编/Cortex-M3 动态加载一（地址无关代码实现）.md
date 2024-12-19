> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/ppym/p/3655425.html) 

这篇文章是自己疑惑究竟地址无关性是如何实现，然后查看汇编和 CPU 指令手册，最后分析解除自己疑惑的，高手不要鄙视，哈哈。
编译 C 代码时候需要制定 --acps/ropi 选项，如下例子：
```c
1 void SystemInit(void)
 2 {
 3 }
 4 void fun_for_sub(void)
 5 {
 6     int j;
 7     for(j=65535;j >=0; j--)
 8       ;
 9 }
10 int main(void)
11 {
12          fun_for_sub();
13          while(1);
14 }
```
编译：
```c
armcc  -c --cpu Cortex-M3 -O0 --apcs=interwork --apcs /ropi/rwpi -o main.o main.c
```
使用 fromelf 查看汇编代码
```c
fromelf.exe -s -c main.o
```
text 段生成的汇编代码如下：
```c
1 ** Section #1 '.text' (SHT_PROGBITS) [SHF_ALLOC + SHF_EXECINSTR]
 2     Size   : 14 bytes (alignment 2)
 3     Address: 0x00000000
 5     $t
 6     .text
 7     SystemInit
 8         0x00000000:    4770        pG      BX       lr
 9     fun_for_sub
10         0x00000002:    4770        pG      BX       lr
11     main
12         0x00000004:    b500        ..      PUSH     {lr}
13         0x00000006:    f7fffffe       ....    BL       fun_for_sub ; 0x2 Section #1
14         0x0000000a:    205a        Z       MOVS     r0,#0x5a
15         0x0000000c:    bd00        ..      POP      {pc}
```
查看关键的一句调用函数 fun_for_sub 的汇编代码：
```c
0x00000006:    f7fffffe       ....    BL       fun_for_sub ; 0x2 Section #1
```
查找 arm 的官方 DDI0403D_arm_architecture_v7m_reference_manual_errata_markup_1_0.pdf 关于 BL 指令的解释如下：
```c
Branch with Link (immediate) calls a subroutine at a PC-relative address.
```
得知 BL 是一条 PC 相关的指令。
具体看 BL 指令的构成：
[[../_resources/Cortex-M3 动态加载一（地址无关代码实现）/6af1301ff9b6437398067a2d3c3cdbee_MD5.jpeg|]]
![[../_resources/Cortex-M3 动态加载一（地址无关代码实现）/6af1301ff9b6437398067a2d3c3cdbee_MD5.jpeg]]
[[../_resources/Cortex-M3 动态加载一（地址无关代码实现）/c2f1ee5886195f41633a96875e80cb63_MD5.jpeg|Open: Pasted image 20241206133858.png]]
![[../_resources/Cortex-M3 动态加载一（地址无关代码实现）/c2f1ee5886195f41633a96875e80cb63_MD5.jpeg]]
根据我们产生的指令 f7fffffe，
对应如下：
```
f7ff  :  15  14  13  12  11  10  9  8  7  6  5  4  3  2  1

        1   1   1   1   0   1  1  1  1  1  1  1  1  1  1
```

```
fffe  :  15  14  13  12  11  10  9  8  7  6  5  4  3  2  1

        1   1   1   1   1   1  1  1  1  1  1  1  1  1  0
```

```
符号位S=1，J1=1，J2=1，imm10 = 11 1111 1111，imm11 = 111 1111 1110
```

所以 I1 = ！(J1~S) = 1,  I2 = !(J2~S) = 1,
imm32 = SignExtend(S:I1:I2:imm10:imm11:’0’,32) = SignExtend(1:1:1:11 1111 1111:111 1111 1110:’0’,32) = 1111 1111 1111 1111 1111 1111 1111 1100 = 0xfffffffc。
0xfffffffc 是 - 4 的补码，另外当前 PC 是 0x00000006，
再根据上面的 Operation 最后一步 BranchWritePC(PC + imm32)
最终跳转到 0x6 + (-4) = 0x2 的地址出，即函数 fun_for_sub 的地址，因此实现根据当前 PC 实现了地址无关性的代码。
在 X86 平台下面也是差不多的原理，使用的也是基于 PC 相关的跳转指令。《程序员的自我修养—链接、装载和库》讲得很好。