> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qwe5959798/article/details/122835924)

接上一篇：

[函数调用](https://blog.csdn.net/qwe5959798/article/details/122717894?spm=1001.2014.3001.5501 "函数调用")

由上一篇大概了解了函数是如何被调用，中断或者说异常又是如何被调用，而这一篇相当于上一篇知识的一个应用，也是上一篇遗留的思考，即在 hardfault 中如何判断是从何处触发这个异常的。本来打算自己写 demo，但是查到 github 上有一个开源的 CmBacktrace，既然有大牛已经写了开源的库，就直接拿来分析印证吧。项目地址：

[CmBacktrace](https://github.com/armink/CmBacktrace "CmBacktrace")

硬件我使用的是 [STM32F103ZET6 最小系统](https://so.csdn.net/so/search?q=STM32F103ZET6%E6%9C%80%E5%B0%8F%E7%B3%BB%E7%BB%9F&spm=1001.2101.3001.7020)板，demo 是项目中提供的，直接下载即可。

1. 原理分析
-------

从串口 1 输出如下：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/eadd4f633cef8aed387ef9081374a1a5_MD5.png]]

实际使用项目中提供的 [addr2line](https://so.csdn.net/so/search?q=addr2line&spm=1001.2101.3001.7020) 运行打印出来的地址时，输出如下：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/faf1c61dd0b71341b7bcc7f59eb488a2_MD5.png]]

我们回过头来看一下 demo，在 app.c 中的 main 里有调用 fault_test_by_div0() 这个函数，它里面故意做了除 0 运算：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/3bc1b504fd442c06b64459d4314af2b5_MD5.png]]

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/204b03c32c422ff6269a866d2e83ecf1_MD5.png]]

 仔细对比 addr2line 的输出，我们可以看到，输出结果是完全正确的，包括行号（有空行的情况可能会有点偏差）。其实 addr2line 这个程序的作用是通过你输入的地址和.[elf 文件](https://so.csdn.net/so/search?q=elf%E6%96%87%E4%BB%B6&spm=1001.2101.3001.7020)，输出这个地址在. elf 文件中表示的函数。原理不难理解，elf 文件中完美保留了每个函数的地址，以及每个函数对应的. c 文件，自然可以去对应. c 中查看该函数行号。可以参考我之前分析 elf 文件的博文：

[(GCC)STM32 进阶修炼之 ELF 文件剖析](https://blog.csdn.net/qwe5959798/article/details/123798730 "(GCC)STM32进阶修炼之ELF文件剖析")

所以这里的关键其实在于我们给 addr2line 传入的三个地址：

addr2line -e CmBacktrace.axf -a -f **08001da6 08001dfc 08000268**

打开. axf 文件可以看到以上三个地址分别对应的函数：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/e8a8d8e80f611c2213f0373d65550e84_MD5.png]]

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/70a54d9974e71b31fbc5e416cf120f24_MD5.png]]

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/deb51017e42521382799f665d62b7735_MD5.png]]

可以看到最后一个地址指向的 **08000268** 其实在源文件中是找不到的而且它保存的也不是某行代码，而是一个全局变量，所以 addr2line 打印的是问号，表示查找不到。

从这里可以看出，我们的关键就在于找到出错误代码的地址，以及调用这个出错误函数的地方，当然，实际可能有很多个函数层层嵌套调用，而只需要依次找到上一层调用地址，就可以一层一层的递进。

这里，我会再次分析整个函数调用流程，是对上一节内容的一个印证。**请注意栈中保存数据在整个流程中的变化**！

首先我们把断点打在 main 函数的起始位置，此时现场如下：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/77471e4a5d4639ed9426a69f29be9f3c_MD5.png]]

由上图我们可以知道几点：

1. 调用 mian 函数结束后， 我们返回到调用 main 函数的地方，并执行下一个指令，它的地址是 0x08000268（为什么减一我前几章有说过），这个地址实际对应：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/59e789765a27ff81ee4882a191c4dca0_MD5.png]]

实际这个地址保存的根本不是可以执行的代码，那为什么还要指明从 main 执行完后回来执行它？

由上图可以看到，我们实际跳转 main 是 0x08000264，这里使用的指令是 **BL**，这个指令会自动装载下一个指令地址到 LR（即执行时 PC 指向的地址），保证调用完函数后，可以很方便的直接用汇编指令：**B LR** 返回到上层函数，然而实际这个工程中，main 函数根本不会返回，所以这个地址即使是错的，也无所谓。

