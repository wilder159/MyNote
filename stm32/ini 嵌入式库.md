Inih

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/hyf_forward/article/details/134438479)

1. 引言
-----

### 0.1.1 编写目的

[ini 文件](https://so.csdn.net/so/search?q=ini%E6%96%87%E4%BB%B6&spm=1001.2101.3001.7020)是 initialization file 的缩写，即初始化文件，是 windows 系统配置文件所采用的存储格式，但在嵌入式系统下缺少 ini 文件解析库，需要自己去编写解析库。本文档描述的是 minIni 开源库，用于指导在嵌入式系 Times New Roman 统下使用 ini 文件解析库。

### 0.1.2 概述

minIni 是一种用于读取和写入 INI 文件的库，minIni 占用很少的资源，具有确定的内存占用空间。minIni 的主要用途是在运行 ROTS（甚至没用任何操作系统）的嵌入式系统上使用。MinIni 要求这样的系统提供一种存储和文件 I/O 系统，但是不需要此文件 I/O 系统与标准 C/C++ 库兼容。

### 0.1.3 特点

1）minIni 大约是 950 行代码（包含注释），是一种真正的 “迷你”INI 文件解析器，非常容易移植到各种嵌入式平台。

2）minIni 不需要标准 C/C++ 库的文件 I/O 函数，且允许通过宏配置配置文件 I/O 接口。

3）minIni 仅使用 stack，完全不使用动态内存（malloc）。

4）有 C++ binding，能够很好地配合 C++ 使用。

2. 使用方法
-------

### 0.1.4 下载链接

[http://GitHub - compuphase/minIni: A small and portable INI file library with read/write support](/minIni:%20A%20small%20and%20portable%20INI%20file%20library%20with%20read/write%20support "http://GitHub - compuphase/minIni: A small and portable INI file library with read/write support")

### 0.1.5 语法描述

ini 配置文件由节、键值对和注释组成，例如：

[section1]

Keyname1=value1

Keyname2=value2

[section2]#section name

Keyname3=value3

Keyname4=value4

1. 节（section）：

1）所有的键值对都是以节为单位结合在一起的。

2）所有的 section 名称都是独占一行，且 section 名字被方括号包围。

3）在 section 声明后的所有键值对都属于这个 section。

4）一个 section 没有明显的结束标识符，一个 section 的开始就是上一个 section 的结束

5）section 不能重复，数据通过 section 去查找，每个 section 下可以有多个 key 和 value 的键值对。

2. 键值对：

以键值对的形式存在，每个参数都有一个 name 和一个 value，name 和 value 由等号分隔。

3. 注释：

注释使用 #或分号表示开始，直到改好结尾全部为注解。

举例：
```shell
[eth]

ip=192.168.1.100  #ip 地址

netmask=255.255.255.0

Gateway=192.168.1.1
```


### 0.1.6 移植方法

以下是整个 minIni 开源库的文件结构，我们的嵌入式系统采用的是 FatFS+freeRTOS 系统开发，所以我们在移植的时候只需要 minGlue-FatFs.h、minIni.h、minIni.c 三个文件，其他头文件是根据不同的平台选择不同的头文件。

![[../_resources/ini 嵌入式库/79d5f036a3125b770a84aae214922e67_MD5.jpg]]

图 1 minIni 文件结构

将以上提到的三个文件添加到工程目录下，需要做以下修改。

1.MinGlue-Fatfs.h：添加包含 stdlib.h、stdio.h 和 string.h 头文件，例外如果需要支持处理浮点数，需要添加以下三行 #define 语句，如果不需要支持处理浮点，可不添加以下三行 #define。

![[../_resources/ini 嵌入式库/3b7eb1435718e1f646d9c8f3b0aefaa5_MD5.jpg]]

图 2 .minGlue-Fatfs 添加头文件

![[../_resources/ini 嵌入式库/0f62057f5a2d4ac4fc312199bbaf83a9_MD5.jpg]]

图 3 minGlue-Fatfs.h 添加支持处理浮点

2.minIni.h 文件

添加 #include "minGlue-FatFs.h" 文件，同时将 minIni.c 中的

#if !defined sizearray

  #define sizearray(a)    (sizeof(a) / sizeof((a)[0]))

#endif

移植到 minIni.h 头文件中。

### 0.1.7 **测试记录结果**

**![[../_resources/ini 嵌入式库/c608fd7bbd445e11a03ec8920e562121_MD5.jpg]]**图 4 测试代码

![[../_resources/ini 嵌入式库/7de8a1c3ad7f625d408f9a9e5f91fa3e_MD5.jpg]]

图 5 测试代码输出结果

![[../_resources/ini 嵌入式库/a549aa0a9b43fff6beef9b7c3b51b94a_MD5.jpg]]

图 6 配置文件

3. 结论
-----

通过以上测试验证，移植成功，但也不敢保证没有其他问题，还需要更进一步的测试验证。