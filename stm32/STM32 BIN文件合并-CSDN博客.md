---
title: "STM32 BIN文件合并-CSDN博客"
source: "https://blog.csdn.net/niepangu/article/details/48524211"
author:
published:
created: 2025-03-08
description: "文章浏览阅读1.3w次，点赞2次，收藏13次。合并BIN文件的两种方法      在单片机的开发过程中，经常需要将两个单独的BIN文件合并成一个文件，方便烧写和生产。下面结合STM32的IAP Bootloader Code和Application Code的合并，介绍两种合并BIN文件的方法。      首先简单介绍一下STM32的IAP。IAP(In-application-programming)，即在应用中编程。有了它，产_bin文件合并"
tags:
  - "clippings"
---
在单片机的开发过程中，经常需要将两个单独的BIN文件合并成一个文件，方便烧写和生产。下面结合STM32的IAP Bootloader Code和Application Code的合并，介绍两种合并BIN文件的方法。

      首先简单介绍一下STM32的IAP。IAP(In-application-programming)，即在应用中编程。有了它，产品发布之后，仍然可以方便的升级固件，而不需要拆机并用JTAG等方式更新程序。IAP系统的固件一般有两部分组成，IAP BootLoader Code和Application Code，如下图所示。

  [![image](https://i-blog.csdnimg.cn/blog_migrate/7b4843f67fae219c4e8e8cef17a77961.png "image")](http://images.cnblogs.com/cnblogs_com/we-hjb/201101/201101062100172964.png)

      系统启动时，首先运行IAP BootLoader Code，并检测相应状态，判断是执行升级的流程还是直接运行本地的Application Code。 一般来说，BootLoader和Application是分别编译的，会生成两个二进制文件。在工厂生产时，如果分别烧写这两个文件，显然有些麻烦。这时，我们就可以将两个BIN文件合并成一个，直接烧写。假设Application Code的偏移地址为0x1000，IAP固件在Flash中的分布如下图所示。

   [![image](https://i-blog.csdnimg.cn/blog_migrate/299492421aee8ee8684d9008bc438fca.png "image")](http://images.cnblogs.com/cnblogs_com/we-hjb/201101/201101062100174326.png)

      下面介绍第一种方法，使用二进制文件合并工具(UBIN.exe)，这个小工具是以前在S3C2410上开发uCOS时做的，功能比较简单，满足一般的需求。

      首先，添加第一个文件1.bin，其地址为0x0000，所以，偏移量设置为0x00000000，设置完偏移量后点击“添加”按钮。

   [![image](https://i-blog.csdnimg.cn/blog_migrate/928f64fa46ef184aa6ebfa720047b20b.png "image")](http://images.cnblogs.com/cnblogs_com/we-hjb/201101/201101062100183388.png)

      然后添加第二个文件，偏移量根据需要设置为0x00001000，如下图所示。

   [![image](https://i-blog.csdnimg.cn/blog_migrate/eddcc26266cc446e245712d2b5a6c3a3.png "image")](http://images.cnblogs.com/cnblogs_com/we-hjb/201101/201101062100187782.png) 

      设置目标文件为C:\\dst.bin，然后点击“合并”按钮。

   [![image](https://i-blog.csdnimg.cn/blog_migrate/644dafd465690cfb2ce61da147d14ccf.png "image")](http://images.cnblogs.com/cnblogs_com/we-hjb/201101/20110106210019224.png)

      正常情况下，会成功生成目标文件，并有如下图所示的提示信息。

   [![image](https://i-blog.csdnimg.cn/blog_migrate/51191d25a676f414893a97f774a817b4.png "image")](http://images.cnblogs.com/cnblogs_com/we-hjb/201101/201101062100192667.png) 

      这种方法相对比较灵活，对合并文件的个数和偏移地址没有限制。缺点是不支持配置文件，不能保存所设的配置，所以，每次合并都得手动做很多重复工作。在调试阶段会比较浪费时间。

      下面介绍一种通过命令行工具合并两个文件的方法。该方法需要用到fsutil.exe、cat.exe和hbin.exe。写一个批处理文件，分别调用这三个工具，最终将1.bin和2.bin合并成dest.bin。批处理文件的内容如下:

del dest.bin  
fsutil  file createnew dest.bin  4096  cat  2 .bin  \>> dest.bin  
hbin  1 .bin dest.bin

       批处理文件各行的简单说明，

- del dest.bin，删除原来的目标文件
- fsutil  file createnew dest.bin 4096，创建一个大小为4096字节的空白文件dest.bin，该值的大小由偏移地址0x1000决定
- cat 2.bin >>dest.bin，将2.bin追加到空白文件dest.bin之后
- hbin 1.bin dest.bin，将1.bin放到dest.bin的头上，填充dest.bin头上4KB的空白

      dest.bin就是我们最终需要的合并完成的文件。将它与第一种方法合并的文件dst.bin对比一下，如下。

    [![image](https://i-blog.csdnimg.cn/blog_migrate/3ea9d8daaa865a814ce9ce11c81bc437.png "image")](http://images.cnblogs.com/cnblogs_com/we-hjb/201101/2011010621002059.png)

      可以看到两种方法合并出的文件，完全一样。

      第二种方法的好处在于，可以在集成开发环境中设置编译选项，在编译完成之后自动执行该批处理，这样，编译完成后即得到能够直接固化到Flash中的二进制文件，节省了一些时间。

     文中介绍的相关工具的下载地址:[http://files.cnblogs.com/we-hjb/HEBING.rar](http://files.cnblogs.com/we-hjb/HEBING.rar "http://files.cnblogs.com/we-hjb/HEBING.rar")