2. 因为我们 main 函数中还要再使用汇编指令 **BL** 和寄存器 R4，所以我们需要把它们保存在栈里，防止被覆盖，所以可以看到 main 函数第一行汇编就是把 LR 和 R4 保存在了栈里。而此时的栈顶是 SP 所指向的 0x2000 0560，打开. map 文件可以看到：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/70cc57f5933446f2e8040ee624a340ae_MD5.png]]

我们分配给栈的空间是从 0x2000 0160 开始，大小为 0x400 的空间，因为栈是从上到下生长的，所以栈顶一开始就是 0x2000 0560。 

我们再次推进断点位置，这次把断点打在进入 fault_test_by_div0 函数前，具体现场如下：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/acdc7f1aa39caa10d4e5d17b149933c0_MD5.png]]

可以看到，根据上文的分析，LR 和 R4 已经被压栈，此时栈顶已经变成了 0x20000558，我们继续向下推进， 这次把断点打在 fault_test_by_div0 函数一开始的位置：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/0726b81c2c2ea2f5054659f6720cc130_MD5.png]]

继续把断点打在出错的那一行函数，继续运行分析：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/7615e84b6485fecc3aa194406ff64c80_MD5.png]]

对比上面两张图可以看到，栈内数据和我们分析的一模一样，现在我们即将运行错误代码，我们知道，Cortex-M 系列的 MCU 出现错误会跳往编号为 3 的中断：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/20e8b3d8e4505ec269502bf249611ee2_MD5.png]]

这里我们找到中断表，最后可以查到跳转到了 HardFault_Handler 函数，我们把下个断点打在这里：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/0e3a08a0d275bb7bf4786aa83ac16192_MD5.png]]

可以看到，出现错误后和我们预期一样，注意两个地方：

1. 进入中断后，LR 的值会被自动更新为特殊的 EXC_RETURN，这个值的含义如下：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/b4017d10458124f62d16f6094a818a09_MD5.png]]

也就是它告诉了我们，这个中断执行完后，我们返回时该使用何种模式，和使用哪个堆栈指针。

2. 正因为 LR 因为上述原因被占用了，导致我们没有办法再直接通过 LR 返回到中断被调用前的位置，所以进入中断后，一部分寄存器是硬件自动入栈的：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/aab0a102c641a6cb02f880f6a066c77c_MD5.png]]

由进入 HardFault_Handler 后的现场图也可以看到，压栈的内容与寄存器是一一对应的。也就是说硬件替我们保存了一部分现场，你可能会问了，为何还有一部分寄存器没有被保存，万一在进入中断前我使用过它们比如 R5，中断中被破坏了，代码从中断中出来后，再回去执行不是一样会出错吗？这在上一章有讲过，函数调用原则里有分哪些寄存器是需要调用者保护，哪些寄存器是需要被调用者保护的，这里除了硬件压栈的那部分，剩余的都应该由被调用者保护。

这样是不是一切都明了了？思考这样一个问题：

如果此时让你在 HardFault_Handler 中写一个函数，用它去寻找是由哪条指令执行后，导致进入了 HardFault_Handler，而那个指令所在的函数又是被哪里所调用，你会写吗？

一切一切的关键又回到了那三个地址：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/146d7019eccabbf71c4fd524d32d77f1_MD5.png]]

对，就是如何从栈中找到这三个地址！

至于为什么可以打印出错误类型是除 0，那很简单，即使你没有移植任何代码，在 keil 中 HardFault_Handler 打上断点，进入后，查看：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/51bbe540b99150a190e3dbd06cd98cb2_MD5.png]]

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/df35f673755cc7d6301a70c7c483c6b2_MD5.png]]

简单来说就是 Cortex-M 内核提供了一些寄存器，它们保存出错时，错误的大概类型。只需在进入错误中断后查询相关寄存器即可。

听起来这一切都很简单是不是？但是想写好一个开源库却比想象中难。

2. 代码分析
-------

就以上文中 demo 里不使用 OS 的情况为例，分析整个代码流程。

首先看 main 函数，和 CmBacktrace 相关的初始化就一个函数：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/2c23ee1dd2146ddc0558c3b5e627d5da_MD5.png]]

而这个函数里面很简单，仅仅是赋值了一些变量：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/7e69d619c599454aaa23b98c55161e7a_MD5.png]]

这些变量定义如下： 

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/5a8ac129577f86b207ba15ecf4fb5396_MD5.png]]

