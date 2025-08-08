---
title: "stm32使用半满中断实现的高可靠串口数据收发_hal uart 开启半满中断-CSDN博客"
source: "https://blog.csdn.net/a1598025967/article/details/120539170"
author:
published:
created: 2025-04-23
description: "文章浏览阅读1w次，点赞30次，收藏111次。本文介绍了如何针对STM32串口通信在接收数据时出现丢包的问题进行优化。最初采用DMA单字节接收，后来通过启用空闲中断和半满中断，实现更稳定的不定长数据帧接收。最终方案结合了DMA、空闲中断和半满中断，确保了在突发数据量大的情况下也能有效避免丢包。"
tags:
  - "clippings"
---
本文介绍了如何针对STM32串口通信在接收数据时出现丢包的问题进行优化。最初采用DMA单字节接收，后来通过启用空闲中断和半满中断，实现更稳定的不定长数据帧接收。最终方案结合了DMA、空闲中断和半满中断，确保了在突发数据量大的情况下也能有效避免丢包。

摘要生成于 [C知道](https://ai.csdn.net/?utm_source=cknow_pc_ai_abstract) ，由 DeepSeek-R1 满血版支持， [前往体验 >](https://ai.csdn.net/?utm_source=cknow_pc_ai_abstract)

## 0.1 写在前面

串口在各种项目中可谓是太常用了，它也是搞 [嵌入式](https://so.csdn.net/so/search?q=%E5%B5%8C%E5%85%A5%E5%BC%8F&spm=1001.2101.3001.7020) 必须弄懂的一个通信协议，最近维护了很久的一个项目，设备内另一模块程序更新后出现了不稳定的情况，现象就是某个功能有时候正常有时候不正常，经排查是通信接口上出现了丢包导致的，通信的接口正是用的串口，然后经过多次优化，解决了问题，以此记录一下优化过程。

## 0.2 软硬件环境

软件 ：MDK5、 stm32 [HAL库](https://so.csdn.net/so/search?q=HAL%E5%BA%93&spm=1001.2101.3001.7020)

硬件：项目上主控芯片为 `stm32f407zet6` (`调试时使用的stm32f103c8t6`)，整板外设只用了5个串口，2个硬件定时器。

## 0.3 库函数接口

首先看一下用到的库函数接口，不重要的忽略：

| \_\_HAL\_DMA\_GET\_COUNTER | 获取DMA剩余未接收数据 |
| --- | --- |
| HAL\_UART\_Transmit | 串口阻塞方式发送函数 |
| HAL\_UART\_Transmit\_IT | 串口中断方式发送函数 |
| HAL\_UART\_Receive\_IT | 串口中断方式接收数据 |
| HAL\_UART\_Transmit\_DMA | 串口DMA方式发送函数 |
| HAL\_UART\_Receive\_DMA | 串口DMA方式接收函数 |
| HAL\_UART\_TxCpltCallback | 串口发送完成回调函数 |
| HAL\_UART\_RxCpltCallback | 串口接收完成回调函数 |
| HAL\_UART\_RxHalfCpltCallback | 串口接收过半回调函数 |

## 0.4 初始实现方式

由于项目中是自定义帧格式，而且每个帧长很短，不超过16字节，所以最开始串口接收使用的是DMA单字节接收，当检测接收到一个完整帧时，将收到的帧写入fifo，然后发送一个信号量，被阻塞的任务得到信号量后从fifo读取帧并作相应操作，大致流程如下：  
![流程图](https://i-blog.csdnimg.cn/blog_migrate/0ca199391c724887869cfea8f7d6bb1a.png#pic_center)  
这样的实现方式比较简单，在数据速率比较恒定的情况下是没有问题的，但最近与之通信的模块程序更新后，出现了偶尔突发数据量会很大的情况，这样就可能会丢失数据。

## 0.5 第一次优化

知道了问题所在后，进行第一次优化，经过分析有以下方案可以选用：

1. DMA(或中断)一次接收多字节
2. DMA加空闲中断

因为最终选用的第二种方式，所以说说为什么第一种方式不行，原因有以下几点：

- 对于 `中断` 方式来说一次接收多字节并未解决频繁中断的问题，还是会一个字节产生一次中断
- 单纯的 `DMA(或中断)` 必须要接收到指定数量的数据才能完全读走数据，否则数据会一直被缓存无法读取
- 通信数据帧是不定长的

空闲中断会在收到一个字节后指定时间内未收到下一个字节时产生，这样的话就可以在产生空闲中断时将收到的数据读走，而不会一直被缓存着，实现代码如下(`在使能DMA接收的前提下`)：

```c
/* 初始化时使能空闲中断 */
__HAL_UART_ENABLE_IT(&UartHandle, UART_IT_IDLE);
12
```
```c
/* 串口中断处理函数中增加对空闲中断的处理 */
void USART_IRQHandler(void)
{
    uint32_t tmp = 0;

    if(__HAL_UART_GET_FLAG(&UartHandle, UART_FLAG_IDLE))
    {
        /* 清空闲中断标志位 */
        __HAL_UART_CLEAR_IDLEFLAG(&UartHandle);
        /* 停止DMA接收 */
        HAL_UART_DMAStop(&UartHandle);
        /* 得到已接收数据长度 */
        tmp = UartHandle.RxXferSize - __HAL_DMA_GET_COUNTER(UartHandle.hdmarx);

        if(0 != tmp)
        {
            /* 存入数据到fifo */
        }
        /* 再次开启DMA接收 */
        HAL_UART_Receive_DMA(&UartHandle, UartHandle.pRxBuffPtr, UartHandle.RxXferSize);
    }

    HAL_UART_IRQHandler(&UartHandle);
}
123456789101112131415161718192021222324
```

理论上来说这样的话只要接收缓冲足够大，写入fifo的操作只会发生在产生空闲中断时，应该会大大缓解丢包的情况，但实际测试效果却不明显，并没有很好的处理突发数据的接收。  
分析原因应该是由于DMA接收是不受控的，在处理空闲中断时短暂关闭了DMA的接收，而就是在这个关闭的过程中如果有新数据到来，则只能丢弃(`并且会丢弃前面已接收的一部分数据`)，从而产生丢包。

## 0.6 第二次优化

既然知道了是因为短暂关闭DMA导致的，那就一步到位，想办法不关闭DMA就能解决了，stm32的DMA除了满中断还有个 `半满中断` ，也就是接收数据过半时产生中断，那就可以在接收数据过半时将已收到的前半数据写入fifo，然后产生满中断时将后半数据写入fifo，此时DMA会自动将写指针移动到接收缓存的头部继续接收，循环这个过程，就不必关闭DMA，大致流程如下：  
![流程图](https://i-blog.csdnimg.cn/blog_migrate/56b45d991fddc1827bba29735a0b8445.png#pic_center)  
按照如上的流程进行优化后，的确没发现丢包的情况了，这种和 `stm32f4` 支持的多缓冲原理类似，但是多缓冲的话在 `stm32f103c8t6` 上面没有这个功能，我是在 `stm32f103c8t6` 上面验证的，所以没有使用多缓冲。  
就在我以为这样就结束了时，又发现了新的问题，前面说过DMA要收到指定数量的数据时才会产生中断，那这里还漏了一种情况，那就是如果发送方发送过来的帧长不足以让DMA产生中断，那数据就会被缓存，直到满足条件才能读取，这样的话肯定不行，所以就得 `空闲中断` 上场了。

## 0.7 最后的修改

现在只需要将空闲中断加入上面那个流程，就能够应对各种情况了，因为是不定长帧，所以空闲中断会在接收的任意期间产生，我用一个全局变量 `head_ptr` 来保存缓冲区的 `读起始偏移` ， `tail_ptr` 来保存缓冲区的 `读结束偏移` (`注意` ： `tail_ptr` 是虚拟的，它的值有三种情况，后面会讲到)，这样实现一个类循环fifo结构。

1. 初始状态， `head_ptr` 和 `tail_ptr` 都指向缓冲区首部，收到一帧数据后，没有新数据到来，触发空闲中断，在空闲中断回调函数中要做的操作就是读走这部分数据(`图中1号区域` ， `head_ptr` 与 `tail_ptr` 之间的数据)，并且将 `head_ptr` 移动到 `tail_ptr` 的位置，模型如下：(**只要产生空闲中断，都适用此流程**)  
	![流程1](https://i-blog.csdnimg.cn/blog_migrate/d17ced1d2013cdb95984ee77663d3a65.png#pic_center)
```c
head_ptr = 上次tail_ptr的位置;
/* huart->RxXferSize为接收缓存的总大小，__HAL_DMA_GET_COUNTER获取的是还未接收的数据大小 */
tail_ptr = huart->RxXferSize - __HAL_DMA_GET_COUNTER(huart->hdmarx);
123
```
1. 继续接收新数据，此为触发半满中断的情况，在半满中断回调函数中的操作是将 `head_ptr` 到 `tail_ptr` 之间的数据写入fifo(`图中2号区域`)，然后移动 `head_ptr` 到 `tail_ptr` 的位置，模型如下：  
	![流程2](https://i-blog.csdnimg.cn/blog_migrate/cfb4cb50ead80b29788a847a27c410ed.png#pic_center)
```c
head_ptr = 上次tail_ptr的位置;
/* huart->RxXferSize为接收缓存的总大小，(huart->RxXferSize & 1)奇数时为1，偶数时为0 */
tail_ptr = (huart->RxXferSize >> 1) + (huart->RxXferSize & 1);
123
```
1. 继续接收数据直到产生满中断，在满中断回调函数中的操作也是将 `head_ptr` 到 `tail_ptr` 之间的数据写入fifo(`图中3号区域`)，然后移动 `head_ptr` 到 `tail_ptr` 的位置，模型如下：  
	![流程3](https://i-blog.csdnimg.cn/blog_migrate/a6d047856234f2ff4e6bfb797848c732.png#pic_center)
```c
head_ptr = 上次tail_ptr的位置;
/* huart->RxXferSize为接收缓存的总大小 */
tail_ptr = huart->RxXferSize;
123
```

至此各种情况就都考虑完整了，一个相对可靠的串口接收程序就实现了。

## 0.8 收发数据模型

此文源码我放在了我的码云仓库上，有需要的可以自行下载([https://gitee.com/wei513723/stm32-stable-uart-transmit-receive](https://gitee.com/wei513723/stm32-stable-uart-transmit-receive))，源码中可以通过宏进行选择使用 `中断接收` 、 `DMA接收` 、 `DMA加空闲中断` 接收三种方式，使用的程序收发数据模型如下：  
![中断收模型](https://i-blog.csdnimg.cn/blog_migrate/c398faa6e5cfebe856e3ccb5c4878a85.png#pic_center)  
![DMA收模型](https://i-blog.csdnimg.cn/blog_migrate/bb4634f0a52a99fce2505fd2db164ca9.png#pic_center)  
![DMA加空闲中断模型](https://i-blog.csdnimg.cn/blog_migrate/cb030ac11a4b841c414d54b959c804e5.png#pic_center)  
![发送模型](https://i-blog.csdnimg.cn/blog_migrate/0e64066078e83da732d1f0d9fa3ee7e4.png#pic_center)

## 0.9 结尾

关于源码中这几个宏的配置须知：

```c
/*是否使能DMA接收*/
#define UART_USE_DMA_RX 1
/*是否使能DMA发送*/
#define UART_USE_DMA_TX 1

#if UART_USE_DMA_RX
    /*是否使能空闲中断*/
    #define UART_USE_IDLE_IT 1
#endif

/*配置接收缓冲区的大小*/
#define UART_BUF_SIZE 64
123456789101112
```

推荐： **DMA+空闲中断方式**

- 中断方式：缺点是中断频繁，每收到一个字节都会产生一次中断；必须接收到指定长度数据才能读走数据；适合定长数据帧；用不了DMA才推荐此种方式
- DMA方式：缺点是必须接收到指定长度数据才能读走数据；适合定长数据帧
- DMA加空闲中断：最优解

接收缓冲区大小根据自己需求而定，波特率越高，接收缓冲区大小相对的也应更大，接收fifo和发送fifo的大小也就越大。

**欢迎扫码关注我的微信公众号**

实付 元

[使用余额支付](https://blog.csdn.net/a1598025967/article/details/)

点击重新获取

扫码支付

钱包余额 0

抵扣说明：

1.余额是钱包充值的虚拟货币，按照1:1的比例进行支付金额的抵扣。  
2.余额无法直接购买下载，可以购买VIP、付费专栏及课程。

[余额充值](https://i.csdn.net/#/wallet/balance/recharge)

举报

 [![](https://csdnimg.cn/release/blogv2/dist/pc/img/toolbar/Group.png) 点击体验  
DeepSeekR1满血版](https://ai.csdn.net/?utm_source=cknow_pc_blogdetail&spm=1001.2101.3001.10583) 隐藏侧栏 ![程序员都在用的中文IT技术交流社区](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_app.png)

程序员都在用的中文IT技术交流社区

![专业的中文 IT 技术社区，与千万技术人共成长](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_wechat.png)

专业的中文 IT 技术社区，与千万技术人共成长

![关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_video.png)

关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！

客服 返回顶部

![](https://i-blog.csdnimg.cn/blog_migrate/0ca199391c724887869cfea8f7d6bb1a.png#pic_center) ![](https://i-blog.csdnimg.cn/blog_migrate/56b45d991fddc1827bba29735a0b8445.png#pic_center) ![](https://i-blog.csdnimg.cn/blog_migrate/d17ced1d2013cdb95984ee77663d3a65.png#pic_center) ![](https://i-blog.csdnimg.cn/blog_migrate/cfb4cb50ead80b29788a847a27c410ed.png#pic_center) ![](https://i-blog.csdnimg.cn/blog_migrate/a6d047856234f2ff4e6bfb797848c732.png#pic_center) ![](https://i-blog.csdnimg.cn/blog_migrate/c398faa6e5cfebe856e3ccb5c4878a85.png#pic_center) ![](https://i-blog.csdnimg.cn/blog_migrate/bb4634f0a52a99fce2505fd2db164ca9.png#pic_center) ![](https://i-blog.csdnimg.cn/blog_migrate/cb030ac11a4b841c414d54b959c804e5.png#pic_center) ![](https://i-blog.csdnimg.cn/blog_migrate/0e64066078e83da732d1f0d9fa3ee7e4.png#pic_center)