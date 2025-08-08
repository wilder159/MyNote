> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/zzssdd2/p/15418778.html)

> **作者：**zzssdd2
> 
> **E-mail：**zzssdd2@foxmail.com

### 0.1.1 〇、前言

近段时间做的项目涉及到设备固件 OTA 升级相关工作，其中有用到`Ymodem`协议传输数据，故整理一下 Ymodem 协议的知识。一是为写上位机 / 下位机做准备，二是做个备忘便于以后用时查阅。

### 0.1.2 一、符号说明

> 协议中用到的符号及其对应的数值和含义说明

<table><thead><tr><th>符号</th><th>数值</th><th>含义</th></tr></thead><tbody><tr><td>SOH</td><td>0x01</td><td>128 字节数据包</td></tr><tr><td>STX</td><td>0x02</td><td>1024 字节数据包</td></tr><tr><td>EOT</td><td>0x04</td><td>结束传输</td></tr><tr><td>ACK</td><td>0x06</td><td>正确接收回应</td></tr><tr><td>NAK</td><td>0x15</td><td>错误接收回应</td></tr><tr><td>CAN</td><td>0x18</td><td>传输中止</td></tr><tr><td>C</td><td>0x43</td><td>请求数据</td></tr></tbody></table>

### 0.1.3 二、传输起始帧

> 接收方发起传输请求后由发送方发送的第一包数据

<table><thead><tr><th>帧头</th><th>帧序</th><th>帧序反码</th><th>文件名</th><th>文件大小</th><th>NULL</th><th>CRC-H</th><th>CRC-L</th></tr></thead><tbody><tr><td>SOH</td><td>00</td><td>FF</td><td>n Bytes</td><td>m Bytes</td><td>x Bytes</td><td>1 Byte</td><td>1 Byte</td></tr></tbody></table>

*   SOH：表示这个数据帧中包含着 128 个字节的数据段
*   00：表示数据帧序号，初始是 0，依次递增（满 255 从 0 开始）
*   FF：是帧序号的取反（提供一种数据是否正确的判断依据）
*   数据段 (128byte)：文件名 + 文件大小 + NULL
    *   文件名：例如文件名是`foo.c`，ACSII 字符转换为 HEX 字符就是`66 6F 6F 2E 63`。它在数据帧中存放格式为`66 6F 6F 2E 63 00` ，最后加一个 00 表示文件名字段结束。
    *   文件大小：假如上面的`foo.c`文件大小为 1024 字节，转化成 16 进制即 0x400。它在数据帧中存放格式为`34 30 30 00`，即 ASCII 格式的 400，最后加一个 00 表示文件大小字段结束
    *   NULL：数据部分大小为 128 字节，除去文件名与文件大小占用的空间外，剩余的 x Bytes 全部用 00 填充
*   CRC-H 和 CRC-L 分别表示 16 位 CRC 校验码的高 8 位与低 8 位，校验只针对数据段。

### 0.1.4 三、传输数据帧

> 文件内容数据包

<table><thead><tr><th>帧头</th><th>帧序</th><th>帧序反码</th><th>数据</th><th>CRC-H</th><th>CRC-L</th></tr></thead><tbody><tr><td>STX/SOH</td><td>num</td><td>~num</td><td>1024/128 Bytes</td><td>1 Byte</td><td>1 Byte</td></tr></tbody></table>

*   STX 表示这帧数据的数据段为 1024 字节，SOH 表示这帧数据的数据段为 128 字节
    
*   第一包帧序为 01，帧序反码为 FE，第二包帧序为 02，其反码为 FD...... 以此类推
    
*   CRC-H 和 CRC-L 分别表示 16 位 CRC 校验码的高 8 位与低 8 位，校验只针对数据段。
    
*   如果文件最后一帧数据在 128~1024 之间，则使用 STX 的 1024 字节传输，但是剩余空间全部用 0x1A 填充，如下结构：
    
    ```
    -------------------------------------------------------
    | STX | num | ~num | data[ ] | 1A ...1A | CRCH | CRCL |
    -------------------------------------------------------
    ```
    
*   如果文件最后一帧数据小于 128 字节，则选择 SOH 的 128 字节来传输，但是剩余空间全部用 0x1A 填充，如下结构：
    
    ```
    ------------------------------------------------------
    | SOH | num | ~num | data[ ] | 1A...1A | CRCH | CRCL |
    ------------------------------------------------------
    ```
    

### 0.1.5 四、传输结束帧

> 发送方发送的最后一包数据

<table><thead><tr><th>帧头</th><th>帧序</th><th>帧序反码</th><th>数据</th><th>CRC-H</th><th>CRC-L</th></tr></thead><tbody><tr><td>SOH</td><td>00</td><td>FF</td><td>NUL[128]</td><td>00</td><td>00</td></tr></tbody></table>

*   结束帧同样以 SOH 开头，表示后面跟着 128 字节的数据，结束帧的帧序也认为是 00 FF
*   结束帧的 128 字节的数据部分不存放任何信息，即 NUL[128] 全部用 00 填充
*   因为数据段全为 00，故校验码也为 00 00

### 0.1.6 五、传输流程

> 文件的传输分为如下几个阶段进行

*   **阶段 1：发起传输请求**

接收方给发送方不断地发送字符'C'，以期望收到发送方的数据响应。

