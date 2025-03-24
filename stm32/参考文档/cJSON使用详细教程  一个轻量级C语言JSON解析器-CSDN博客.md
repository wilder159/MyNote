---
title: "cJSON使用详细教程 | 一个轻量级C语言JSON解析器-CSDN博客"
source: "https://blog.csdn.net/Mculover666/article/details/103796256"
author:
published:
created: 2025-03-03
description: "文章浏览阅读10w+次，点赞585次，收藏2.2k次。本文深入探讨了JSON数据格式及其C语言实现cJSON的使用。介绍了JSON的基本概念、语法规则，cJSON的特性、数据结构及设计思想。详细讲解了如何使用cJSON封装和解析JSON数据，包括创建、添加、输出JSON数据以及解析方法和注意事项。"
tags:
  - "clippings"
---
## 0.1 1\. JSON与cJSON

### 0.1.1 JSON —— 轻量级的数据格式

[JSON](https://www.json.org/) 全称 JavaScript Object Notation，即 JS对象简谱，是一种轻量级的数据格式。

它采用完全独立于编程语言的文本格式来存储和表示数据，语法简洁、层次结构清晰，易于人阅读和编写，同时也易于机器解析和生成，有效的提升了网络传输效率。

### 0.1.2 JSON语法规则

JSON对象是一个无序的"名称/值"键值对的集合：

- 以"`{`“开始，以”`}`"结束，允许嵌套使用；
- 每个名称和值成对出现，名称和值之间使用"`:`"分隔；
- 键值对之间用"`,`"分隔
- 在这些字符前后允许存在无意义的空白符；

对于键值，可以有如下值：

- 一个新的json对象
- 数组：使用"`[`“和”`]`"表示
- 数字：直接表示，可以是整数，也可以是浮点数
- 字符串：使用引号`"`表示
- 字面值：false、null、true中的一个(必须是小写)

示例如下：

```json
{
    "name": "mculover666",
    "age": 22,
    "weight": 55.5,
    "address":
    {
        "country": "China",
        "zip-code": 111111
    },
    "skill": ["c", "Java", "Python"],
    "student": false
}

123456789101112
```

### 0.1.3 cJSON

cJSON是一个使用C语言编写的JSON数据解析器，具有超轻便，可移植，单文件的特点，使用MIT开源协议。

cJSON项目托管在Github上，仓库地址如下：

> https://github.com/DaveGamble/cJSON

使用Git命令将其拉取到本地：

```bash
git clone https://github.com/DaveGamble/cJSON.git

1
```

从Github拉取cJSON源码后，文件非常多，但是其中cJSON的源码文件只有两个：

- `cJSON.h`
- `cJSON.c`

使用的时候，只需要将这两个文件复制到工程目录，然后包含头文件`cJSON.h`即可，如下：

```c
#include "cJSON.h"

1
```

## 0.2 2\. cJSON数据结构和设计思想

**cJSON的设计思想从其数据结构上就能反映出来。**

cJSON使用cJSON结构体来表示**一个JSON数据**，定义在`cJSON.h`中，源码如下：

```c
/* The cJSON structure: */
typedef struct cJSON
{
    /* next/prev allow you to walk array/object chains. Alternatively, use GetArraySize/GetArrayItem/GetObjectItem */
    struct cJSON *next;
    struct cJSON *prev;
    /* An array or object item will have a child pointer pointing to a chain of the items in the array/object. */
    struct cJSON *child;

    /* The type of the item, as above. */
    int type;

    /* The item's string, if type==cJSON_String  and type == cJSON_Raw */
    char *valuestring;
    /* writing to valueint is DEPRECATED, use cJSON_SetNumberValue instead */
    int valueint;
    /* The item's number, if type==cJSON_Number */
    double valuedouble;

    /* The item's name string, if this item is the child of, or is in the list of subitems of an object. */
    char *string;
} cJSON;

12345678910111213141516171819202122
```

cJSON的设计很巧妙。

首先，**它不是将一整段JSON数据抽象出来，而是将其中的一条JSON数据抽象出来，也就是一个键值对**，用上面的结构体 `strcut cJSON` 来表示，其中用来存放值的成员列表如下：

- `String`：用于表示该键值对的名称；
- `type`：用于表示该键值对中值的类型；
- `valuestring`：如果键值类型(type)是字符串，则将该指针指向键值；
- `valueint`：如果键值类型(type)是整数，则将该指针指向键值；
- `valuedouble`：如果键值类型(type)是浮点数，则将该指针指向键值；

其次，一段完整的JSON数据中由很多键值对组成，并且涉及到键值对的查找、删除、添加，所以**使用链表来存储整段JSON数据**，如上面的代码所示：

- `next`指针：指向下一个键值对
- `prev`指针指向上一个键值对

最后，因为JSON数据支持嵌套，所以一个**键值对的值会是一个新的JSON数据对象（一条新的链表）**，也有可能是一个数组，方便起见，在cJSON中，数组也表示为一个数组对象，用链表存储，所以：

在键值对结构体中，当该键值对的值是一个嵌套的JSON数据或者一个数组时，由`child`指针指向该条新链表。

## 0.3 3\. JSON数据封装

### 0.3.1 封装方法

封装JSON数据的过程，其实就是创建链表和向链表中添加节点的过程。

首先来讲述一下链表中的一些术语：

- 头指针：指向链表头结点的指针；
- 头结点：不存放有效数据，方便链表操作；
- 首节点：第一个存放有效数据的节点；
- 尾节点：最后一个存放有效数据的节点；

明白了这几个概念之后，我们开始讲述创建一段完整的JSON数据，即如何创建一条完整的链表。

- ① 创建头指针：

```c
 cJSON* cjson_test = NULL;

1
```

- ② 创建头结点，并将头指针指向头结点：

```c
cjson_test = cJSON_CreateObject();

1
```

- ③ 尽情的向链表中添加节点：

```
cJSON_AddNullToObject(cJSON * const object, const char * const name);

cJSON_AddTrueToObject(cJSON * const object, const char * const name);

cJSON_AddFalseToObject(cJSON * const object, const char * const name);

cJSON_AddBoolToObject(cJSON * const object, const char * const name, const cJSON_bool boolean);

cJSON_AddNumberToObject(cJSON * const object, const char * const name, const double number);

cJSON_AddStringToObject(cJSON * const object, const char * const name, const char * const string);

cJSON_AddRawToObject(cJSON * const object, const char * const name, const char * const raw);

cJSON_AddObjectToObject(cJSON * const object, const char * const name);

cJSON_AddArrayToObject(cJSON * const object, const char * const name);

1234567891011121314151617
```

### 0.3.2 输出JSON数据

上面讲述，一段完整的JSON数据就是一条长长的链表，那么，如何打印出这段JSON数据呢？

cJSON提供了一个API，可以将整条链表中存放的JSON信息输出到一个字符串中：

```c
(char *) cJSON_Print(const cJSON *item);

1
```

使用的时候，只需要接收该函数返回的指针地址即可。

### 0.3.3 封装数据和打印数据示例

单纯的讲述方法还不够，下面用一个例子来说明，封装出开头给出的那段JSON数据：

```c
#include <stdio.h>
#include "cJSON.h"

int main(void)
{
    cJSON* cjson_test = NULL;
    cJSON* cjson_address = NULL;
    cJSON* cjson_skill = NULL;
    char* str = NULL;

    /* 创建一个JSON数据对象(链表头结点) */
    cjson_test = cJSON_CreateObject();

    /* 添加一条字符串类型的JSON数据(添加一个链表节点) */
    cJSON_AddStringToObject(cjson_test, "name", "mculover666");

    /* 添加一条整数类型的JSON数据(添加一个链表节点) */
    cJSON_AddNumberToObject(cjson_test, "age", 22);

    /* 添加一条浮点类型的JSON数据(添加一个链表节点) */
    cJSON_AddNumberToObject(cjson_test, "weight", 55.5);

    /* 添加一个嵌套的JSON数据（添加一个链表节点） */
    cjson_address = cJSON_CreateObject();
    cJSON_AddStringToObject(cjson_address, "country", "China");
    cJSON_AddNumberToObject(cjson_address, "zip-code", 111111);
    cJSON_AddItemToObject(cjson_test, "address", cjson_address);

    /* 添加一个数组类型的JSON数据(添加一个链表节点) */
    cjson_skill = cJSON_CreateArray();
    cJSON_AddItemToArray(cjson_skill, cJSON_CreateString( "C" ));
    cJSON_AddItemToArray(cjson_skill, cJSON_CreateString( "Java" ));
    cJSON_AddItemToArray(cjson_skill, cJSON_CreateString( "Python" ));
    cJSON_AddItemToObject(cjson_test, "skill", cjson_skill);

    /* 添加一个值为 False 的布尔类型的JSON数据(添加一个链表节点) */
    cJSON_AddFalseToObject(cjson_test, "student");

    /* 打印JSON对象(整条链表)的所有数据 */
    str = cJSON_Print(cjson_test);
    printf("%s\n", str);

    return 0;
}

123456789101112131415161718192021222324252627282930313233343536373839404142434445
```

编译运行：

```c
gcc cJSON.c example1.c -o example1.exe

1
```

实验结果如图：

![实验结果](https://i-blog.csdnimg.cn/blog_migrate/03ce9710f834180ddde11260da4da6bb.png)

该JSON数据链表的关系如图：

![JSON关系图](https://i-blog.csdnimg.cn/blog_migrate/77f1a6ca52e327c827281a121735b632.png)

## 0.4 4\. cJSON数据解析

### 0.4.1 解析方法

解析JSON数据的过程，其实就是剥离一个一个链表节点(键值对)的过程。

解析方法如下：

- ① 创建链表头指针：

```c
cJSON* cjson_test = NULL;

1
```

- ② 解析整段JSON数据，并将链表头结点地址返回，赋值给头指针：

解析整段数据使用的API只有一个：

```c
(cJSON *) cJSON_Parse(const char *value);

1
```

- ③ 根据键值对的名称从链表中取出对应的值，返回该键值对(链表节点)的地址

```c
(cJSON *) cJSON_GetObjectItem(const cJSON * const object, const char * const string);

1
```

- ④ 如果JSON数据的值是数组，使用下面的两个API提取数据：

```c
(int) cJSON_GetArraySize(const cJSON *array);
(cJSON *) cJSON_GetArrayItem(const cJSON *array, int index);

12
```

### 0.4.2 解析示例

下面用一个例子来说明如何解析出开头给出的那段JSON数据：

```c
#include <stdio.h>
#include "cJSON.h"

char *message = 
"{                              \
    \"name\":\"mculover666\",   \
    \"age\": 22,                \
    \"weight\": 55.5,           \
    \"address\":                \
        {                       \
            \"country\": \"China\",\
            \"zip-code\": 111111\
        },                      \
    \"skill\": [\"c\", \"Java\", \"Python\"],\
    \"student\": false          \
}";

int main(void)
{
    cJSON* cjson_test = NULL;
    cJSON* cjson_name = NULL;
    cJSON* cjson_age = NULL;
    cJSON* cjson_weight = NULL;
    cJSON* cjson_address = NULL;
    cJSON* cjson_address_country = NULL;
    cJSON* cjson_address_zipcode = NULL;
    cJSON* cjson_skill = NULL;
    cJSON* cjson_student = NULL;
    int    skill_array_size = 0, i = 0;
    cJSON* cjson_skill_item = NULL;

    /* 解析整段JSO数据 */
    cjson_test = cJSON_Parse(message);
    if(cjson_test == NULL)
    {
        printf("parse fail.\n");
        return -1;
    }

    /* 依次根据名称提取JSON数据（键值对） */
    cjson_name = cJSON_GetObjectItem(cjson_test, "name");
    cjson_age = cJSON_GetObjectItem(cjson_test, "age");
    cjson_weight = cJSON_GetObjectItem(cjson_test, "weight");

    printf("name: %s\n", cjson_name->valuestring);
    printf("age:%d\n", cjson_age->valueint);
    printf("weight:%.1f\n", cjson_weight->valuedouble);

    /* 解析嵌套json数据 */
    cjson_address = cJSON_GetObjectItem(cjson_test, "address");
    cjson_address_country = cJSON_GetObjectItem(cjson_address, "country");
    cjson_address_zipcode = cJSON_GetObjectItem(cjson_address, "zip-code");
    printf("address-country:%s\naddress-zipcode:%d\n", cjson_address_country->valuestring, cjson_address_zipcode->valueint);

    /* 解析数组 */
    cjson_skill = cJSON_GetObjectItem(cjson_test, "skill");
    skill_array_size = cJSON_GetArraySize(cjson_skill);
    printf("skill:[");
    for(i = 0; i < skill_array_size; i++)
    {
        cjson_skill_item = cJSON_GetArrayItem(cjson_skill, i);
        printf("%s,", cjson_skill_item->valuestring);
    }
    printf("\b]\n");

    /* 解析布尔型数据 */
    cjson_student = cJSON_GetObjectItem(cjson_test, "student");
    if(cjson_student->valueint == 0)
    {
        printf("student: false\n");
    }
    else
    {
        printf("student:error\n");
    }
    
    return 0;
}

123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778
```

编译：

```c
gcc cJSON.c example2.c -o example2.exe

1
```

运行结果如图：

![运行结果](https://i-blog.csdnimg.cn/blog_migrate/e5604a7d5b928cf1deeefcbcf2c3b1c0.png)

### 0.4.3 注意事项

在本示例中，因为我提前知道数据的类型，比如字符型或者浮点型，所以我直接使用指针指向对应的数据域提取，**在实际使用时，如果提前不确定数据类型，应该先判断type的值，确定数据类型，再从对应的数据域中提取数据**。

## 0.5 5\. cJSON使用过程中的内存问题

### 0.5.1 内存及时释放

cJSON的所有操作都是基于链表的，所以cJSON在使用过程中**大量的使用`malloc`从堆中分配动态内存的，所以在使用完之后，应当及时调用下面的函数，清空cJSON指针所指向的内存**，该函数也可用于删除某一条数据：

```c
(void) cJSON_Delete(cJSON *item);

1
```

> 注意：该函数删除一条JSON数据时，如果有嵌套，会连带删除。

### 0.5.2 内存钩子

cJSON在支持自定义malloc函数和free函数，方法如下：

- ① 使用`cJSON_Hooks`来连接自定义malloc函数和free函数：

```c
typedef struct cJSON_Hooks
{
      /* malloc/free are CDECL on Windows regardless of the default calling convention of the compiler, so ensure the hooks allow passing those functions directly. */
      void *(CJSON_CDECL *malloc_fn)(size_t sz);
      void (CJSON_CDECL *free_fn)(void *ptr);
} cJSON_Hooks;

123456
```

- ② 初始化钩子cJSON\_Hooks

```c
(void) cJSON_InitHooks(cJSON_Hooks* hooks);

1
```