---
title: "一文详解循环冗余校验校验算法（CRC校验）及C语言代码的实现 ---- 以CRC-16/MODBUS为例讲解一起养成写作 - 掘金"
source: "https://juejin.cn/post/7090567719329726472"
author:
published: 2022-04-25
created: 2025-03-31
description: "一起养成写作习惯！这是我参与「掘金日新计划 · 4 月更文挑战」的第23天，点击查看活动详情。 一、概述 现在的产品开发过程中，无论是数据的储存还是传输，都需要确保数据的准确性，所以就需要在数据帧后面"
tags:
  - "clippings"
---
一起养成写作习惯！这是我参与「掘金日新计划 · 4 月更文挑战」的第23天， [点击查看活动详情](https://juejin.cn/post/7080800226365145118 "https://juejin.cn/post/7080800226365145118") 。

## 一、概述

现在的产品开发过程中，无论是数据的储存还是传输，都需要确保数据的准确性，所以就需要在数据帧后面附加一串校验码，方便接收方使用校验码校验接收到的数据是否是正确的。

常用的校验方式有奇偶校验、异或校验、累加和校验（可以看我之前的一篇文章 [累加和校验算法（CheckSum算法）](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fm0_37697335%2Farticle%2Fdetails%2F83867199%3Fops_request_misc%3D%2525257B%25252522request%2525255Fid%25252522%2525253A%25252522161174854316780266252976%25252522%2525252C%25252522scm%25252522%2525253A%2525252220140713.130102334..%25252522%2525257D%26request_id%3D161174854316780266252976%26biz_id%3D0%26utm_medium%3Ddistribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-83867199.first_rank_v2_pc_rank_v29%26utm_term%3D%2525E7%2525B4%2525AF%2525E5%25258A%2525A0%2525E5%252592%25258C%2525E6%2525A0%2525A1%2525E9%2525AA%25258C "https://blog.csdn.net/m0_37697335/article/details/83867199?ops_request_misc=%25257B%252522request%25255Fid%252522%25253A%252522161174854316780266252976%252522%25252C%252522scm%252522%25253A%25252220140713.130102334..%252522%25257D&request_id=161174854316780266252976&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-83867199.first_rank_v2_pc_rank_v29&utm_term=%25E7%25B4%25AF%25E5%258A%25A0%25E5%2592%258C%25E6%25A0%25A1%25E9%25AA%258C") ）、循环冗余校验（CRC校验）等等。

奇偶校验、异或校验、累加和校验都比较简单，且易于实现，但是检错能力也一般，而本文的主角CRC校验，则大大提高了数据的检错能力。（对比文章看这篇 [《奇偶校验 累加和校验 CRC校验》](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fzhang1308299607%2Farticle%2Fdetails%2F74854840 "https://blog.csdn.net/zhang1308299607/article/details/74854840") ）

讲CRC校验的文章很多，也有非常多的好文章，本文只是将这些文章总结，用自己的理解再阐述一遍，再次感谢各位大佬优质的文章。

CRC校验有很多的参数模型，尤其是有很多标准模型的存在，这些标准模型经过理论计算，大大提高校验检错的能力，所以推荐使用标准模型，由于有大量的教程和网上资源，参考后实现起来也比较简单，同时也利于不同部门间的沟通和定下标准。 本文就是采用CRC-16/MODBUS参数模型，该模型就被各行各业大量使用

## 二、CRC的基本原理

网上有看到一篇讲解CRC原理的文章非常不错的文章： [循环冗余检验 (CRC) 算法原理](https://link.juejin.cn/?target=https%3A%2F%2Fwww.cnblogs.com%2Fesestt%2Farchive%2F2007%2F08%2F09%2F848856.html "https://www.cnblogs.com/esestt/archive/2007/08/09/848856.html") 。 文章讲解了CRC校验的基本原理，讲的很清楚，尤其是查表法的原理，需要静下来心来看几遍才能理解透彻。 如果还不懂，还可以看看这篇： [最通俗的CRC校验原理剖析](https://link.juejin.cn/?target=https%3A%2F%2Fblog.51cto.com%2Fwinda%2F1063951 "https://blog.51cto.com/winda/1063951") 。

CRC的校验的结果不仅由选择的多项式决定，还包括其他各种参数，下面就是需要注意的参数：（来自CRC在线计算工具 [www.ip33.com/crc.html](https://link.juejin.cn/?target=http%3A%2F%2Fwww.ip33.com%2Fcrc.html "http://www.ip33.com/crc.html") ） ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c704b41d44734d688e37244a6d508287~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 这篇文章里面有详细地介绍了这些参数的讲解： [CRC校验原理及其C语言实现](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fwhik1194%2Farticle%2Fdetails%2F108837493 "https://blog.csdn.net/whik1194/article/details/108837493") 。

## 三、CRC-16/MODBUS的介绍以及实现

modbus是美国的modicon公司开发的一种报文传输协议，1979年该公司成为施耐德公司的一部分。modbus协议在工业控制中得到了广泛的应用，它已经成为一种通用的工业标准，该协议支持rs-232、rs-422、rs-485和以太网设备。不同厂商生产的控制设备通过modbus协议可以连成通信网络，进行集中监控。许多工控产品，例如plc、变频器、人机界面、dcs和自动化仪表等，都在广泛地使用modbus协议。

而CRC-16/MODBUS作为modbus的数据校验方式，正成为比较通用的CRC检验的参数模型。

**CRCCRC-16/MODBUS** C语言实现代码可以看这一篇文章： [【CRC笔记】CRC-16 MODBUS C语言实现】](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fu012028275%2Farticle%2Fdetails%2F112066723 "https://blog.csdn.net/u012028275/article/details/112066723") 。

CRC-16/MODBUS C语言的实现一般有直接计算法和查表法，直接计算法省存储耗时间，而查表法使用空间区换时间，校验过程耗时少但占用存储空间大。在平时的产品开发过程中，我们更看重时间，且查表法占用的存储一般是我们可以接受的，所以查表法的使用更加普遍。

## 3.1、直接计算法的实现

计算步骤为(摘抄自 [www.cnblogs.com/jungle1989/…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.cnblogs.com%2Fjungle1989%2Fp%2F6372527.html "https://www.cnblogs.com/jungle1989/p/6372527.html") )

> (1).预置 16 位寄存器为十六进制 FFFF（即全为 1） ，称此寄存器为 CRC 寄存器；  
> (2).把第一个 8 位数据与 16 位 CRC 寄存器的低位相异或，把结果放于 CRC 寄存器； (3).检测相异或后的CRC寄存器的最低位， 若最低位为1：CRC寄存器先右移1位，再与多项式A001H进行异或； 若为0，则CRC寄存器右移1位，无需与多项式进行异或。 (4).重复步骤 3 ，直到右移 8 次，这样整个 8 位数据全部进行了处理； (5).重复步骤 2 到步骤4，进行下一个 8 位数据的处理； (6).最后得到的 CRC 寄存器即为 CRC 码。

根据上面的步骤，得到其C语言的代码实现为（摘抄自 [blog.csdn.net/u012028275/…](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fu012028275%2Farticle%2Fdetails%2F112066723%25EF%25BC%258C%25E4%25B8%25AA%25E4%25BA%25BA%25E6%25B7%25BB%25E5%258A%25A0%25E6%25B3%25A8%25E9%2587%258A%25EF%25BC%2589 "https://blog.csdn.net/u012028275/article/details/112066723%EF%BC%8C%E4%B8%AA%E4%BA%BA%E6%B7%BB%E5%8A%A0%E6%B3%A8%E9%87%8A%EF%BC%89")

```c
//直接计算法计算crc
unsigned short do_crc(unsigned char *ptr, int len)
{
    unsigned int i;
    unsigned short crc = 0xFFFF;  //crc16位寄存器初始值

    while(len--)
    {
        crc ^= *ptr++;
        for (i = 0; i < 8; ++i)
        {
            if (crc & 1)
                crc = (crc >> 1) ^ 0xA001; //多项式 POLY（0x8005)的高低位交换值，这是由于其模型的一些参数决定的
            else
                crc = (crc >> 1);
        }
    }

    return crc;
}
```

## 3.2、查表法实现

查表法确实是一种比较优秀的方法。实在佩服那些推导出来的大佬们，其原理的实现可以看这篇文章 [《循环冗余检验 (CRC) 算法原理》](https://link.juejin.cn/?target=https%3A%2F%2Fwww.cnblogs.com%2Fesestt%2Farchive%2F2007%2F08%2F09%2F848856.html "https://www.cnblogs.com/esestt/archive/2007/08/09/848856.html") 。 其实现其实是直接计算法的优化版，将一个字节的校验（8个位的校验）用查表一个操作来实现，大大减少了其运行时间。 其C语言的代码实现如下所示（摘抄自 [blog.csdn.net/u012028275/…](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fu012028275%2Farticle%2Fdetails%2F112066723%25EF%25BC%258C%25E4%25B8%25AA%25E4%25BA%25BA%25E6%25B7%25BB%25E5%258A%25A0%25E6%25B3%25A8%25E9%2587%258A%25EF%25BC%2589 "https://blog.csdn.net/u012028275/article/details/112066723%EF%BC%8C%E4%B8%AA%E4%BA%BA%E6%B7%BB%E5%8A%A0%E6%B3%A8%E9%87%8A%EF%BC%89")

```c
/* CRC余式表 */
const unsigned int crc_table[256] = {
    0x0000, 0xc0c1, 0xc181, 0x0140, 0xc301, 0x03c0, 0x0280, 0xc241,
    0xc601, 0x06c0, 0x0780, 0xc741, 0x0500, 0xc5c1, 0xc481, 0x0440,
    0xcc01, 0x0cc0, 0x0d80, 0xcd41, 0x0f00, 0xcfc1, 0xce81, 0x0e40,
    0x0a00, 0xcac1, 0xcb81, 0x0b40, 0xc901, 0x09c0, 0x0880, 0xc841,
    0xd801, 0x18c0, 0x1980, 0xd941, 0x1b00, 0xdbc1, 0xda81, 0x1a40,
    0x1e00, 0xdec1, 0xdf81, 0x1f40, 0xdd01, 0x1dc0, 0x1c80, 0xdc41,
    0x1400, 0xd4c1, 0xd581, 0x1540, 0xd701, 0x17c0, 0x1680, 0xd641,
    0xd201, 0x12c0, 0x1380, 0xd341, 0x1100, 0xd1c1, 0xd081, 0x1040,
    0xf001, 0x30c0, 0x3180, 0xf141, 0x3300, 0xf3c1, 0xf281, 0x3240,
    0x3600, 0xf6c1, 0xf781, 0x3740, 0xf501, 0x35c0, 0x3480, 0xf441,
    0x3c00, 0xfcc1, 0xfd81, 0x3d40, 0xff01, 0x3fc0, 0x3e80, 0xfe41,
    0xfa01, 0x3ac0, 0x3b80, 0xfb41, 0x3900, 0xf9c1, 0xf881, 0x3840,
    0x2800, 0xe8c1, 0xe981, 0x2940, 0xeb01, 0x2bc0, 0x2a80, 0xea41,
    0xee01, 0x2ec0, 0x2f80, 0xef41, 0x2d00, 0xedc1, 0xec81, 0x2c40,
    0xe401, 0x24c0, 0x2580, 0xe541, 0x2700, 0xe7c1, 0xe681, 0x2640,
    0x2200, 0xe2c1, 0xe381, 0x2340, 0xe101, 0x21c0, 0x2080, 0xe041,
    0xa001, 0x60c0, 0x6180, 0xa141, 0x6300, 0xa3c1, 0xa281, 0x6240,
    0x6600, 0xa6c1, 0xa781, 0x6740, 0xa501, 0x65c0, 0x6480, 0xa441,
    0x6c00, 0xacc1, 0xad81, 0x6d40, 0xaf01, 0x6fc0, 0x6e80, 0xae41,
    0xaa01, 0x6ac0, 0x6b80, 0xab41, 0x6900, 0xa9c1, 0xa881, 0x6840,
    0x7800, 0xb8c1, 0xb981, 0x7940, 0xbb01, 0x7bc0, 0x7a80, 0xba41,
    0xbe01, 0x7ec0, 0x7f80, 0xbf41, 0x7d00, 0xbdc1, 0xbc81, 0x7c40,
    0xb401, 0x74c0, 0x7580, 0xb541, 0x7700, 0xb7c1, 0xb681, 0x7640,
    0x7200, 0xb2c1, 0xb381, 0x7340, 0xb101, 0x71c0, 0x7080, 0xb041,
    0x5000, 0x90c1, 0x9181, 0x5140, 0x9301, 0x53c0, 0x5280, 0x9241,
    0x9601, 0x56c0, 0x5780, 0x9741, 0x5500, 0x95c1, 0x9481, 0x5440,
    0x9c01, 0x5cc0, 0x5d80, 0x9d41, 0x5f00, 0x9fc1, 0x9e81, 0x5e40,
    0x5a00, 0x9ac1, 0x9b81, 0x5b40, 0x9901, 0x59c0, 0x5880, 0x9841,
    0x8801, 0x48c0, 0x4980, 0x8941, 0x4b00, 0x8bc1, 0x8a81, 0x4a40,
    0x4e00, 0x8ec1, 0x8f81, 0x4f40, 0x8d01, 0x4dc0, 0x4c80, 0x8c41,
    0x4400, 0x84c1, 0x8581, 0x4540, 0x8701, 0x47c0, 0x4680, 0x8641,
    0x8201, 0x42c0, 0x4380, 0x8341, 0x4100, 0x81c1, 0x8081, 0x4040,
};

//查表法计算crc
unsigned short do_crc_table(unsigned char *ptr, int len)
{
    unsigned short crc = 0xFFFF;
    
    while(len--) 
    {
        crc = (crc >> 8) ^ crc_table[(crc ^ *ptr++) & 0xff]; //原理参考直接计算法
    }
    
    return (crc);
}
```

## 3.3、CRC 余式表的获取（C语言代码输出）

一定有人好奇，上述的余式表是如何获取的呢？下面就使用C语言来输出这个表格。 （注：本节的内容针对CRCCRC-16/MODBUS，别的参数模型需要根据具体协议修改）

- 先获取一个单个字节在CRC 余式表对应的数据：（代码是不是很熟悉，其实就是直接计算法计算单个字节的那一部分代码）
	```c
	uint16_t byte_crc_get(uint8_t value)
	{
	    uint8_t i;
	    uint16_t crc = value;
	    /* 数据往右移了8位，需要计算8次 */
	    for (i=8; i>0; --i)
	    { 
	        if (crc & 1)  /* 判断最高位是否为1 */
	        {
	            crc = (crc >> 1) ^ 0xA001;    
	        }
	        else
	        {
	            crc = (crc >> 1);
	        }
	    }
	    return crc;
	}
	```
- 获取所有字节数据对应的数据（一共256个），组成表格
	```c
	void  create_crc_table(void)
	{
	    uint16_t i;
	    uint16_t j;
	    for (i=0; i<=0xFF; i++)
	    {
	        if (0 == (i%16))
	            printf("\n");
	        j = i&0xFF;
	        printf("0x%.4x, ", byte_crc_get (j));  /*依次计算每个字节的crc校验值*/
	    }
	}
	```
- 最后在main函数中调用 `create_crc_table()` 即可输出余式表，之后复制即可使用。 ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19e699f1e6f640669eb0e94efd098aea~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

## 四、参考文章

- [循环冗余检验 (CRC) 算法原理](https://link.juejin.cn/?target=https%3A%2F%2Fwww.cnblogs.com%2Fesestt%2Farchive%2F2007%2F08%2F09%2F848856.html "https://www.cnblogs.com/esestt/archive/2007/08/09/848856.html")
- [最通俗的CRC校验原理剖析](https://link.juejin.cn/?target=https%3A%2F%2Fblog.51cto.com%2Fwinda%2F1063951 "https://blog.51cto.com/winda/1063951")
- [CRC校验原理及其C语言实现](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fwhik1194%2Farticle%2Fdetails%2F108837493 "https://blog.csdn.net/whik1194/article/details/108837493")
- [最详细易懂的CRC-16校验原理（附源程序）](https://link.juejin.cn/?target=https%3A%2F%2Fwww.cnblogs.com%2F94cool%2Fp%2F3559585.html "https://www.cnblogs.com/94cool/p/3559585.html")
- [CRC（循环冗余校验在线计算](https://link.juejin.cn/?target=http%3A%2F%2Fwww.ip33.com%2Fcrc.html "http://www.ip33.com/crc.html")
- [【CRC笔记】CRC-16 MODBUS C语言实现](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fu012028275%2Farticle%2Fdetails%2F112066723 "https://blog.csdn.net/u012028275/article/details/112066723")