这里你需要知道一些 MDK 的知识。在 MDK 中，使用 **AREA** 关键字创建的数据段，通常在段名后加 **$$Base** 表示起始地址，加 **$$Limit** 表示结束地址，比如这里的 STACK：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/40c795712f2492ab0e9997dfba2d832b_MD5.png]]

以及：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/fa2d7a8634d9608945206b05b737ac7d_MD5.png]]

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/0927f165e2d7282799de782cdcab3ab2_MD5.png]]

所以这里的这些宏定义起始就是为了取得其中两个段：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/b1288769c7fa27fe63f0146ba129d7b7_MD5.png]]

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/55eb842240b52c4958016836fbadf0af_MD5.png]]

由最开始的解析我们知道，这里查找函数调用的思路就是从栈区找到属于代码段范围的地址。这也就是为什么我们一开始要知道栈区范围和代码段范围。初始化就这么简单就完成了，主要工作都在错误中断中：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/a44ce0d6355744286d11f92f272ff9aa_MD5.png]]

这里就是我上一章讲到的函数调用规则，当汇编调用 C 函数时，需要遵守这个规则。

现在进入到关键函数 cm_backtrace_fault 里：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/cb764241da4aa14ca5a3fd3d9b915d5d_MD5.png]]

上述代码都是一些简单的判断与打印，不再详细展开， 看一下下面的打印栈区：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/6f76c59ea1990c0da1947fffdd425bee_MD5.png]]

这里如果有堆栈溢出，且上层函数栈顶已经超过栈区最大地址，那什么也不会打印，具体打印数据逻辑如下：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/a56dfe7d54e2bb27415ea5b889a00760_MD5.png]]

这里的相关打印，我再贴一下：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/8f628c0cad3b6da189719559887b03cb_MD5.png]]

这和前文中原理分析相关内容，可以一一对应着去看。 下面则是打印的硬件压栈的那些寄存器数据：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/a9b907423ad030ce536fb39df9a2ec26_MD5.png]]

实际串口打印如下：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/3d40b15ed6c8f0a1e319c5b96fa17802_MD5.png]]

接下去就是我说的，通过查找寄存器对应的每个位，来判断当前故障的类型：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/b07051f42fa54e56233f80019c95dd6e_MD5.png]]

具体不再分析，想要知道细节的可以查看《Cortex-M3 权威指南》表 D.17 之后的几个表。

最关键的函数留在了最后：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/21812c2c1989dccab1aedaa6c04cc9c7_MD5.png]]

我们最后打印的就是：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/9c1cc75f6dc79f1c503fc623edef83f2_MD5.png]]

所以关键在函数 cm_backtrace_call_stack 中：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/c3a4b51f5aad79124d3a1e7088137aaa_MD5.png]]

在没有堆栈溢出的情况下，buffer 里面会先保存两个值，一个是执行错误代码时的 PC，在这个 demo 中是 0x08001da6，然后又保存了执行错误代码时的 LR-1，在这里是：0x08001dfd-1=0x08001dfc。

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/4f2ffb099ed1845769bb9d6b6ce4c467_MD5.png]]

最后就是循环检测栈区，然后把符合的保存下来。  

3.GCC 下使用该库
-----------

其实和 MDK 差不多，但是看过我置顶的那篇讲内存的文章会知道，CubeMX 生成的工程中，栈空间大小并非是我们分配的大小。详细参考：

[(GCC)STM32 基础详解之内存分配](https://blog.csdn.net/qwe5959798/article/details/122562894?spm=1001.2014.3001.5502 "(GCC)STM32基础详解之内存分配")

所以我们最后看下 GCC 下该库需要如何使用。这里只是简单讲一下，因为原理已经讲清楚了，具体的修改可自行解决。

主要在初始化时，获取栈起始地址和栈大小，以及代码段起始地址和代码段大小：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/78ef8c25e031ef18d4413c1d35b305eb_MD5.png]]

它们的定义如下：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/e54f7a937e1252c114507572ba142094_MD5.png]]

但是我们随便使用 CubeMX 创建一个 STM32F103ZET6 的 GCC 工程，打开. ld 文件会发现 

找不到_sstack 和_stext，只能找到结尾：

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/2135b7436e153c321f2110fb64e98cf2_MD5.png]]

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/a48750d88ba6a0b54d826f45ef86df4f_MD5.png]]

我们可以这样修改：

​​​​​​​ 

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/79e8923017e71eb6e6e6c6185ffb4045_MD5.png]]

![[../_resources/GCC)STM32 进阶详解之栈回溯_stm32 栈回溯 - CSDN 博客/29362bc34da737462926399c361037f2_MD5.png]]