---
title: "minIni - a minimal INI file parser_minini的使用教程-CSDN博客"
source: "https://blog.csdn.net/hefangchi/article/details/107935179"
author:
published:
created: 2025-03-17
description: "文章浏览阅读1.9k次。minIni是一款轻量级的嵌入式INI文件解析器，专为资源受限的嵌入式系统设计。它占用资源少，内存占用确定，支持多种文件I/O库。minIni适用于运行RTOS或无操作系统的设备，提供读取、写入和删除INI文件键值的能力。"
tags:
  - "clippings"
---
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/hefangchi/article/details/107935179)


minIni 是一个程序员的库，用于在嵌入式系统中读取和写入 “INI” 文件。minIni 占用很少的资源，具有确定的内存占用空间，并且可以针对各种文件 I / O 库进行配置。minIni 提供了用于从 INI 文件中读取，写入和删除密钥的功能，全部功能都用 830 行 C 语言的（注释）源代码（1.2 版）（该代码也用 C ++ 编译，并带有包装器类）。

minIni 的主要用途是在运行 RTOS（甚至没有任何操作系统）的嵌入式系统上使用。minIni 要求这样的系统提供一种存储和文件 I / O 系统，但是不需要此文件 I / O 系统与标准 C / C ++ 库兼容 - 实际上，标准库也经常与之兼容。嵌入式系统的庞大资源需求。

如果您想了解有关 minIni 的更多信息，请参见源代码存档中的手册。

### 0.1.1 _下载和许可_

