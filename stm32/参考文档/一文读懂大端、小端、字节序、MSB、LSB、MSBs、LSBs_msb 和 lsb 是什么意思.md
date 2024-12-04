> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/wangyx1234/article/details/130352079)

大端、小端、[字节序](https://so.csdn.net/so/search?q=%E5%AD%97%E8%8A%82%E5%BA%8F&spm=1001.2101.3001.7020)、MSB、LSB、MSBs、LSBs
------------------------------------------------------------------------------------------------------------------

5 分钟完全理解上述嵌入式、物联网开发中很扯蛋的几个被玩坏概念。

### [MSB](https://so.csdn.net/so/search?q=MSB&spm=1001.2101.3001.7020)、LSB?

#### 对于涉及 bit 流的概念中

MSB（Most Significant Bit）：最高有效位，二进制中代表最高值的比特位，这一位对数值的影响最大。  
[LSB](https://so.csdn.net/so/search?q=LSB&spm=1001.2101.3001.7020)（Least Significant Bit）：最低有效位，二进制中代表最低值的比特位。  
![[../../_resources/一文读懂大端、小端、字节序、MSB、LSB、MSBs、LSBs_msb 和 lsb 是什么意思/a26f7e74b4211d6d40517ee507201711_MD5.png]]

以字面值数字 9 为例，其二进制是 1001。

*   如果其 MSB 发生错误，即最高位的 1 发生错误变为了 0，则整个数字就变为了 1。误差为 8。
*   如果其 LSB 发生错误，即最低位的 1 发生错误变为了 0，则整个数字就变为了 8。误差为 1。

#### MSBs、LSBs

一个 8bit 的数据 10001111 中有两种 7bit 的数据：  
![[../../_resources/一文读懂大端、小端、字节序、MSB、LSB、MSBs、LSBs_msb 和 lsb 是什么意思/a14e220ded6c5140e933f6fa73f28276_MD5.png]]

*   7bit MSBs: 这种写法通常指的是高 7bit：1000111
*   7bit LSBs: 这种写法通常指的是低 7bit：0001111

#### 对于涉及 byte 流的概念中

MSB（Most Significant Byte）：多字节序列中最高权重的一个字节。  
LSB（Least Significant Byte）：多字节序列中最小权重的字节。  
![[../../_resources/一文读懂大端、小端、字节序、MSB、LSB、MSBs、LSBs_msb 和 lsb 是什么意思/7f3bd0978b7d5c40a75542827279cc7e_MD5.png]]

#### 如何确认 MSB、LSB 代表的具体含义?

具体如何区分 MSB（Most Significant Bit）与 MSB（Most Significant Byte）？  
看具体的使用场景，如果对象是一个与 bit 流相关的概念（比如数据传输领域，标准的串口传输方式是低位先行，芯片在通过 TX 引脚发送数据时，依次发送位 0、位 1、、、、、位 7。），则 MSB 此时是（Most Significant Bit）。  
如果是一个与 byte 相关的概念（如在 32 位机器上一个整型数据在内存上存储顺序），则是一个 （Most Significant Byte）. 主要看讨论的对象是一个 bit 流，还是存储、传输时的 byte 流。

### 什么是字节序？

字节序，指的是占用多个字节的数据在嵌入式设备的内存中或在网络通信链路中的字节排列顺序。  
字面值 0x12345678，其一共有 4 个字节，这四个字节按照字面值的 **低位 -> 高位** 的顺序分别是：0x78、0x56、0x34、0x12。  
它们在内存**存储**、网络**传输**中的顺序可能是：

*   第一种可能：0x12、0x34、0x56、0x78
*   第二种可能：0x78、0x56、0x34、0x12  
    给这两种可能起一个看起来更专业的名字，就有了大端字节序、小端字节序的名称，简称大端序（Big-Endian）、小端序（Little-Endian）。  
    一副图表示两种字节序：  
    ![[../../_resources/一文读懂大端、小端、字节序、MSB、LSB、MSBs、LSBs_msb 和 lsb 是什么意思/3f0b3fcc498cf81878cfbdbbd8a819fe_MD5.png]]
*   大端序（Big-Endian）将字面值的低位字节存放在内存的高位地址（或者在网络传输中发送端先发送高字节，接收端把接收到的第一个字节当作高位字节看待）。
*   小端序（Little-Endian），将字面值的低位放在较小的地址处（或者在网络传输中先传输低字节）。

### 为什么有字节序

计算机内存单元的特性，小端字节序通常**加载运行效率比较高**，比较喜欢小端序。网络传输中**先传输对数据影响最重要的部分**，自然是让接收端先接收高位字节（参考上面 Most Significant Byte 的介绍）。通常**网络字节序**就是大端字节序。

### 编写程序确认当前设备的大小端字节序。

嵌入式面试中总是有这样的问题，编写一个测试程序啊小伙伴：

```
bool check_endian()
{
	union test {
		uint8_t u8;
		int i32;
	}test;
	test.i32 = 1;
	return (test.u8 == 0);
}
int main()
{
	if (check_endian() == true)
		printf("big\n");
	else
		printf("little\n");

	return 0;
}
```

### 总结

1.  MSB 可能指 MSB（Most Significant Bit）或者 MSB（Most Significant Byte）。同理，LSB 可能指 LSB（Least Significant Bit）或者（Least Significant Byte）具体指的是 Bit 还是 Byte，需要结合上下文语境进行判定。
2.  MSBs: 这种写法通常指的是高位的几个 bit，LSBs: 这种写法通常指的是几位的几个 bit。
3.  字节序，指的是占用多个字节的数据在嵌入式设备的内存中或在网络通信链路中的字节排列顺序。字节序有大端、小端之分，网络字节序是大端字节序。  
    （谢谢点赞或收藏，助力工程师文化越来愈好）