---
title: "如何使用 ESP32 AT 固件设置 HTTP Get 数据断点续传或分段续传？_esp ap 断点续传-CSDN博客"
source: "https://blog.csdn.net/weixin_42083266/article/details/128568066"
author:
published:
created: 2025-03-17
description: "文章浏览阅读815次。文章介绍了两种在ESP32上实现HTTPGet数据的方法。第一种方法利用AT+HTTPCLIENT指令和HTTP的Range头进行断点续传，要求服务器支持。第二种方法依赖UART硬件流控，通过MCU控制CTS管脚来暂停或恢复数据传输，确保MCU能及时处理数据。"
tags:
  - "clippings"
---
s> **需求：ESP32 通过 `HTTP Get` 数据，每次 Get 一包 `2048 字节`数据后，等待主控 MCU 回复应答，再继续 `Get` 下一包数据。**

### 0.1.1 实现方法 1 ：

> **要求：此方法要求`服务器`端也支持`断点续传`**

- **可以使用 [AT+HTTPCLIENT](https://docs.espressif.com/projects/esp-at/zh_CN/latest/esp32/AT_Command_Set/HTTP_AT_Commands.html#at-httpclient-http) 指令，通过设置 `<”http_req_header”>` 参数来指定 Get 数据的 Range 。具体说明如下：**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fc2dda9525f25556a728cd513081445f.png)

- **参考 AT 指令如下:**

```bash
AT+HTTPCLIENT=2,0,"http://www.baidu.com/img/bdlogo.gif",,,0,"Range: bytes=0-2048" 

AT+HTTPCLIENT=2,0,"http://www.baidu.com/img/bdlogo.gif",,,0,"Range: bytes=2048-4096" 

123
```

---

### 0.1.2 实现方法 2 ：

- **也可以使用 `UART 硬件流控`设置数据缓冲，这样 MCU 端来不及处理命令的时候，可以通过控制 `UART CTS` 管脚来阻止 AT 给 MCU 发数据。**

> **要求：使用 `UART 硬件流控`的方式，需要[主 MCU 控制 AT 设备的 CTS](https://www.cnblogs.com/aaronLinux/p/5668414.html) 管脚是`拉低`或`拉高`；**

- **AT 设备这边需要接 UART 流控管脚**  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e8c1cc26445590c03e7d1150c315fb4d.png)
- **通过 [AT+UART\_CUR](https://docs.espressif.com/projects/esp-at/zh_CN/release-v2.4.0.0/esp32/AT_Command_Set/Basic_AT_Commands.html#id16) 命令可使能 UART 流控**