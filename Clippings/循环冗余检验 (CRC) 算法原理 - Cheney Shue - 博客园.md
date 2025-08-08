---
title: "循环冗余检验 (CRC) 算法原理 - Cheney Shue - 博客园"
source: "https://www.cnblogs.com/esestt/archive/2007/08/09/848856.html"
author:
published:
created: 2025-03-31
description:
tags:
  - "clippings"
---
[![](https://img2024.cnblogs.com/blog/35695/202412/35695-20241201073014811-1847930772.jpg)](https://www.doubao.com/?channel=cnblogs&source=hw_db_cnblogs&type=lunt&theme=bianc)

## 0.1 导航

统计
- 随笔 - 77
- 文章 - 0
- 评论 - 575
- 阅读 - 35万

最近工作中需要用到数据传输校验，写一些对CRC的理解和算法思路。

Cyclic R edundancy C heck 循环冗余检验，是基于数据计算一组效验码，用于核对数据传输过程中是否被更改或传输错误。

## 0.2 算法原理

假设数据传输过程中需要发送15位的二进制信息g=101001110100001，这串二进制码可表示为代数多项式g(x) = x^14 + x^12 + x^9 + x^8 + x^7 + x^5 + 1，其中g中第k位的值，对应g(x)中x^k的系数。将g(x)乘以x^m，既将g后加m个0，然后除以m阶多项式h(x)，得到的(m-1)阶余项r(x)对应的二进制码r就是CRC编码。

h(x)可以自由选择或者使用国际通行标准，一般按照h(x)的阶数m，将CRC算法称为CRC-m，比如CRC-32、CRC-64等。国际通行标准可以参看 [http://en.wikipedia.org/wiki/Cyclic\_redundancy\_check](http://en.wikipedia.org/wiki/Cyclic_redundancy_check)

g(x)和h(x)的除运算，可以通过g和h做xor（异或）运算。比如将11001与10101做xor运算：

![](https://www.cnblogs.com/images/cnblogs_com/esestt/CRC/crc001.gif)

明白了xor运算法则后，举一个例子使用CRC-8算法求101001110100001的效验码。CRC-8标准的h(x) = x^8 + x^7 + x^6 + x^4 + x^2 + 1，既h是9位的二进制串111010101。  
  
![](https://www.cnblogs.com/images/cnblogs_com/esestt/CRC/crc002.gif)

经过迭代运算后，最终得到的r是10001100，这就是CRC效验码。

通过示例，可以发现一些规律，依据这些规律调整算法：  
  
1\. 每次迭代，根据gk的首位决定b，b是与gk进行运算的二进制码。若gk的首位是1，则b=h；若gk的首位是0，则b=0，或者跳过此次迭代，上面的例子中就是碰到0后直接跳到后面的非零位。

![](https://www.cnblogs.com/images/cnblogs_com/esestt/CRC/crc003.gif)  
2\. 每次迭代，gk的首位将会被移出，所以只需考虑第2位后计算即可。这样就可以舍弃h的首位，将b取h的后m位。比如CRC-8的h是111010101，b只需是11010101。  
  
![](https://www.cnblogs.com/images/cnblogs_com/esestt/CRC/crc004.gif)  
  
3\. 每次迭代，受到影响的是gk的前m位，所以构建一个m位的寄存器S，此寄存器储存gk的前m位。每次迭代计算前先将S的首位抛弃，将寄存器左移一位，同时将g的后一位加入寄存器。若使用此种方法，计算步骤如下：  
  
![](https://www.cnblogs.com/images/cnblogs_com/esestt/CRC/crc005.gif)

※蓝色表示寄存器S的首位，是需要移出的，b根据S的首位选择0或者h。黄色是需要移入寄存器的位。S'是经过位移后的S。

## 0.3 查表法

同样是上面的那个例子，将数据按每4位组成1个block，这样g就被分成6个block。

![](https://www.cnblogs.com/images/cnblogs_com/esestt/CRC/crc006.gif)

下面的表展示了4次迭代计算步骤，灰色背景的位是保存在寄存器中的。  
  
![](https://www.cnblogs.com/images/cnblogs_com/esestt/CRC/crc007.gif)  
  

经4次迭代，B1被移出寄存器。被移出的部分，不我们关心的，我们关心的是这4次迭代对B2和B3产生了什么影响。注意表中红色的部分，先作如下定义：  
  
B23 = 00111010  
b1 = 00000000  
b2 = 01010100  
b3 = 10101010  
b4 = 11010101  
b' = b1 xor b2 xor b3 xor b4  

4次迭代对B2和B3来说,实际上就是让它们与b1,b2,b3,b4做了xor计算，既：

B23 xor b1 xor b2 xor b3 xor b4

可以证明xor运算满足交换律和结合律，于是：

B23 xor b1 xor b2 xor b3 xor b4 = B23 xor (b1 xor b2 xor b3 xor b4) = B23 xor b'

b1是由B1的第1位决定的，b2是由B1迭代1次后的第2位决定（既是由B1的第1和第2位决定），同理,b3和b4都是由B1决定。通过B1就可以计算出b'。另外，B1由4位组成，其一共2^4有种可能值。于是我们就可以想到一种更快捷的算法，事先将b'所有可能的值，16个值可以看成一个表；这样就可以不必进行那4次迭代，而是用B1查表得到b'值，将B1移出，B3移入，与b'计算，然后是下一次迭代。  
  
![](https://www.cnblogs.com/images/cnblogs_com/esestt/CRC/crc008.gif)

可看到每次迭代，寄存器中的数据以4位为单位移入和移出，关键是通过寄存器前4位查表获得  
，这样的算法可以大大提高运算速度。

上面的方法是半字节查表法，另外还有单字节和双字节查表法，原理都是一样的——事先计算出2^8或2^16个b'的可能值，迭代中使用寄存器前8位或16位查表获得b'。  
  

## 0.4 反向算法

之前讨论的算法可以称为正向 CRC 算法，意思是将 g 左边的位看作是高位，右边的位看作低位。 G 的右边加 m 个 0 ，然后迭代计算是从高位开始，逐步将低位加入到寄存器中。在实际的数据传送过程中，是一边接收数据，一边计算 CRC 码，正向算法将新接收的数据看作低位。

逆向算法顾名思义就是将左边的数据看作低位，右边的数据看作高位。这样的话需要在 g 的左边加 m 个 0 ， h 也要逆向，例如正向 CRC-16 算法 h=0x4c11db8 ，逆向 CRC-16 算法 h=0xedb88320 。 b 的选择 0 还是 h ，由寄存器中右边第 1 位决定，而不是左边第 1 位。寄存器仍旧是向左位移，就是说迭代变成从低位到高位。

![](https://www.cnblogs.com/images/cnblogs_com/esestt/CRC/crc012.gif)

posted on [Cheney Shue](https://www.cnblogs.com/esestt)   阅读( 101681 )  评论( 23 ) [编辑](https://i.cnblogs.com/EditPosts.aspx?postid=848856) [收藏](https://www.cnblogs.com/esestt/archive/2007/08/09/) [举报](https://www.cnblogs.com/esestt/archive/2007/08/09/)

登录后才能查看或发表评论，立即 [登录](https://www.cnblogs.com/esestt/archive/2007/08/09/) 或者 [逛逛](https://www.cnblogs.com/) 博客园首页

[【推荐】100%开源！大型工业跨平台软件C++源码提供，建模，组态！](http://www.uccpsoft.com/index.htm)  
[【推荐】还在用 ECharts 开发大屏？试试这款永久免费的开源 BI 工具！](https://dataease.cn/?utm_source=cnblogs)  
[【推荐】国内首个AI IDE，深度理解中文开发场景，立即下载体验Trae](https://www.trae.com.cn/?utm_source=juejin&utm_medium=juejin_trae&utm_campaign=bokeyuan)  
[【推荐】编程新体验，更懂你的AI，立即体验豆包MarsCode编程助手](https://www.marscode.cn/?utm_source=advertising&utm_medium=cnblogs.com_ug_cpa&utm_term=hw_marscode_cnblogs&utm_content=home)  
[【推荐】轻量又高性能的 SSH 工具 IShell：AI 加持，快人一步](http://ishell.cc/)  
[![](https://img2024.cnblogs.com/blog/35695/202503/35695-20250307201637566-849030891.jpg)](https://www.trae.com.cn/?utm_source=juejin&utm_medium=juejin_trae&utm_campaign=bokeyuan)

**编辑推荐：**  
· [理解Rust引用及其生命周期标识（下）](https://www.cnblogs.com/w4ngzhen/p/18801243)  
· [从二进制到误差：逐行拆解C语言浮点运算中的4008175468544之谜](https://www.cnblogs.com/ging/p/18796255)  
· [.NET制作智能桌面机器人：结合BotSharp智能体框架开发语音交互](https://www.cnblogs.com/GreenShade/p/18771608)  
· [软件产品开发中常见的10个问题及处理方法](https://www.cnblogs.com/jiujuan/p/18794416)  
· [.NET 原生驾驭 AI 新基建实战系列：向量数据库的应用与畅想](https://www.cnblogs.com/code-daily/p/18784938)  

**阅读排行：**  
· [2025成都.NET开发者Connect圆满结束](https://www.cnblogs.com/edisonchou/p/-/chengdu-dotnet-developers-connect-2025)  
· [后端思维之高并发处理方案](https://www.cnblogs.com/skychen1218/p/17421134.html)  
· [千万级大表的优化技巧](https://www.cnblogs.com/12lisu/p/18801613)  
· [在 VS Code 中，一键安装 MCP Server！](https://www.cnblogs.com/formulahendry/p/18801464)  
· [10年+.NET Coder 心语 ── 继承的思维：从思维模式到架构设计的深度解析](https://www.cnblogs.com/code-daily/p/18801661)  

点击右上角即可分享 ![微信分享提示](https://img2023.cnblogs.com/blog/35695/202309/35695-20230906145857937-1471873834.gif) [AI原生IDE](https://www.trae.com.cn/?utm_source=juejin&utm_medium=juejin_trae&utm_campaign=bokeyuan)