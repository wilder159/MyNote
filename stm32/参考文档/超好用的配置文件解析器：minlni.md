> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/511249743)

哈喽，我是老吴。这两天发现一个还不错的开源项目，记录一下学习心得。

对于嵌入式开发，INI 文件使用门槛低、用途广，掌握一个稳定的开源 INI 解析器可以提升编码效率、避免重复造轮子。

另外，对于一个 C 初学者，MinIni 代码严谨且健壮，API 设计合理，是一个非常适合用来练习文本读写、字符串操作的开源项目。  

minIni 是什么？
-----------

```
https://github.com/compuphase/minIni
```

minIni 是一个用于读取和写入 INI 文件的库。

**什么是 INI 文件？**

INI 是一种有简单语法的纯文本文件，例如：

```
[Network]
hostname=My Computer
address=dhcp
dns=192.168.1.1
timeout=10
```

[Network] 是一个 Section，Section 下面有多个键值对。

**INI 文件的特点是简单易用、可读性好。**

Linux 系统 etc 目录下的许多配置文件都是 INI 文件，只是有的配置文件没有显式声明 Section。

例如 bluez 的配置文件 /etc/bluetooth/main.conf：

```
[General]
# Default adaper name
Name = BlueZ

[Policy]
# AutoEnable defines option to enable all controllers when they are found.
AutoEnable=true
```

**minIni 的几个特点：**

1> minIni 大约是 950 行代码 (包括注释)，是一个真正的 “迷你” INI 文件解析器，非常容易移植到各种嵌入式平台。

2> minIni 不需要标准 C/C++ 库中的 文件 I/O 函数，且允许通过宏配置要选择文件 I/O 接口。

3> minIni 仅使用 stack ，完全不使用动态内存（malloc）。

4> 有 C++ binding, 能很好地配合 C++ 使用。  

### minIni 怎么用？

minIni 的核心源码就一个 .c 和 .h 文件，我们直接将它们集成到项目代码里即可。

**快速地了解它的 API**:

```
#include "minIni.h"

#define sizearray(a)  (sizeof(a) / sizeof((a)[0]))

const char inifile[] = "example.ini";

int main(void)
{
    char str[100];
    char section[50];
    long n;

    n = ini_gets("Network", "address", "dummy", str, sizearray(str), inifile);
    if (n >= 0) printf("Network/address=%s", str);

    n = ini_getl("Network", "timeout", -1, inifile);
    printf("Network/timeout=%ld\n", n);
```

待解析的文件内容：

```
[Network]
hostname=My Computer
address=dhcp
dns=192.168.1.1
timeout=10
```

运行结果：

```
Network/address=dhcp
Network/timeout=10
```

ini_gets() 用于获取字符串类型的值。

参数 1 是 Section name;

参数 2 是 Key name;

参数 3 是获取不到值时的默认值;

参数 4 是用于保存目标键值的 Buffer;

参数 5 是 Buffer 的长度;

参数 6 是 INI 文件的路径;

ini_getl() 用于获取整型类型的值，参数的含义和 ini_gets() 大同小异。

**完整的 API 列表：**

![[../../_resources/超好用的配置文件解析器：minlni/01f7ca3223d70d540a499377216b897d_MD5.png]]

### minIni 的内部实现？

我简单地浏览过 minIni 的实现代码，感觉良好。

想实现解析 INI 文件的功能并不难，难的是要认真考虑各种边界情况和异常情况，保证程序的健壮性和 API 的合理性。

**以 ini_gets() 为例：**

![[../../_resources/超好用的配置文件解析器：minlni/fbbdd40251b2f2099022deacb530f619_MD5.png]]

就是先打开文件，然后用 getkeystring() 找到目标键值，最后拷贝给调用者。

**getkeystring() 负责真正地解析文本：**

大约有 80 行代码，我简单地总结一下思路：

1> 用 fgets 进行逐行读取，用 strrchr 找到包含 '[' 和 ']' 的行，然后再用 strncasecmp 找到目标 Secntion 所在的行。

2> 继续用 fgets 进行逐行读取，用 strrchr 找到包含 '=' 的行，然后再用 strncasecmp 找到目标 Key 所在的行。

3> 用 strncpy 将目标 Key 的值拷贝给调用者。

虽然解析文本只需要做到这 3 个步骤，但是由于需要对各种边界情况进行判断，以及处理空格之类的无效字符，所以需要细心且反复调试才能编写好这个函数。  

总结
--

对于嵌入式开发，INI 文件使用门槛低、用途广，掌握一个稳定的开源 INI 解析器可以提升编码效率、避免重复造轮子。

对于一个 C 初学者，MinIni 代码严谨且健壮，API 设计合理，是一个非常适合用来练习文本读写、字符串操作的开源项目，值得一看。

**其他 INI 解析的开源项目：**

[https://github.com/rxi/ini](https://link.zhihu.com/?target=https%3A//github.com/rxi/ini)

[https://github.com/benhoyt/inih](https://link.zhihu.com/?target=https%3A//github.com/benhoyt/inih)

感谢阅读，欢迎转发！
----------

—— The End ——
-------------

推荐阅读：
-----

[专辑 | Linux 系统编程](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/mp/appmsgalbum%3F__biz%3DMzU3NDY4NTk3Mg%3D%3D%26action%3Dgetalbum%26album_id%3D1378333579549491203%23wechat_redirect)[专辑 | Linux 驱动开发](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/mp/appmsgalbum%3F__biz%3DMzU3NDY4NTk3Mg%3D%3D%26action%3Dgetalbum%26album_id%3D1378331497144664066%23wechat_redirect)[专辑 | Linux 内核品读](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/mp/appmsgalbum%3F__biz%3DMzU3NDY4NTk3Mg%3D%3D%26action%3Dgetalbum%26album_id%3D1378335865025740805%23wechat_redirect)[专辑 | 每天一点 C](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/mp/appmsgalbum%3F__biz%3DMzU3NDY4NTk3Mg%3D%3D%26action%3Dgetalbum%26album_id%3D1437817804165890049%23wechat_redirect)[专辑 | 开源软件](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/mp/appmsgalbum%3F__biz%3DMzU3NDY4NTk3Mg%3D%3D%26action%3Dgetalbum%26album_id%3D1378339777707393025%23wechat_redirect)[专辑 | Qt 入门](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/mp/appmsgalbum%3F__biz%3DMzU3NDY4NTk3Mg%3D%3D%26action%3Dgetalbum%26album_id%3D1820872276280426502%23wechat_redirect)