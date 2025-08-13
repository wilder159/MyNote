> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Luckiers/article/details/128221506)

#### 0.1.1.1 目录

*   *   [一、简介](#_1)
    *   [二、ARM 汇编指令说明](#ARM__4)
    *   *   [寄存器分类介绍](#_5)
        *   *   [1. 通用寄存器](#1__6)
            *   [2. 专用寄存器](#2__70)
            *   [3. 控制寄存器](#3__87)
        *   [2.1 32 位数据操作指令](#21_32_89)
        *   [2.2 32 位存储器数据传送指令](#22_32_140)
        *   [2.3 32 位转移指令](#23_32_157)
        *   [2.4 其它 32 位指令](#24_32_165)
        *   [2.5 立即数](#25__188)
        *   [2.6 逻辑数](#26__191)
        *   [2.7 逻辑运算和算术运算](#27__194)
    *   [三、实例讲解](#_197)
    *   *   [3.1 MRS](#31_MRS_198)
        *   [3.2 MSR](#32_MSR_207)
        *   [3.3 PRIMASK](#33_PRIMASK_216)
        *   [3.4 FAULTMASK](#34_FAULTMASK_222)
        *   [3.5 BX 指令](#35_BX_233)
        *   [3.6 零寄存器 wzr、xzr](#36__wzrxzr_241)
        *   [3.7 立即寻址指令 MOV](#37_MOV_244)
        *   [3.8 寄存器间接寻址指令 LDR](#38_LDR_252)
        *   [3.9 寄存器移位寻址指令 LSL](#39_LSL_268)
        *   [3.10 基址寻址指令 STR](#310__STR_299)
        *   [3.11 多寄存器寻址指令](#311__306)
        *   [3.12 无条件转移 B，BAL](#312_BBAL_312)
        *   [3.13 条件转移 B.cont](#313_Bcont_323)
        *   [3.14 WFE 和 WFI 对比](#314_WFE__WFI__350)
        *   [3.15 MRC：协处理器寄存器到 ARM 寄存器的数据传输](#315_MRCARM_360)
        *   [3.16 MCR：寄存器到协处理器寄存器的数据传输](#316_MCR_369)
        *   [3.17 STM：将指令中寄存器列表中的各寄存器数值写入到连续的内存单元中](#317_STM_376)
        *   [3.18 LDM：将数据从连续内存单元中读取到指令的寄存器列表中的各寄存器中](#318_LDM_414)
        *   [3.19 LDR：从内存中将一个 32 位的字读取到目标寄存器](#319_LDR32_421)
        *   [3.20 STR：将 32 位字数据写入到指定的内存单元](#320_STR32_426)
        *   [3.21 SWI：软中断指令](#321_SWI_441)
        *   [3.22 BIC 清除位](#322_BIC_450)
        *   [3.23 EOR 逻辑异或指令](#323_EOR_463)
        *   [3.24 CMN 与负数对比](#324_CMN_470)
        *   [3.25 MVN 取反](#325_MVN_474)
        *   [3.26 LSL（Logical Shift Left）左移运算](#326_LSLLogical_Shift_Left_483)
        *   [3.27 STP](#327_STP_486)
    *   [实例解析](#_503)

### 0.1.2 一、简介

本文主要整理了 arm 常用的汇编指令，同时通过实例进一步讲述语句的用法。

### 0.1.3 二、ARM 汇编指令说明

#### 0.1.3.1 [寄存器](https://so.csdn.net/so/search?q=%E5%AF%84%E5%AD%98%E5%99%A8&spm=1001.2101.3001.7020)分类介绍

##### 0.1.3.1.1 通用寄存器

通用寄存器是一组用于存储数据和地址的寄存器。在 [ARM 架构](https://so.csdn.net/so/search?q=ARM%20%E6%9E%B6%E6%9E%84&spm=1001.2101.3001.7020)的不同版本中，这些寄存器的数量和命名有所不同。  
R0-R15 (R0-R14 + PC):  
在 ARMv7 和之前的版本中，有 16 个通用寄存器，编号从 R0 到 R15。  
R0 到 R14 用于存储数据和地址。  
R15 通常被称为程序计数器（PC），用于存储下一条指令的地址。

X0-X30 (X0-X28+ FR + LR):  
在 ARMv8 和之后的版本中，有 31 个通用寄存器，编号从 X0 到 X30。  
X0 到 X28 用于存储数据和地址。  
X29： Frame Pointer (FP) 寄存器, 它的主要作用是指向当前函数的栈帧 (Stack Frame)  
X30: 链接寄存器（LR），用于保存返回地址。

在 ARMv7 架构中使用程序状态寄存器 (Current Program Status Register,CPSR) 来表示当前的处理器状态(processor stste), 而在 ARMv8 里使用 PSTATE 寄存器来表示。

**ARMv8 及以后版本：**

<table><thead><tr><th align="left">寄存器</th><th>位数</th><th align="left">描述</th></tr></thead><tbody><tr><td align="left">X0-X30</td><td>64bit</td><td align="left">通用寄存器，如果有需要可以当作 32bit 使用：W0-W30</td></tr><tr><td align="left">FP(X29)</td><td>64bit</td><td align="left">保存栈帧地址 (栈底指针)</td></tr><tr><td align="left">LR(X30)</td><td>64bit</td><td align="left">程序链接寄存器，保存子程序结束后需要执行的下一条指令</td></tr><tr><td align="left">SP</td><td>64bit</td><td align="left">保存栈顶指针，使用 SP/WSP 来进行对 SP 寄存器的访问。</td></tr><tr><td align="left">PC</td><td>64bit</td><td align="left">程序计数器，俗称 PC 指针，总是指向即将要执行的下一条指令，在 arm64 中，软件不能修改 PC 寄存器</td></tr><tr><td align="left">PSTATE</td><td>64bit</td><td align="left">状态寄存器，用于保存处理器的当前状态信息。</td></tr></tbody></table>

ARM 64 包含 31 个 64bit 寄存器，记为 X0~X30。  
每一个通用寄存器，它的低 32bit 都可以被访问，记为 W0~W30。

LDR 和 STR 分别从地址中读入内容到寄存器和向地址中写入寄存器的内容，后缀 B 表示字节，H 表示半字，W 表示单字

向量和浮点寄存器。32 个寄存器，向量和浮点共用，每个 128 位，用 V0 到 V31 来表示。不同的记号可以表示不同的长度，B 表示字节，H 表示半字，S 表示单字，D 表示双字，Q 表示四字  
![[../_resources/ARM 全汇编指令（详细）整理附实例快速掌握_arm 指令集/c00b8f3c09582f980182c47c971dc7bc_MD5.webp]]

X0 - X7： 这 8 个寄存器通常用作函数参数寄存器，在函数调用时用来传递前 8 个参数，若参数个数大于 8，就采用栈来传递。64 位的函数返回值通常存放在 X0 寄存器中，128 位的返回结果存在 X0 和 X1 两个寄存器中。  
X8： 间接结果位置寄存器，用于保存子函数的返回地址。在一些情况下，X8 可以用于普通的临时寄存器，用于存储中间计算结果。  
X9 - X15： 通常被用作临时变量和中间计算结果的存储。调用者有责任在函数调用前保存它们的值, 以免被覆盖。函数返回后, 调用者也需要恢复这些寄存器的值。  
X16、X17： X16 (IP0, Intra-Procedure-call scratch register 0) 这个寄存器通常被用作临时寄存器, 用于存储函数内部的中间计算结果。它在函数调用过程中可能会被修改, 所以调用者需要自行保存和恢复。X17 与 X16 类似。

X18： X18 (Platform register)这个寄存器通常被用作平台相关的寄存器, 其用途取决于具体的硬件平台和软件环境。在某些系统中, X18 可能被用作过程链接表 (PLT) 指针, 用于动态链接。在其他系统中, X18 可能被用作线程局部存储 (Thread Local Storage) 的指针。  
X19 - X28： 通常用于存储函数的局部变量和中间计算结果。被调用的函数有责任在返回前保存和恢复这些寄存器的值, 确保调用者可以继续使用。

X29： Frame Pointer (FP) 寄存器, 它的主要作用是指向当前函数的栈帧 (Stack Frame), 方便访问函数内部的局部变量和参数。当一个函数被调用时, x29 寄存器会被设置为指向该函数的栈帧起始地址。这样可以通过 x29 寄存器轻松访问函数内部的局部变量和参数, 而不需要依赖 x30 寄存器(Link Register) 中存储的返回地址。

X30: Link Register (LR), 它的主要作用是在函数调用时存储函数的返回地址, 以便函数执行完毕后能够正确地返回到调用点。当一个函数被调用时, CPU 会将当前执行点的地址保存到 x30 寄存器中。这样在函数执行完毕后, 只需要从 x30 寄存器中恢复返回地址, 就可以正确地返回到调用点。注意：当一个函数被调用时, CPU 会自动将当前执行点的地址 (也就是函数调用语句的下一条指令地址) 保存到 x30 寄存器中。

X9 - X15 和 X19 - X28 这两组寄存器的区别:

x9 到 x15 则主要用作临时变量和中间计算结果的存储。x19 到 x28 通常用作函数的局部变量和中间计算结果的存储。对于 x9 到 x15 这些 "caller-saved" 寄存器, 调用者 (caller) 有责任在函数调用前保存它们的值, 并在调用后恢复。对于 x19 到 x28 这些 "callee-saved" 寄存器, 被调用的函数 (callee) 有责任在返回前保存和恢复它们的值。使用 "callee-saved" 寄存器通常可以减少对栈的访问, 提高性能。但同时也增加了函数调用时保存和恢复寄存器的开销。

**ARMv8 前版本：**  
R0-R15 寄存器 根据 “ARM-thumb 过程调用标准”：  
R0-R3 其中，R0 通常用于存储函数的返回值，R1-R3 则常用于传递函数参数。在子程序调用之前，可以将 R0-R3 用于任何用途。被调用函数在返回之前不必恢复 R0-R3。如果调用函数需要再次使用 r0-r3 的内容，则它必须保留这些内容。  
R4-R11 被用来存放函数的局部变量。如果被调用函数使用了这些寄存器，它在返回之前必须恢复这些寄存器的值。  
R12 是内部调用暂时寄存器 IP。它在过程链接胶合代码（例如，交互操作胶合代码）中用于此角色。  
在过程调用之间，可以将它用于任何用途。被调用函数在返回之前不必恢复 R12。  
R13 是栈指针 SP：  
在 ARM 指令集中，R13 常被用作堆栈指针，用于存储程序中的局部变量和函数调用时的返回地址。它不能用于任何其它用途。SP 中存放的值在退出被调用函数时必须与进入时的值相同。用户也可以使用其他寄存器作为堆栈指针，但在 Thumb 指令集中，某些指令强制要求使用 R13 作为堆栈指针。  
R14 是链接寄存器 LR：  
用于存储函数调用之前的返回地址，如果您保存了返回地址，则可以在调用之前将 R14 用于其它用途，程序返回时要恢复。当执行子程序调用指令（如 BL 或 BLX）时，R14 会被设置成该子程序的返回地址。在子程序返回时，将 R14 的值复制回程序计数器 PC 即可完成子程序的调用返回。  
R15 是程序计数器 PC：  
用于存储当前正在执行的指令的地址。程序计数器是处理器控制指令执行的关键寄存器之一。它不能用于任何其它用途。由于 ARM 采用了流水线机制，当正确读取了 PC 的值后，该值为当前指令地址加 8 个字节，即 PC 指向当前指令的下两条指令地址。  
注意：在中断程序中，所有的寄存器都必须保护，编译器会自动保护 R4～R11。

##### 0.1.3.1.2 专用寄存器

ARM 处理器中的专用寄存器主要包括程序状态寄存器（CPSR）和备份的程序状态寄存器（SPSRs）。  
程序状态寄存器（CPSR）

CPSR 是一个 32 位的特殊寄存器，用于存储当前程序的状态信息。它包含以下内容：

ALU 状态标志 ：如条件码（如零标志 Z、负标志 N、进位标志 C 等），用于反映 ALU 的运算结果。  
中断使能位 ：用于控制中断的使能状态。  
执行模式位 ：用于标识当前处理器的执行模式（如用户模式、系统模式、中断模式等）。

CPSR 和 SPSR 都是程序状态寄存器，其中 SPSR 是用来保存中断前的 CPSR 中的值，以便在中断返回之后恢复处理器程序状态。  
CPSR 在任何处理器模式下都可被访问和修改（但某些位可能需要特权级代码才能修改）。通过读取和修改 CPSR 寄存器的各个标志位和控制位，可以控制程序的执行流程和处理器的行为。  
备份的程序状态寄存器（SPSRs）

ARM 处理器还包含 5 个备份的程序状态寄存器（SPSR_fiq、SPSR_irq、SPSR_svc、SPSR_abt、SPSR_und），用于在异常处理期间保存 CPSR 的值。当处理器进入异常模式时，会将 CPSR 的内容复制到对应的 SPSR 中；当从异常模式返回时，则可以将 SPSR 的内容复制回 CPSR 以恢复处理器的状态。

##### 0.1.3.1.3 控制寄存器

虽然控制寄存器不直接归类为通用或专用寄存器，但它们在 ARM 处理器的控制中发挥着重要作用。这些寄存器通常包含处理器的控制位和配置位，用于控制处理器的行为和工作模式。由于控制寄存器的访问和修改通常需要特权级代码，因此它们在普通的应用程序中很少被直接访问。如控制 CPU 的行为，如 CTRL 和 ACTLR。

#### 0.1.3.2 32 位数据操作指令

<table><thead><tr><th align="left">名字</th><th align="left">功能</th></tr></thead><tbody><tr><td align="left">ADC</td><td align="left">带进位加法</td></tr><tr><td align="left">ADD</td><td align="left">加法</td></tr><tr><td align="left">ADDW</td><td align="left">宽加法（可以加 12 位立即数）</td></tr><tr><td align="left">AND</td><td align="left">按位与</td></tr><tr><td align="left">ASR</td><td align="left">算术右移</td></tr><tr><td align="left">BIC</td><td align="left">位清零（把一个数按位取反后，与另一个数逻辑与）</td></tr><tr><td align="left">BFC</td><td align="left">位段清零</td></tr><tr><td align="left">BFI</td><td align="left">位段插入</td></tr><tr><td align="left">CMN</td><td align="left">负向比较（把一个数和另一个数的二进制补码比较，并更新标志位）</td></tr><tr><td align="left">CMP</td><td align="left">比较两个数并更新标志位</td></tr><tr><td align="left">CLZ</td><td align="left">计算前导零的数目</td></tr><tr><td align="left">EOR</td><td align="left">按位异或</td></tr><tr><td align="left">LSL</td><td align="left">逻辑左移</td></tr><tr><td align="left">LSR</td><td align="left">逻辑右移</td></tr><tr><td align="left">MLA</td><td align="left">乘加</td></tr><tr><td align="left">MLS</td><td align="left">乘减</td></tr><tr><td align="left">MOVW</td><td align="left">把 16 位立即数放到寄存器的底 16 位，高 16 位清 0</td></tr><tr><td align="left">MOV</td><td align="left">加载 16 位立即数到寄存器（其实汇编器会产生 MOVW——译注）</td></tr><tr><td align="left">MOVT</td><td align="left">把 16 位立即数放到寄存器的高 16 位，低 16 位不影响</td></tr><tr><td align="left">MVN</td><td align="left">移动一个数的补码</td></tr><tr><td align="left">MUL</td><td align="left">乘法</td></tr><tr><td align="left">ORR</td><td align="left">按位或</td></tr><tr><td align="left">ORN</td><td align="left">把源操作数按位取反后，再执行按位或（</td></tr><tr><td align="left">RBIT</td><td align="left">位反转（把一个 32 位整数先用 2 进制表达，再旋转 180 度——译注）</td></tr><tr><td align="left">REV</td><td align="left">对一个 32 位整数做按字节反转</td></tr><tr><td align="left">REVH/REV16</td><td align="left">对一个 32 位整数的高低半字都执行字节反转</td></tr><tr><td align="left">REVSH</td><td align="left">对一个 32 位整数的低半字执行字节反转，再带符号扩展成 32 位数</td></tr><tr><td align="left">ROR</td><td align="left">圆圈右移</td></tr><tr><td align="left">RRX</td><td align="left">带进位的逻辑右移一格（最高位用 C 填充，且不影响 C 的值——译注）</td></tr><tr><td align="left">SFBX</td><td align="left">从一个 32 位整数中提取任意的位段，并且带符号扩展成 32 位整数</td></tr><tr><td align="left">SDIV</td><td align="left">带符号除法</td></tr><tr><td align="left">SMLAL</td><td align="left">带符号长乘加（两个带符号的 32 位整数相乘得到 64 位的带符号积，再把积加到另一个带符号 64 位整数中）</td></tr><tr><td align="left">SMULL</td><td align="left">带符号长乘法（两个带符号的 32 位整数相乘得到 64 位的带符号积）</td></tr><tr><td align="left">SSAT</td><td align="left">带符号的饱和运算</td></tr><tr><td align="left">SBC</td><td align="left">带借位的减法</td></tr><tr><td align="left">SUB</td><td align="left">减法</td></tr><tr><td align="left">SUBW</td><td align="left">宽减法，可以减 12 位立即数</td></tr><tr><td align="left">SXTB</td><td align="left">字节带符号扩展到 32 位数</td></tr><tr><td align="left">TEQ</td><td align="left">测试是否相等（对两个数执行异或，更新标志但不存储结果）</td></tr><tr><td align="left">TST</td><td align="left">测试（对两个数执行按位与，更新标志但不存储结果）</td></tr><tr><td align="left">UBFX</td><td align="left">无符号位段提取</td></tr><tr><td align="left">UDIV</td><td align="left">无符号除法</td></tr><tr><td align="left">UMLAL</td><td align="left">无符号长乘加（两个无符号的 32 位整数相乘得到 64 位的无符号积，再把积加到另一个无符号 64 位整数中）</td></tr><tr><td align="left">UMULL</td><td align="left">无符号长乘法（两个无符号的 32 位整数相乘得到 64 位的无符号积）</td></tr><tr><td align="left">USAT</td><td align="left">无符号饱和操作（但是源操作数是带符号的——译注）</td></tr><tr><td align="left">UXTB</td><td align="left">字节被无符号扩展到 32 位（高 24 位清 0——译注）</td></tr><tr><td align="left">UXTH</td><td align="left">半字被无符号扩展到 32 位（高 16 位清 0——译注）</td></tr></tbody></table>

#### 0.1.3.3 32 位存储器数据传送指令

<table><thead><tr><th align="left">名字</th><th align="left">功能</th></tr></thead><tbody><tr><td align="left">LDR</td><td align="left">加载字到寄存器</td></tr><tr><td align="left">LDRB</td><td align="left">加载字节到寄存器</td></tr><tr><td align="left">LDRH</td><td align="left">加载半字到寄存器</td></tr><tr><td align="left">LDRSH</td><td align="left">加载半字到寄存器，再带符号扩展到 32 位</td></tr><tr><td align="left">LDM</td><td align="left">从一片连续的地址空间中加载多个字到若干寄存器</td></tr><tr><td align="left">LDRD</td><td align="left">从连续的地址空间加载双字（64 位整数）到 2 个寄存器</td></tr><tr><td align="left">STR</td><td align="left">存储寄存器中的字</td></tr><tr><td align="left">STRB</td><td align="left">存储寄存器中的低字节</td></tr><tr><td align="left">STRH</td><td align="left">存储寄存器中的低半字</td></tr><tr><td align="left">STM</td><td align="left">存储若干寄存器中的字到一片连续的地址空间中</td></tr><tr><td align="left">STRD</td><td align="left">存储 2 个寄存器组成的双字到连续的地址空间中</td></tr><tr><td align="left">PUSH</td><td align="left">把若干寄存器的值压入堆栈中</td></tr><tr><td align="left">POP</td><td align="left">从堆栈中弹出若干的寄存器的值</td></tr></tbody></table>

#### 0.1.3.4 32 位转移指令

<table><thead><tr><th align="left">名字</th><th align="left">功能</th></tr></thead><tbody><tr><td align="left">B</td><td align="left">无条件转移</td></tr><tr><td align="left">BL</td><td align="left">转移并连接（呼叫子程序）</td></tr><tr><td align="left">TBB</td><td align="left">以字节为单位的查表转移。从一个字节数组中选一个 8 位前向跳转地址并转移</td></tr><tr><td align="left">TBH</td><td align="left">以半字为单位的查表转移。从一个半字数组中选一个 16 位前向跳转的地址并转移</td></tr></tbody></table>

#### 0.1.3.5 其它 32 位指令

<table><thead><tr><th align="left">名字</th><th align="left">功能</th></tr></thead><tbody><tr><td align="left">LDREX</td><td align="left">加载字到寄存器，并且在内核中标明一段地址进入了互斥访问状态</td></tr><tr><td align="left">LDREXH</td><td align="left">加载半字到寄存器，并且在内核中标明一段地址进入了互斥访问状态</td></tr><tr><td align="left">LDREXB</td><td align="left">加载字节到寄存器，并且在内核中标明一段地址进入了互斥访问状态</td></tr><tr><td align="left">STREX</td><td align="left">检查将要写入的地址是否已进入了互斥访问状态，如果是则存储寄存器的字</td></tr><tr><td align="left">STREXH</td><td align="left">检查将要写入的地址是否已进入了互斥访问状态，如果是则存储寄存器的半字</td></tr><tr><td align="left">STREXB</td><td align="left">检查将要写入的地址是否已进入了互斥访问状态，如果是则存储寄存器的字节</td></tr><tr><td align="left">CLREX</td><td align="left">在本地的处理上清除互斥访问状态的标记（先前由 LDREX/LDREXH/LDREXB 做的标记）</td></tr><tr><td align="left">MRS</td><td align="left">加载特殊功能寄存器的值到通用寄存器</td></tr><tr><td align="left">MSR</td><td align="left">存储通用寄存器的值到特殊功能寄存器</td></tr><tr><td align="left">NOP</td><td align="left">无操作</td></tr><tr><td align="left">SEV</td><td align="left">发送事件</td></tr><tr><td align="left">WFE</td><td align="left">休眠并且在发生事件时被唤醒</td></tr><tr><td align="left">WFI</td><td align="left">休眠并且在发生中断时被唤醒</td></tr><tr><td align="left">ISB</td><td align="left">指令同步隔离（与流水线和 MPU 等有关——译注）</td></tr><tr><td align="left">DSB</td><td align="left">数据同步隔离（与流水线、MPU 和 cache 等有关——译注）</td></tr><tr><td align="left">DMB</td><td align="left">数据存储隔离（与流水线、MPU 和 cache 等有关——译注）</td></tr><tr><td align="left">DMB</td><td align="left">数据存储器隔离。DMB 指令保证： 仅当所有在它前面的存储器访问操作都执行完毕后，才提交 (commit) 在它后面的存储器访问操作。</td></tr><tr><td align="left">DSB</td><td align="left">数据同步隔离。比 DMB 严格： 仅当所有在它前面的存储器访问操作都执行完毕后，才执行在它后面的指令（亦即任何指令都要等待存储器访 问操作——译者注）</td></tr><tr><td align="left">ISB</td><td align="left">指令同步隔离。最严格：它会清洗流水线，以保证所有它前面的指令都执行完毕之后，才执行它后面的指令。</td></tr></tbody></table>

#### 0.1.3.6 立即数

1、立即数：一个立即数是一块数据存储作为指令本身，而不是在一个中的一部分内容存储器位置或寄存器。立即值通常用于加载值或对常量执行算术或逻辑运算的指令。  
2、比如一个数 10，把他存入内存中，高级语言表示法是 int i=10，这个数放入内存之前叫立即数，放入之后就不是了，再比如一个数 10，把他存入寄存器中，这个数放入寄存器之前叫立即数，放入之后就不是了。

#### 0.1.3.7 逻辑数

逻辑数是用来表示二值逻辑中的 "是" 与 "否"、或称 "真" 与 "假" 两个状态的数据。在计算机中，可以用一位基 2 码表示逻辑数据，即 8 个逻辑数据可以存放在 1 个字节中，可用其中的每个 bit（位）表示一个逻辑数据。逻辑数可以用计算机中的基 2 码的两个状态 "1" 和 "0" 来表示，其中 "1" 表示真，"0" 表示假。

#### 0.1.3.8 逻辑运算和算术运算

逻辑运算是一种只存在于二进制中的运算。在计组中逻辑运算经常出现的是 或、与、非和异或，这几种运算方式。  
算数运算我们平常十进制的 加减乘除，但因为在计算机中是二进制所以就只能是加法运算。在计算机中也可以算数运算也可以区分成进位的算数运算和不进位的算数运算。带进位的算数运算

### 0.1.4 三、实例讲解

#### 0.1.4.1 MRS

将状态寄存器 CPSR 或 SPSR 的内容移动到一个通用寄存器

```
MRS R0，CPSR                         //传送CPSR的内容到R0
MRS R0，SPSR                         //传送 SPSR的内容到R0
```

#### 0.1.4.2 MSR

将立即数或通用寄存器的内容加载到 CPSR 或 SPSR 的指定字段中

```
MSR CPSR，R0        //传送R0的内容到CPSR
MSR SPSR，R0        //传送R0的内容到SPSR
MSR CPSR_c，R0     //传送R0的内容到SPSR，但仅仅修改CPSR中的控制位域
```

#### 0.1.4.3 PRIMASK

用于 disable NMI 和硬 fault 之外的所有异常，它有效地把当前优先级改为 0（可编程 优先级中的最高优先级）。  
CPS 指令会更改 CPSR 中的一个或多个模式以及 A、I 和 F 位，但不更改其他 CPSR 位。CPSID 就是中断禁止，CPSIE 中断允许，

A：表示启用或禁止不精确的中止；I：表示启用或禁止 IRQ 中断；F：表示启用或禁止 FIQ 中断

#### 0.1.4.4 FAULTMASK

```
CPSIE f; / CPSID f;

MSR FAULTMASK,R0
```

FAULTMASK 更绝，它把当前优先级改为 - 1。这么一来，连硬 fault 都被掩蔽了。使用方案与  
PRIMASK 的相似。但要注意的是，FAULTMASK 会在异常退出时自动清零。

#### 0.1.4.5 BX 指令

BX{条件} 目标地址  
BX 指令跳转到指令中所指定的目标地址，目标地址处的指令既可以是 ARM 指令，也可以是 Thumb 指令。

#### 0.1.4.6 零寄存器 wzr、xzr

因为我们在使用 str 的是没法使用立即数 0 给寄存器赋值，所以 wzr xzr 就是干这个事情的。是一个比较特殊又常常见到的寄存器。

#### 0.1.4.7 立即寻址指令 MOV

```
SUBS R0,R0,#1   //R0 减 1 ，结果放入 R0 ，并且影响标志位
MOV R0,#0xFF000 //将立即数 0xFF000 装入 R0 寄存器 寄存器寻址指令举例如下： 
MOV R1,R2    //将 R2 的值存入 R1
SUB R0,R1,R2 //将 R1 的值减去 R2 的值，结果保存到 R0
```

#### 0.1.4.8 寄存器间接寻址指令 LDR

```
LDR R1,[R2]    //将 R2 指向的存储单元的数据读出保存在 R1 中 
SWP R1,R1,[R2] //将寄存器 R1 的值和 R2 指定的存储单元的内容交换，将R2的数值作为一个地址，将此地址处的数值与R1中的内容交换
```

```
按照从简单到复杂的分类方法，可以通过以下方式来指定访存指令的地址：从寄存器中获取地址；通过寄存器内容再加上偏移来获取地址；对偏移进行扩展、移位等运算之后，再与寄存器内容相加，获得地址。
LDR X0, [X1] ; 直接从寄存器X1的内容中获取地址。
LDR X0, [X1, #-8] ; X1的内容加上-8的偏移，得到地址。
LDR X0, [X1, X2] ; X!的内容和X2的内容相加得到地址。
LDR X0, [X1, W2, SXTW] ; 对W2的内容做符号扩展，再与X1的内容相加，作为地址。
LDR X0, [X1, X2, LSL #2] ; 把X2的内容左移2位，再与X1的内容相加，作为地址
```

#### 0.1.4.9 寄存器移位寻址指令 LSL

```
MOV R0,R2,LSL #3     //R2 的值左移 3 位，结果放入R0 ，即是R0=R2×8 
ANDS R1,R1,R2,LSL R3  //R2 的值左移 R3 位，然后和R1相“与”操作，结果放入R1
```

```
寻址模式

简单模式：X1的内容不会被改变，例如。
LDR X0, [X1]
LDR X0, [X1, #4]

前变址模式，X1的内容在load之前变化，例如。
LDR X0, [X1, #4]!
等价于
ADD X1, X1, #4
LDR X0, [X1]

后变址模式，X1的内容在load之后变化，例如。
LDR X0, [X1], #4
等价于
LDR X0, [X1]
ADD X1, X1, #4

支持对整型、浮点标量和向量，要求源寄存器和目的寄存器必须具有相同的宽度，例如。
LDP W3, W7, [X0] ; [X0] => W3, [X0 + 4 bytes] =>W7
STP Q0, Q1, [X4] ; Q0 => [X4], Q1=>[X4 + 16 bytes]
```

#### 0.1.4.10 基址寻址指令 STR

```
LDR R2,[R3,#0x0C]  //读取 R3+0x0C 地址上的存储单元的内容，放入 R2 
STR R1,[R0,#-4]! 	//先 R0=R0-4 ，然后把 R1 的值寄存到 R0 指定的存储单元
```

#### 0.1.4.11 多寄存器寻址指令

```
LDMIA R1!,{R2-R7,R12}   //将 R1 指向的单元中的数据读出到R2 ～R7、R12 中 (R1自动加1) 
STMIA R0!,{R2-R7,R12}    //将寄存器 R2 ～ R7 、 R12 的值保存到 R0 指向的存储单元中(R0自动加1)
```

#### 0.1.4.12 无条件转移 B，BAL

举例： B LABEL ; LABEL 为某个位置

```
CMP      x3,x4
B.CS     {pc}+0x10 ; 0xc000800094
```

BCC 是指 CPSR 寄存器条件标志位为 0 时的跳转。结合 CMP R3, R1，意思是比较 R3 R1 寄存器，当相等时跳转到环测试。因为 CMP 指令减去两个值并在 CPSR 中设置条件标志位。

#### 0.1.4.13 条件转移 B.cont

```
BEQ 相等
BNE 不等
BPL 非负
BMI 负
BCC 无进位
BCS 有进位
BLO 小于（无符号数）
BHS 大于等于（无符号数）
BHI 大于（无符号数）
BLS 小于等于（无符号数）
BVC 无溢出（有符号数）
BVS 有溢出（有符号数）
BGT 大于（有符号数）
BGE 大于等于（有符号数）
BLT 小于（有符号数）
BLE 小于等于（有符号数）
```

```
blr Xm：跳转到由Xm目标寄存器指定的地址处，同时将下一条指令存放到X30寄存器中。例如：blr x20.
br Xm：跳转到由Xm目标寄存器指定的地址处。不是子程序返回
ret {Xm}：跳转到由Xm目标寄存器指定的地址处。是子程序返回。Xm可以不写，默认是X30.
```

#### 0.1.4.14 WFE 和 WFI 对比

wfi 和 wfe 指令都是让 ARM 核进入 standby 睡眠模式。wfi 是直到有 wfi 唤醒事件发生才会唤醒 CPU，wfe 是直到 wfe 唤醒事件发生，这两类事件大部分相同。唯一不同之处在于 wfe 可以被其他 CPU 上的 sev 指令唤醒，sec 指令用于修改 event 寄存器的指令。

WFE  
Wait For Event，是否实现此指令是可选的。如果此指令未实现，它将作为 NOP 指令来执行。如果指令作为 NOP 在目标处理器上执行，汇编程序将生成诊断消息。

SEV  
Set Event，其是否实现是可选的。如果未实现，它将作为 NOP 执行。如果指令作为 NOP 在目标上执行，汇编程序将生成诊断消息。  
SEV 在 ARMv6T2 中作为 NOP 指令执行。

#### 0.1.4.15 MRC：协处理器寄存器到 ARM 寄存器的数据传输

MRC 指令将协处理器的寄存器中数值传送到 ARM 处理器的寄存器中。如果协处理器不能成功地执行该操作，将产生未定义的指令异常中断。

```
MRC p2,5,r3,c5,c6协处理器p2把c5和c6经过5操作的结果赋给r3
MRC p3,9,r3,c5,c6,2协处理器p3把c5和c6经过9操作（类型2）的结果赋给r3
```

#### 0.1.4.16 MCR：寄存器到协处理器寄存器的数据传输

MCR 指令将 ARM 处理器的寄存器中的数据传送到协处理器的寄存器中。如果协处理器不能成功地执行该操作，将产生未定义的指令异常中断。

```
MCR p6,0,r4,c5,c6协处理器p6把r4执行0操作后将结果存放进c5
```

#### 0.1.4.17 STM：将指令中寄存器列表中的各寄存器数值写入到连续的内存单元中

STM 指令是 Store Multiple 的缩写，它的作用是将多个寄存器的值保存到栈中。在 ARM 汇编中，栈是一种后进先出 (LIFO) 的数据结构，用来存储临时数据和函数调用过程中的返回地址

STM 指令的语法如下:

```
STM{条件码}{模式} SP!,{寄存器列表}
```

其中，条件码是可选项，用来指定条件执行 STM 指令的条件; 模式用来指定存储模式，

**1、寻址模式（mode）**  
mode 决定了基址寄存器是在执行指令前地址增减还是指令执行后增减.  
I 为 Increment(递增)  
D 为 Decrement (递减)  
B 为 Before  
A 为 After

常用的模式有 IA (递增后存储) 、IB (递增前存储) 、DA (减后存储) 和 DB(递减前存储);SP 是栈指针寄存器，用来指定栈的起始地址; 寄存器列表指定要保存的寄存器。  
另外四种也是寻址模式  
FD 慢递减堆栈  
FA 满递增堆栈  
ED 空递减堆栈  
EA 空递增堆栈

```
STMFD SP![RO,R1,R2)
```

在上述代码中，STMFD 指令存储了 RO、R1 和 R2 的值到栈中。SP! 表示栈指针寄存器递增，即存储完后栈指针自动增加，以便下一次保存操作。

**2、“!”**  
在传输数据完成后，更新基址寄存器中的值

**3、“^”**

在数据传输完成后，将 SPSR 的值复制到 CPSR 中，常用于异常模式下的返回.

#### 0.1.4.18 LDM：将数据从连续内存单元中读取到指令的寄存器列表中的各寄存器中

LDMIA

```
LDMIA R0! ,{R3-R9} ; //将R0指向的地址上连续空间的数据，保存到R3-R9当中，!表示R0值更新,IA后缀表示按WORD递增
```

#### 0.1.4.19 LDR：从内存中将一个 32 位的字读取到目标寄存器

```
ldr 加载指令： LDR{条件} 目的寄存器，<存储器地址>
```

LDR 指令用亍从存储器中将一个 32 位的字数据传送到目的寄存器中。该指令通常用于从存储器中读取 32 位的字数据到通用寄存器，然后对数据进行处理。当程序计数器 PC 作为目的寄存器时，指令从存储器中读取的字数据被当作目的地址，从而可以实现程序流程的跳转。

#### 0.1.4.20 STR：将 32 位字数据写入到指定的内存单元

STR 指令的格式为：

```
STR{条件} 源寄存器，<存储器地址>
```

STR 指令用亍从源寄存器中将一个 32 位的字数据传送到存储器中。该指令在程序设计中比较常用，寻址方式灵活多样，使用方式可参考指令 LDR。

```
STR R0，[R1]，＃8  ；将R0中的字数据写入以R1为地址的存储器中，并将新地址R1＋8写入R1。
STR R0，[R1，＃8]  ；将R0中的字数据写入以R1＋8为地址的存储器中。
str     r1, [r0]  ；将r1寄存器的值，传送到地址值为r0的（存储器）内存中
```

#### 0.1.4.21 SWI：软中断指令

SWI 指令格式如下：

```
SWI{cond} immed_24
```

MOV R0,#34 ；设置功能号为 34  
SWI 12 ；产生软中断，中断号为 12

#### 0.1.4.22 BIC 清除位

```
BIC指令的格式为： BIC{条件}{S} 目的寄存器，操作数1，操作数2
```

BIC 指令用于清除操作数 1 的某些位，并把结果放置到目的寄存器中。操作数 1 应是一个寄存器， 操作数 2 可以是一个寄存器、被移位的寄存器、或一个立即数。操作数 2 为 32 位的掩码，如果在 掩码中置了某一位 1，则清除这一位。未设置的掩码位保持不变。

```
BIC R0,R0,#0X1F
0X1F=11111B
//其含义：清除R0的bit[4:0]位。
BIC R4, R4, #0xFF000000 指令将E4高8位清除为0
```

#### 0.1.4.23 EOR 逻辑异或指令

```
EOR{<cond>}{S} <Rd>,<Rn>,<shifter_operand>
```

逻辑异或 EOR（Exclusive OR）指令将寄存器中的值和 <shifter_operand> 的值执行按位 “异或” 操作，并将执行结果存储到目的寄存器中，同时根据指令的执行结果更新 CPSR 中相应的条件标志位。

#### 0.1.4.24 CMN 与负数对比

CMN 同于 CMP，但它允许你与负值进行比较，比如难于用其他方法实现的用于结束列表的 -1。这样与 -1 比较将使用:  
CMN R0, #1 ; 把 R0 与 -1 进行比较

#### 0.1.4.25 MVN 取反

将每一位操作数都取反，若为有符号的数据则进行补码保存

```
MVN R0 0x4
```

其中上图中的 0x4 用二进制数 (00000100) 表示, 然后对其取反得到（11111011），可见取反后为负数，因此针对负数求其补码则为储存在 R0 中的值，先将负数最高位转换为正数（01111011）取反，得到（10000100），加 1 得到其补码，最后结果为（10000101），即结果为 - 5；

#### 0.1.4.26 LSL（Logical Shift Left）左移运算

用于将寄存器的值向左移位，末尾填充 0。在 ARM 处理器中，每个寄存器都有 32 位，当 LSL 被使用时，指令将寄存器中的二进制数值向左移动指定的位数，并用 0 填充未使用的右侧位数。

#### 0.1.4.27 STP

STP 是一条用于将 General-Purpose Registers（通用寄存器）的值存储到内存地址的指令。STP 是 Store Pair 的缩写，用于同时将两个寄存器的值存储到连续的内存地址中。

```
STP <Rt1>, <Rt2>, [<Xn|SP>, #<imm>]
```

Rt1 和 Rt2 是要存储的寄存器，可以是 X0-X30 中的任何一个。  
Xn 是基础地址寄存器，可以是 X0-X30 中的任何一个，但不能是 Rt1 或 Rt2。  
#是一个常数值，用于指定一个偏移量，范围是 - 2048 到 2047。  
例如，以下是使用 STP 指令将 X2 和 X3 寄存器的值存储到基础地址寄存器 X18 指定的内存地址偏移 24 字节处的代码：

```
STP X2, X3, [X18, #24]!
```

注意：结尾的! 表示同时更新基础寄存器的值，即存储操作后，X18 将指向下一个地址。如果不需要更新基础寄存器，可以省略!

### 0.1.5 实例解析

```
IMPORT |Image$RW_IRAM1$Base|            //从别处导入data段的链接地址
IMPORT |Image$RW_IRAM1$Length|         //从别处导入data段的长度
IMPORT |Load$RW_IRAM1$Base|            //从别处导入data段的加载地址
IMPORT |Image$RW_IRAM1$ZI$Base|        //从别处导入ZI段的链接地址
IMPORT |Image$RW_IRAM1$ZI$Length|      //从别处导入ZI段的长度
Load$$region_name$$Base 	//Load address of the region.
Load$$region_name$$Length 	//Region length in bytes.
Load$$region_name$$Limit 	//Address of the byte beyond the end of the execution region.
```

```
//复制数据段
LDR R0, = |Load$RW_IRAM1$Base|           //将data段的加载地址存入R0寄存器
LDR R1, = |Image$RW_IRAM1$Base|   //将data段的链接地址存入R1寄存器
LDR R2, = |Image$RW_IRAM1$Length| //将data段的长度存入R2寄存器
CopyData                
SUB R2, R2, #4                      //每次复制4个字节的data段数据
LDR R3, [R0, R2]                    //把加载地址处的值取出到R3寄存器
STR R3, [R1, R2]                   //把取出的值从R3寄存器存入到链接地址                                        
CMP R2, #0                         //将计数和0相比较
BNE CopyData                      //如果不相等，跳转到CopyData标签处，相等则往下执行

//清除BSS段
LDR R0, = |Image$RW_IRAM1$ZI$Base|   //将bss段的链接地址存入R1寄存器
LDR R1, = |Image$RW_IRAM1$ZI$Length| //将bss段的长度存入R2寄存器
CleanBss        
SUB R1, R1, #4                                                //每次清除4个字节的bss段数据
MOV R3, #0                                                  //将0存入r3寄存器
STR R3, [R0, R1]                                        //把R3寄存器存入到链接地址                                        
CMP R1, #0                                                 //将计数和0相比较
BNE CleanBss                       //如果不相等，跳转到CleanBss标签处，相等则往下执行

IMPORT  mymain                                              //通知编译器要使用的标号在其他文件
BL                mymain                                    //跳转去执行main函数
B                .                                          //原地跳转，即处于循环状态
ENDP
ALIGN                                                      //填充字节使地址对齐
END                                                        //整个汇编文件结束
```