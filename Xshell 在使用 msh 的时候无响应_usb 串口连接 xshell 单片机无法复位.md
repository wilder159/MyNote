> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_27508477/article/details/102589292)

在使用 [Xshell](https://so.csdn.net/so/search?q=Xshell&spm=1001.2101.3001.7020) 开发正点原子的战舰 V3 的时候，下载程序或者复位单片机后无响应，在 [RTT 官方文档](https://www.rt-thread.org/document/site/tutorial/quick-start/stm32f407-atk-explorer/quick-start/)看到有如下说明：

> 注：正点原子一键下载电路和终端工具冲突，在使用终端工具如：[PuTTy](https://so.csdn.net/so/search?q=PuTTy&spm=1001.2101.3001.7020)、XShell 时，会出现系统不能启动的问题，推荐使用串口调试助手如：sscom

在[开源电子网论坛](http://www.openedv.com/forum.php?mod=viewthread&tid=71137)看到原子哥的解释：

> 因为一键下载电路的问题。超级终端之类的软件，会控制 DTR/RTS，导致 B0 被拉高，然后你按复位，直接就进入 bootloader 模式了。。。。

而 Xshell 中的这个流控又不知道怎么设置取消……  
![](https://i-blog.csdnimg.cn/blog_migrate/be162d35d077ba197fa554b6f760a363.png)  
个人又尝试了 SecureCRT，同样有这个问题……

~最终放弃使用优雅的 xhell 工具……~

不，我还没有放弃，打开开发指南：  
![](https://i-blog.csdnimg.cn/blog_migrate/4592c8234193be8fa48577be5709c6db.png)  
![](https://i-blog.csdnimg.cn/blog_migrate/7e3ac03b22a2e75a3e60712d4ff9cdd5.png)  
既然是因为 BOOT0 引脚被拉高的原因…… 那么……  
把 R49 拆了不就行了嘛……![](https://i-blog.csdnimg.cn/blog_migrate/9024ea4519da2b294d40fb1c905a87f9.jpeg)  
于是 R49 就被博主拆了，开心的 reboot~  
![](https://i-blog.csdnimg.cn/blog_migrate/1c42e3a6b861f0170bfacdc064ef2187.png)