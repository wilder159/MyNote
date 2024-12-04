> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/william_munch/article/details/84347788)

#### 0.1.1.1 **1. 硬盘 (可以认为硬盘就是磁盘)**

#### 0.1.1.2 ![[../../_resources/扇区 (sector)，块 (block)，簇 (cluster)_u 盘上的 cluster 是什么意思/ea73e62a9512a88fec69e5acfb7d004a_MD5.png]]

# 1 fdisk -l  
Disk /dev/cciss/c0d0: 146.7 GB, 146778685440 bytes  
255 heads, 63 sectors/track, 17844 cylinders  
Units = cylinders of 16065 * 512 = 8225280 bytes  
可以看到几个名词：heads/sectors/cylinders，分别就是磁头 / 扇区 / 柱面，每个扇区 512byte（现在新的硬盘每个扇区有 4K）了  
硬盘容量就是 heads*sectors*cylinders*512=255*63*17844*512=146771896320b=146.7G  
**_注意：硬盘的最小存储单位就是扇区了，而且硬盘本身并没有 block 和 cluster 的概念。_**

**磁头 和 柱面**

硬盘通常由重叠的一组盘片构成，每个盘面都被划分为数目相等的磁道，并从外缘的 “0” 开始编号，具有相同编号的磁道形成一个圆柱，称之为磁盘的柱面。磁盘的柱面数与一个盘面上的磁道数是相等的。由于每个盘面都有自己的磁头，因此，盘面数等于总的磁头数。 如下图

![[../../_resources/扇区 (sector)，块 (block)，簇 (cluster)_u 盘上的 cluster 是什么意思/da838dc98aaaa16b17210965b9b112d6_MD5.png]]

#### 1.1.1.1 块和簇

由于扇区的空间比较小且数目众多，在寻址时比较困难，所以操作系统就将多个的扇区组合在一起，形成一个更大的单位，再对这个单位进行整体的操作。这个单位，在 Windows 下，FAT，[FAT32](https://www.baidu.com/s?wd=FAT32&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao) 和 NTFS 文件系统中叫做簇（cluster）；在 Linux 下如 Ext4 等文件系统中叫做块（block）。每个簇或者块可以包括 2、4、8、16、32、64…2 的 n 次方个扇区。

#### 1.1.1.2 总结

**磁盘读写基本单位是扇区**。
**操作系统是通过块和簇来做为单位读取等操作数据的**。
文件系统就是操作系统的一部分，所以文件系统操作文件的最小单位是块和簇。
**磁盘控制器**，其作用除了读取数据、控制磁头等作用外，还有的功能就是**映射扇区和磁盘块的关系**。
  
**综上，扇区是对硬盘而言，是物理层的，块和簇是对文件系统而言，是逻辑层的。磁盘控制器是用来映射两层的。**

**这个问题我想了好久，查了好多资料。如有错误，欢迎指正！**