```
接收方->>发送方: Char 'C'
接收方->>发送方: Char 'C'
......
```

*   **阶段 2：起始帧的发送及确认**

发送方收到接收方发来'C'字符，开始发送起始帧数据。等待接收方响应 ACK 标记，发送方收到 ACK 标记后等待接收方发送字符'C'则开始正式传输文件内容。

```
发送方->>接收方: Start packet
接收方->>发送方: ACK
接收方->>发送方: C
```

*   **阶段 3：文件内容的传输及确认**

发送方每发一个文件内容数据包，就期待接收方响应一个 ACK 标记，以继续发送下一个包。

```
发送方->>接收方: Packet 1
接收方->>发送方: ACK
发送方->>接收方: Packet 2
接收方->>发送方: ACK
......
```

*   **传输过程中的异常处理**

若发送方发完数据包后收到了接收方 NAK 标记的响应，则重发此包，直到收到 ACK 响应或者超时退出。

```
发送方->>接收方: Packet n
接收方->>发送方: NAK
发送方->>接收方: Packet n
接收方->>发送方: ACK
发送方->>接收方: Packet n+1
......
```

若发送方发完数据包后收到了接收方 CAN 标记的响应，则停止数据包发送，结束传输。

```
......
发送方->>接收方: Packet n
接收方->>发送方: ACK
发送方->>接收方: Packet n+1
接收方->>发送方: CAN
中止传输
```

*   **阶段 4：数据传输结束**

若发送方已将数据包全部发完，则发送 EOT 标记等待接收方的 NAK 响应，当发送方收到 NAK 后会再次发送 EOT 等待接收方的 C 标记来请求结束帧，发送结束帧后收到接收方的 ACK 标记则表示本次传输完成

```
发送方->>接收方: EOT
接收方->>发送方: NAK
发送方->>接收方: EOT
接收方->>发送方: C
发送方->>接收方: Over packet
接收方->>发送方: ACK
传输结束
```

### 0.1.7 六、实例说明

> 假设以 foo.c，大小为 4196Bytes（16 进制为 0x1064）的文件作为传输对象，则它的传输过程如下：

<table><thead><tr><th>发送方</th><th>传输方向</th><th>接收方</th></tr></thead><tbody><tr><td></td><td>&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;</td><td>C（请求起始帧）</td></tr><tr><td>SOH 00 FF 66 6F 6F 2E 63 00 31 30 36 34 00 NUL[117] CRCH CRCL</td><td>&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;</td><td></td></tr><tr><td></td><td>&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;</td><td>ACK</td></tr><tr><td></td><td>&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;</td><td>C（请求数据帧）</td></tr><tr><td>STX 01 FE data[1024] CRCH CRCL</td><td>&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;</td><td></td></tr><tr><td></td><td>&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;</td><td>ACK</td></tr><tr><td>STX 02 FD data[1024] CRCH CRCL</td><td>&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;</td><td></td></tr><tr><td></td><td>&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;</td><td>ACK</td></tr><tr><td>STX 03 FC data[1024] CRCH CRCL</td><td>&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;</td><td></td></tr><tr><td></td><td>&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;</td><td>NAK(接收错误请求重发)</td></tr><tr><td>STX 03 FC data[1024] CRCH CRCL</td><td>&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;</td><td></td></tr><tr><td></td><td>&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;</td><td>ACK</td></tr><tr><td>STX 04 FB data[1024] CRCH CRCL</td><td>&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;</td><td></td></tr><tr><td></td><td>&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;</td><td>ACK</td></tr><tr><td>SOH 05 FA data[100] 1A[28] CRCH CRCL</td><td>&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;</td><td></td></tr><tr><td></td><td>&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;</td><td>ACK</td></tr><tr><td>EOT</td><td>&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;</td><td></td></tr><tr><td></td><td>&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;</td><td>NAK（响应结束命令）</td></tr><tr><td>EOT</td><td>&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;</td><td></td></tr><tr><td></td><td>&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;</td><td>C（请求结束帧）</td></tr><tr><td>SOH 00 FF NUL[128] CRCH CRCL</td><td>&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;</td><td></td></tr><tr><td></td><td>&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;&lt;</td><td>ACK</td></tr></tbody></table>

### 0.1.8 七、CRC16 校验

> 数据帧中校验码的计算方式——C 语言

```
/******************************************************************************************
*	函	数: CRC16_Xmodem
*	描	述: 计算CRC16-Xmodem
*		    多项式x16+x12+x5+1(0x1021) | 初始值0x0000 | 低位在后,高位在前 | 结果与0x0000异或
*	输	入: pData : 数据指针
*			ulSize : 数据长度
*	输	出: CRC16检验码
******************************************************************************************/
uint16_t CRC16_Xmodem(uint8_t *pData, uint32_t ulSize)  
{  
	uint8_t i;
    uint16_t usCRC = 0x0000;  
    
    while (ulSize--)     
    {  
        usCRC ^= (*pData++ << 8);
        for(i = 0; i < 8; i++)  
        {  
            if(usCRC & 0x8000)  
            {
                usCRC = (CRCin << 1) ^ 0x1021;  
            }
            else
            {              
                usCRC = usCRC << 1;
            }
        }  
    }  
    
    return usCRC;  
}
```