简单理解为：MCU 中的库函数
GPIO 子系统是Linux 内核中用于管理 GPIO 资源的一套系统，它提供了许多与 GPIO 相关的 API 接口。

使用 GPIO 子系统相关的 API（函数）是通用的
目的就是为了减轻开发的压力
使用过程：申请 GPIO，定义 GPIO 的方向，设置 GPIO 的值，获取 GPIO 的值
**相关 API**
---
头文件：#include <linux/gpio.h>
**申请 GPIO**
---
int gpio_request(unsigned gpio, const char *tag)

函数参数
**gpio：GPIO 的定义 --- 要使用的 GPIO**
**对于当前的板子 **
**先输入 EXYNOS**

![[../_resources/八、GPIO子系统/78339570b62959c73bde5572e23eda73_MD5.png]]

**对于后续的开发 --- 设备树**
**gpio 会从设备树上获取 --- int 类型的变量**
label：标签 --- 无所谓，尽可能有意义
函数返回值
申请成功返回 0
失败返回负数
**GPIO 释放**
---
void gpio_free(unsigned gpio)
函数参数
就是申请到的 GPIO
**设置 GPIO 为输入模式**
---
static inline int gpio_direction_input(unsigned gpio)
函数参数
被设置的 GPIO
函数返回值
成功返回 0，失败返回负数
**设置 GPIO 为输出模式**
---
static inline int gpio_direction_output(unsigned gpio, int value)
函数参数
被设置的 GPIO
默认电平
1 就是高，0 就是低
函数返回值
成功返回 0，失败返回负数
**获取 GPIO 电平**
---
static inline int gpio_get_value(unsigned int gpio)
函数参数
被获取的 GPIO
函数返回值
高电平返回 1，低电平返回 0