*   [minIni 1.2.a 源存档](https://www.compuphase.com/software/minini_12b.zip) （195 KiB，包括手册）。
*   可以在 [GitHub 上](https://github.com/compuphase/minIni)访问最新的源代码

minIni 库是根据 [Apache 许可证 2.0 版](http://www.opensource.org/licenses/apache2.0.php)分发的 ，外加[一个例外条款，](https://www.compuphase.com/minini.htm#APACHE_EXCEPTION)以明确允许将该库静态链接到商业应用程序。

### 0.1.2 _INI 文件语法_

INI 文件具有简单的语法，在纯文本文件中具有名称 / 值对。名称必须唯一（每节），并且值必须在一行中。INI 文件通常分为几部分 - 在 minIni 中，这是可选的。节是方括号之间的名称，`[Network]`例如下面的示例中的 “ ”。

```
[Network]
hostname=My Computer
address=dhcp
dns = 192.168.1.1
```

在 API 和本文档中，设置的 “名称” 表示为设置的 _键_。键和值之间用等号（“ `=`”）分隔。minIni 支持使用冒号（“ `:`”）替代键 / 值定界符的等号。

值或键名周围的前导空格被删除。如果需要在值中包含前导和 / 或尾随空格，请将值放在双引号之间。该`ini_gets()`函数（来自 minIni 库，请参见 minIni 手册）从返回值中去除双引号。`ini_puts()`如果要写入的值包含尾随空格（或特殊字符），则函数 将双引号引起来。

minIni 会忽略定界符 “ `=`” 或 “ `:`” 周围的空格，但不会忽略节名称中括号之间的空格。换句话说，最好不要在节名称`[`的左括号 “ `]`” 后面或右括号 “ ” 之前放置空格。

INI 中的注释必须以分号（“;”）或井号（“＃”）开头，并一直到行尾。注释可以是一行，也可以是键 / 值对（“＃” 字符和尾随注释是 minIni 的扩展名）。

### 0.1.3 _INI 文件读取范例_

有两种方法可以从 INI 文件中读取设置。一种方法是调用一个函数，例如`GetProfileString()`针对所需的每个部分和键。如果有一个大的 INI 文件，这特别方便，但是您随时都需要从该文件中进行一些设置，尤其是在程序运行时 INI 文件也可以更改的情况下。这是 Microsoft Windows API 使用的方法。

但是，当您需要连续从 INI 文件中检索很多设置时，尤其是如果 INI 文件未缓存在内存中（在 minIni 中不是），上述过程效率很低。从 INI 文件中获取设置的另一种方法是调用 “解析” 函数，然后让该函数使用部分和键名以及相关数据来调用应用程序。XML 解析库通常使用这种方法。参见例如 [Expat 库](http://expat.sourceforge.net/)。

minIni 支持两种方法。要读取单个设置，请使用功能 `ini_gets()`。对于回调方法，实现一个回调并调用 `ini_browse()`。有关详细信息，请参见 minIni 手册。

### 0.1.4 _使 minIni 适应文件系统_

必须借助所谓的 “胶水文件” 为平台配置 minIni 库。此粘合文件包含宏（可能还有函数），这些宏将 minIni 库使用的文件读写功能映射到操作系统提供的功能。胶合文件必须称为“ `minGlue.h`”。

首先，minIni 发行版附带以下示例粘合文件：

*   映射到标准 C / C ++ 库的粘合文件（特别是 “ `stdio`” 包中的文件 I / O 函数），
*   [Microchip 的内存磁盘驱动器文件系统库](http://www.microchip.com/stellent/idcplg?IdcService=SS_GET_PAGE&nodeId=1824&appnote=en532040) 的粘合文件，
*   [CCS PIC 编译器](http://www.ccsinfo.com/content.php?page=compilers) 随附的 FAT 库的粘合文件，
*   [EFS 库（EFSL）](http://www.efsl.be/) 的粘合文件，
*   以及 [FatFs 和 Petit-FatFs](http://elm-chan.org/fsw/ff/00index_e.html) 库的胶合文件。

### 0.1.5 _多任务_

minIni 库没有任何全局变量，并且不使用任何动态分配的内存。但是，该库不应被认为是 “线程安全的” 或可重入的，因为它隐式使用了一种特殊的共享资源：文件系统。

从 INI 文件读取多个任务不会造成问题。但是，当一个任务正在写入 INI 文件时，没有其他任务可以访问该 INI 文件 - 既不读取也不写入。在实现中，序列化 INI 文件的_所有_访问可能会更容易。

在多任务环境中保护资源免受并发访问的第一个建议是避免在任务之间共享资源。如果仅单个任务使用资源，则不需要信号量保护，也不会发生优先级倒置或死锁问题。此建议也适用于 minIni 库。如果可能，将单个任务设为 INI 文件的 “所有者”，并为其他任务创建客户端 / 服务器体系结构以查询和调整设置。

如果必须在任务之间共享 INI 文件（并且至少有一个任务写入了 INI 文件），则需要围绕 minIni 库的函数编写包装程序，这些函数在互斥量或二进制信号量上阻塞。  
 

```
test.ini
 
[First]
String=noot # trailing commment
Val=1
 
[Second]
Val = 2
#comment=3
String = mies
```

```c
/* Simple test program
 *
 *  gcc -o test test.c minIni.c
 */
#include <assert.h>
#include <stdio.h>
#include <string.h>
#include "minIni.h"
 
#define sizearray(a)  (sizeof(a) / sizeof((a)[0]))
 
const char inifile[] = "test.ini";
const char inifile2[] = "testplain.ini";
 
int Callback(const char *section, const char *key, const char *value, const void *userdata)
{
  (void)userdata; /* this parameter is not used in this example */
  printf("    [%s]\t%s=%s\n", section, key, value);
  return 1;
}
 
int main(void)
{
  char str[100];
  long n;
  int s, k;
  char section[50];
 
  /* string reading */
  n = ini_gets("first", "string", "dummy", str, sizearray(str), inifile);
  assert(n==4 && strcmp(str,"noot")==0);
  n = ini_gets("second", "string", "dummy", str, sizearray(str), inifile);
  assert(n==4 && strcmp(str,"mies")==0);
  n = ini_gets("first", "undefined", "dummy", str, sizearray(str), inifile);
  assert(n==5 && strcmp(str,"dummy")==0);
  /* ----- */
  n = ini_gets("", "string", "dummy", str, sizearray(str), inifile2);
  assert(n==4 && strcmp(str,"noot")==0);
  n = ini_gets(NULL, "string", "dummy", str, sizearray(str), inifile2);
  assert(n==4 && strcmp(str,"noot")==0);
  /* ----- */
  printf("1. String reading tests passed\n");
 
  /* value reading */
  n = ini_getl("first", "val", -1, inifile);
  assert(n==1);
  n = ini_getl("second", "val", -1, inifile);
  assert(n==2);
  n = ini_getl("first", "undefined", -1, inifile);
  assert(n==-1);
  /* ----- */
  n = ini_getl(NULL, "val", -1, inifile2);
  assert(n==1);
  /* ----- */
  printf("2. Value reading tests passed\n");
 
  /* string writing */
  n = ini_puts("first", "alt", "flagged as \"correct\"", inifile);
  assert(n==1);
  n = ini_gets("first", "alt", "dummy", str, sizearray(str), inifile);
  assert(n==20 && strcmp(str,"flagged as \"correct\"")==0);
  /* ----- */
  n = ini_puts("second", "alt", "correct", inifile);
  assert(n==1);
  n = ini_gets("second", "alt", "dummy", str, sizearray(str), inifile);
  assert(n==7 && strcmp(str,"correct")==0);
  /* ----- */
  n = ini_puts("third", "test", "correct", inifile);
  assert(n==1);
  n = ini_gets("third", "test", "dummy", str, sizearray(str), inifile);
  assert(n==7 && strcmp(str,"correct")==0);
  /* ----- */
  n = ini_puts("second", "alt", "overwrite", inifile);
  assert(n==1);
  n = ini_gets("second", "alt", "dummy", str, sizearray(str), inifile);
  assert(n==9 && strcmp(str,"overwrite")==0);
  /* ----- */
  n = ini_puts(NULL, "alt", "correct", inifile2);
  assert(n==1);
  n = ini_gets(NULL, "alt", "dummy", str, sizearray(str), inifile2);
  assert(n==7 && strcmp(str,"correct")==0);
  /* ----- */
  printf("3. String writing tests passed\n");
 
  /* section/key enumeration */
  printf("4. Section/key enumertion, file contents follows\n");
  for (s = 0; ini_getsection(s, section, sizearray(section), inifile) > 0; s++) {
    printf("    [%s]\n", section);
    for (k = 0; ini_getkey(section, k, str, sizearray(str), inifile) > 0; k++) {
      printf("\t%s\n", str);
    } /* for */
  } /* for */
 
  /* browsing through the file */
  printf("5. browse through all settings, file contents follows\n");
  ini_browse(Callback, NULL, inifile);
 
  /* string deletion */
  n = ini_puts("first", "alt", NULL, inifile);
  assert(n==1);
  n = ini_puts("second", "alt", NULL, inifile);
  assert(n==1);
  n = ini_puts("third", NULL, NULL, inifile);
  assert(n==1);
  /* ----- */
  n = ini_puts(NULL, "alt", NULL, inifile2);
  assert(n==1);
  printf("6. String deletion tests passed\n");
 
  return 0;
}
```