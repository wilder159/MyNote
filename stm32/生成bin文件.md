> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/Suzkfly/p/16484631.html)

　　参考自：[https://blog.csdn.net/XieWinter/article/details/96475510](https://blog.csdn.net/XieWinter/article/details/96475510)

　　众所周知 keil 编译可以生成 hex 文件，但是有时需要用 bin 文件，那是不是要用一个工具将 hex 文件转换为 bin 文件。

　　好消息是 keil 安装的时候自带了转换软件，不过它不是将 hex 转为 bin，而是将 axf 转为 bin。

　　在目标选项中的 User->After Build/Rebuild 勾选 Run#1，在框中填入下面这一行：

```
D:\Keil_v5\ARM\ARMCC\bin\fromelf.exe --bin --output Objects/@L.bin Objects/@L.axf
```

　　如下图：

![[../_resources/生成bin文件/d7105e4011b558d1d583dfe662ac196d_MD5.png]]

 　　这个意思就是编译完程序后利用 fromelf.exe 这个工具，将 @L.axf 转换为 @L.bin，@L 意思是与工程名相同，也可以直接指定名字。因此重新编译工程就可以生成 bin 文件了。

　　使用的时候需要注意 fromelf.exe 所在的路径与用户安装的路径有关，根据实际情况修改。另外执行这条命令的路径是工程所在的路径，因此 axf 文件所在的路径是相对于工程文件的路径，也要根据实际情况修改。如果还是编译不过就检查是不是多加了空格或者少加了空格之类的。