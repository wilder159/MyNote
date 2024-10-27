> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/m0_73936730/article/details/136780208?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_utm_term~default-0-136780208-blog-123747306.235^v43^pc_blog_bottom_relevance_base5&spm=1001.2101.3001.4242.1&utm_relevant_index=3)

**目录**

[前言](#t0)

[1. 下载 DOSBOX](#t1)

 [2.debug.exe 下载](#t2)

[3. 配置 debug.exe 和 DosBox](#t3)

[3.1 配置 debug.exe](#1.%E9%85%8D%E7%BD%AEdebug.exe)

[3.2 配置 DosBox](#%C2%A02.%E9%85%8D%E7%BD%AEDosBox)

[窗口大小设置](#%E7%AA%97%E5%8F%A3%E5%A4%A7%E5%B0%8F%E8%AE%BE%E7%BD%AE)

 [4.MASM 配置](#t4)

[​编辑](#t5)

[结语](#t6)

#### 0.1.1.1 前言

学习汇编语言时，需要进入 dos 模式并使用 debug 工具调试。但是 64 位 [win10 系统](https://so.csdn.net/so/search?q=win10%E7%B3%BB%E7%BB%9F&spm=1001.2101.3001.7020)没有自带这些工具。因此，需要额外安装 DosBox 和 debug.exe 这两个软件。因为学校选修了汇编语言这门课，就写个博客防止忘记搭建，顺便给大家提供参考。

#### 0.1.1.2 下载 DOSBOX

> DosBox 是一个基于 DOS 的 x86 [模拟器](https://so.csdn.net/so/search?q=%E6%A8%A1%E6%8B%9F%E5%99%A8&spm=1001.2101.3001.7020)，它允许在现代操作系统上运行 DOS 程序。，由于它采用的是 SDL 库，所以可以很方便的移植到其他的平台。DOSBox 的最新版本已经支持在 Windows、[Linux](https://upimg.baike.so.com/doc/5349227-5584683.html "Linux")、Mac OS X、[BeOS](https://upimg.baike.so.com/doc/5443949-5682303.html "BeOS") 、palmOS、Android 、webOS、os/2 等系统中运行。DosBox 提供了一个方便的方式来模拟 DOS 环境，以便编写、调试和运行汇编程序。

> DOSBOX 官网 [DOSBox, an x86 emulator with DOS](https://www.dosbox.com/download.php?main=1 "DOSBox, an x86 emulator with DOS")

> 自己选择要的版本，我的是 Windows。
> 
> (不想进去的可以去后面 debug.exe 的链接下载，里面有安装。)

![[_resources/未命名 1/44ff614aec777110852acd9419d927b1_MD5.png]]

> 点进去会出现倒计时，倒计时结束时就会自动帮你下载 

![[_resources/未命名 1/6eef854c59519a5db41b2400288d400b_MD5.png]]

![[_resources/未命名 1/833def8c01a5f920d0974315d61b95bc_MD5.png]]

 安装完成后可能会出现窗口过小的问题，后面会讲如何调试。

![[_resources/未命名 1/5226aaa7155ab25eaf9dfa0a48c4b3e9_MD5.png]]

#### 0.1.1.3  2.debug.exe 下载

这个直接下载就行，无需安装。

> gitee：[汇编语言编程环境. rar · 宋辞 / DOSBOX - 码云 - 开源中国 (gitee.com)](https://gitee.com/xiaopang1117/dosbox/blob/master/%E6%B1%87%E7%BC%96%E8%AF%AD%E8%A8%80%E7%BC%96%E7%A8%8B%E7%8E%AF%E5%A2%83.rar "汇编语言编程环境.rar · 宋辞/DOSBOX - 码云 - 开源中国 (gitee.com)")
> 
> 百度网盘：链接：https://pan.baidu.com/s/1G26TPIi5z_hmbIYC6tghSA   
> 提取码：kbns 

#### 0.1.1.4 
3. 配置 debug.exe 和 DosBox

##### 0.1.1.4.1 配置 debug.exe

![[_resources/未命名 1/a927c001883e4ab86663899ad45f4e1b_MD5.png]]

在 DosBox **所在盘的根目录**下创建一个 DEBUG，粘贴进去。![[_resources/未命名 1/853c615610bc6c8e5c6321b066b80c21_MD5.png]]

![[_resources/未命名 1/b11991d75eb954ef0d6735b55725eb2a_MD5.png]]

##### 0.1.1.4.2 配置 DosBox

###### 0.1.1.4.2.1 窗口大小设置

之前我说过刚安装的 DosBox 窗口过小的问题。

![[_resources/未命名 1/5226aaa7155ab25eaf9dfa0a48c4b3e9_MD5.png]]

> 找到之前我们 DosBox 安装的路径，用记事本打开如图文件。

![[_resources/未命名 1/b7a1dc04b6982522615733114cc1feab_MD5.png]]

 修改这两个地方，第一个是窗口大小设置

这里给一下我感觉合适的大小，你们可以自行调整。

> windowresolution=1600x700
> 
>   
> output=opengl

![[_resources/未命名 1/c0fee28746333727665c15025a7cd580_MD5.png]]

然后划到最底部，输入：（大小写都行）

> mount d d:\debug    mount d d:\（自己之前新建的 debug 文件位置）  
> d:                             （自己的 debug 文件所在磁盘）

 ![[_resources/未命名 1/5de2742ee9d50821641146f7e9bcf411_MD5.png]]

保存后，打开 DosBbox。

![[_resources/未命名 1/21612bea49849e6414f06a9a47918d40_MD5.png]]

这样就算成功了。

输入 debug，然后出现一条不断跳动的横线就可以了。

![[_resources/未命名 1/5f0552eb95922fcb15a400f3d205419a_MD5.png]]

![[_resources/未命名 1/878f3f9afdf54549c154a2763fefbcb6_MD5.png]]

#### 0.1.1.5  4.MASM 配置

#### 0.1.1.6 ![[_resources/未命名 1/c531de8a6377f0b60ef11e36ab00e81e_MD5.png]]

这和之前 Debug 一样，我们需要将文件配置在 Dosbox 相应的根目录硬盘下，然后将之前的语句改成

> mount d d:\MASM    mount d d:\（自己之前新建的 debug 文件位置）  
> d:                             （自己的 debug 文件所在磁盘）

 就能运行成功了。

#### 0.1.1.7 结语

到这里我们就可以开始汇编语言的学习了，希望对大家有帮助。

![[_resources/未命名 1/f9e1bfd441ab1a046425046539175870_MD5.gif]]