> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/m0_46577050/article/details/137633272)

#### 关于 MCU [产品开发](https://edu.csdn.net/cloud/pm_summit?utm_source=blogglc&spm=1001.2101.3001.7020)参数存储的几种方案

*   [Chapter1 关于 MCU 产品开发参数存储的几种方案](#Chapter1__MCU_6)
*   [Chapter2 单片机参数处理 [保存与读取]](#Chapter2___22)
*   [Chapter3 嵌入式设备参数存储技巧](#Chapter3___112)
*   [Chapter4 STM32 硬件 I2C 的一点心得 (AT24C32C 和 AT24C64C)](#Chapter4__STM32I2CAT24C32CAT24C64C_350)

Chapter1 关于 MCU 产品开发参数存储的几种方案
-----------------------------

[原文链接](https://blog.csdn.net/morixinguan/article/details/113209756?ops_request_misc=&request_id=&biz_id=102&utm_term=%E5%8D%95%E7%89%87%E6%9C%BA%20%E5%8F%82%E6%95%B0%E4%BF%9D%E5%AD%98&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-5-113209756.142%5Ev100%5Epc_search_result_base7&spm=1018.2226.3001.4187)

在工作中，几乎所有参与产品开发的产品都将实现参数存储功能。

通常，参数存储将使用以下存储介质，例如：eeprom，[spi](https://so.csdn.net/so/search?q=spi&spm=1001.2101.3001.7020) 闪存，nand 闪存，SD 卡等，至于如何存储，现在有很多种类。

1. 使用 eeprom（以 at24c02 为例）定义结构，然后定义两个结构变量，一个用于读取参数，一个用于立即写入修改的参数。

参考：2. 使用 spi_flash（以 w25q64 为例）方法 1 与使用 eeprom 方法相同。

方法 2 使用文件系统并创建一个 ini 文件来获取参数。

Chapter2 单片机参数处理 [保存与读取]
------------------------

原文链接：[https://blog.csdn.net/WangSanHuai2010/article/details/6988583](https://blog.csdn.net/WangSanHuai2010/article/details/6988583)

```
/*------------------------------------------------------------
 Func: 加载参数到系统
 Time: 2011-11-13
 Ver.: V1.0
 Note:
------------------------------------------------------------*/
void WFS_LoadParams(uint16 Addr,uint16 *Buffer,uint16 Length)
{
	Addr<<=1;Length<<=1;
	EEPROM_Read(Addr+2,(uint8 *)Buffer,Length);
}
```

参数按以上方法加载到内存，注意参数的起始地址为 2，这是因为前两个字节区域要用来做校验用。

```
/*------------------------------------------------------------
 Func: 保存参数
 Time: 2011-11-13
 Ver.: V1.0
 Note:
------------------------------------------------------------*/
void WFS_SaveParams(uint16 Addr,uint16 *Buffer,uint16 Length)
{
	Addr<<=1;Length<<=1;
	EEPROM_Write(Addr+2,(uint8 *)Buffer,Length);
}
```

以上方法保存参数到 EEPROM 中，实际上与 Load 方法一一对应。

```
/*------------------------------------------------------------
 Func: 参数系统初始化
 Time: 2011-11-13
 Ver.: V1.0
 Note:
------------------------------------------------------------*/
uint8 WFS_InitParams(void *DefaultValues,uint16 Length)
{
	uint16 D;
	EEPROM_Read(0,(uint8 *)(&D),2);
	if(D!=0x55AA){
		D=0x55AA;
		EEPROM_Write(0,(uint8 *)(&D),2);
		EEPROM_Write(2,(uint8 *)DefaultValues,Length);
		return 0xFF;
	}
	return 0x00;
}
```

参数的初始化方法，首先读取 EEPROM 的 0 位置处的数据，判断是否为 0x55AA 合法标志，若不是 0x55AA，则说明参数区为首次使用，需要进行初始化默认参数填充，于是将 DefaultValues 所指的默认值填入 EEPROM 中，并设置 0x55AA 标志，以后每次上电便会检测到参数的合法性。

以下为使用示例，存储了地址码，波特率，数据位，停止位四个参数，以及一个 18 字的数组。

```
const uint16 WFS_ParmasValue_Default[]=
{
	1,9600,8,1,
	0,0,0,0,0,0,
	0,0,0,0,0,0,
	0,0,0,0,0,0,
};
```

以下为参数进行初始化并加载到内存：

```
WFS_InitParams(WFS_ParmasValue_Default,sizeof(WFS_ParmasValue_Default));
WFS_LoadParams(0,&DevAddr,1);
WFS_LoadParams(1,&BaudRate,1);
WFS_LoadParams(2,&DataLength,1);
WFS_LoadParams(3,&StopBits,1);
WFS_LoadParams(4,Array,18);
```

以下为参数修改后进行保存：

```
BaudRate=115200;
StopBits=2;
WFS_SaveParams(1,&BaudRate,1);
WFS_LoadParams(3,&StopBits,1);
```

Chapter3 嵌入式设备参数存储技巧
--------------------

[原文链接](https://blog.csdn.net/qq_32348883/article/details/123761781?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_utm_term~default-0-123761781-blog-53081657.235%5Ev43%5Epc_blog_bottom_relevance_base9&spm=1001.2101.3001.4242.1&utm_relevant_index=3)

1、前言  
想必做嵌入式产品开发都遇到过设备需要保存参数，常用的方式就是按照结构体的方式管理参数，保存时将整个结构体数据保存在 Flash 中，方便下次读取。

1.1、目的  
本文时分析嵌入式 / 单片机中参数保存的几种方式的优点和缺点（仅针对单片机 / 嵌入式开发而言），同时针对以结构体的方式解决一些弊端问题（重点在第 3 节）。

2、参数保存格式  
2.1、结构体格式  
该方式是嵌入式 / 单片机中开发最常用的，将所有的系统参数通过结构体的方式定义，然后保存数据，介绍一下该方式的优缺点。

储存方式：二进制 bin 文件格式  
1  
优点：

管理简单：无需额外的代码直接就能很方便的管理参数  
内存最小：通过结构体的形式保存在 Flash 中，占用内存最小  
缺点：

1. 扩展性差：

从产品角度来说，产品需要升级，若是涉及增加参数，则升级后参数通常无法校验通过（通常包含长度校验等），导致参数被恢复默认  
若是每个模块都存在自己的独有结构体参数定义，删除 / 新增时势必影响到其他的，导致设备升级后参数错乱（结构体中的变量地址在 bin 文件中是固定的）  
2. 阅读性差：  
若参数需要导出，bin 文件没有可读性

改进措施：  
结构体增加预留定义，若之后需要新增参数，则在预留空间新增即可，能在一定程度上解决扩展性差的问题，即新增不影响原有的结构体大小和其他成员变量的位置，删除恢复成预留即可。

为啥说只能在一定程度上解决该问题，因为之后的升级某些模块可能很长时间或者从不需要增加新的参数，这种势必就会造成内存的无效占用，或者有些模块频繁增加参数导致预留大小不够等问题，只能在前期设计时多加思考预留的分配情况（毕竟内存只有那么大）

```
/*****************************
           改进之前
*****************************/
 
typedef struct
{
    uint8_t testParam;
    uint8_t testParam2;
} TestParam_t;    /* 某模块参数 */
 
typedef struct
{
    uint8_t testParam;
    uint8_t testParam2;
    TestParam_t tTestParam;
} SystemParam_t; /* 系统参数 */
 
/*****************************
           改进之后
*****************************/
 
typedef struct
{
    uint8_t testParam;
    uint8_t testParam2;
    uint8_t reserve[6];    // 预留
} TestParam_t;    /* 某模块参数 */
 
typedef struct
{
    uint8_t testParam;
    uint8_t testParam2;
    TestParam_t tTestParam;
    uint8_t reserve[50];   // 预留
} SystemParam_t; /* 系统参数 */
```

2.2、JSON 格式  
最近 Json 格式很是流行使用，特别是数据交换中用的很多，但是它也可以用来保存参数使用，JSON 的是 “{键：值}” 的方式。

储存方式：字符串格式，即文本的形式  
1  
优点：

扩展性好：由于 Json 的格式，找到对应键值（一般都是该变量的标识），就能找到对应的值  
阅读性好：有标识所以导出参数文件通过普通的文本文件打开都能看懂  
缺点：

管理相对复杂：没有结构体那么简单，不熟还得先学习 JSON 的写法  
内存占用较大：内容不只有值，而且都按照字符串的形式保存的  
使用相关困难：需要解析，C 语言虽然有[开源](https://edu.csdn.net/cloud/pm_summit?utm_source=blogglc&spm=1001.2101.3001.7020)库，但是由于语言性质使用不方便，C++ 反而使用简单

```
{
    "SYS":
    {
        "testParam" : 2,
        "testParam2" : 5,
        "tTestParam":
        {
            "testParam" : 2,
            "testParam2" : 5
        }
    }
}
 
//压缩字符串为：
{"SYS":{"testParam":2,"testParam2":5,"tTestParam":{"testParam":2,"testParam2":5}}}
```

2.3、键值格式  
和上述的 JSON 格式很类似，都是键值对的格式，但是比 JSON 简单

储存方式：字符串格式，即文本的形式  
1  
优点：

扩展性好：找到对应键值（一般都是该变量的标识），就能找到对应的值  
阅读性好：有标识所以导出参数文件通过普通的文本文件打开都能看懂  
缺点：

内存占用较大：内容不只有值，而且都按照字符串的形式保存的  
使用稍微困难：需要简单解析处理  
管理不变：不方便按照一定的规则管理各模块的参数

```
testParam=2
testParam2=5
T_testParam=2
T_testParam2=5
```

2.4 其他  
还有其他，如 xml (类似 JSON) 等，就不多介绍了

3、编译器检查结构体的大小和成员变量的偏移  
在第 2 节中介绍了关于参数保存的三种方式，但是对于嵌入式单片机开发而言，Flash 大小不富裕，所以通常都是通过二进制的形式保存的，所以这节重点解决结构体管理保存参数的扩展性问题。

先说一下痛点（虽然对扩展性问题做了改进措施，除了前面讲到的问题，还有其他痛点，虽不算问题，但是一旦出现往往最要命）

在原来的预留空间中新增参数，要确保新增后结构体的大小不变，否则会导致后面的其他参数偏移，最后升级设备后参数出现[异常](https://marketing.csdn.net/p/3127db09a98e0723b83b2914d9256174?pId=2782&utm_source=glcblog&spm=1001.2101.3001.7020)（如果客户升级那就是要命啊）  
确保第一点，就必须在每次新增参数都要计算检查一下结构体的大小有没有发生变化，而且有没有对结构体中的其他成员也产生影响  
每次新增参数，手动计算和校验 99% 可以检查出来，但是人总有粗心的时候（加班多了，状态不好…），且结构体存在填充，一不留神就以为没问题，提交代码，出版本（测试不一定能发现），给客户，升级后异常，客户投诉、扣工资（难啊…）  
遇到这种问题后：难道编译器就不能在编译的时候检查这个大小或者结构体成员的偏移吗，每次手动计算校验好麻烦啊，一不留神还容易算错 # _ #

按照正常情况，编译器可不知道你写的结构体大小和你想要的多大，所以检查不出来（天啊，崩溃了 0.0…）

别急，有另类的方式可以达到这种功能，在编译时让编译器为你检查，而且准确性 100%（当然，这个添加新参数时你还得简单根据新增的参数大小减少预留的大小，这个是必须要的）

见代码：

```
/**
  * @brief 检查结构体大小是否符合
  *        在编译时会进行检查
  * @param type 结构体类型
  * @param size 结构体检查大小
  */
#define TYPE_CHECK_SIZE(type, size) extern int sizeof_##type##_is_error [!!(sizeof(type)==(size_t)(size)) - 1]
 
/**
  * @brief 结构体成员
  * @param type   结构体类型
  * @param member 成员变量
  */
#define TYPE_MEMBER(type, member) (((type *)0)->member)
 
 
/**
  * @brief 检查结构体成员大小是否符合
  *        在编译时会进行检查
  * @param type 结构体类型
  * @param member 结构体类型
  * @param size 结构体检查大小
  */
#define TYPE_MEMBER_CHECK_SIZE(type, member, size) extern int sizeof_##type##_##member##_is_error \
    [!!(sizeof(TYPE_MEMBER(type, member))==(size_t)(size)) - 1]
 
 
/**
  * @brief 检查结构体中结构体成员大小是否符合
  *        在编译时会进行检查
  * @param type 结构体类型
  * @param member 结构体类型
  * @param size 结构体检查大小
  */
#define TYPE_CHILDTYPE_MEMBER_CHECK_SIZE(type, childtype, member, size) extern int sizeof_##type##_##childtype##_##member##_is_error \
    [!!(sizeof(TYPE_MEMBER(type, childtype.member))==(size_t)(size)) - 1]
 
 
/**
  * @brief 检查结构体成员偏移位置是否符合
  *        在编译时会进行检查
  * @param type 结构体类型
  * @param member 结构体成员
  * @param value 成员偏移
  */
#define TYPE_MEMBER_CHECK_OFFSET(type, member, value) \
         extern int offset_of_##member##_in_##type##_is_error \
        [!!(__builtin_offsetof(type, member)==((size_t)(value))) - 1]
 
 
/**
  * @brief 检查结构体成员偏移位置是否符合
  *        在编译时会进行检查
  * @param type 结构体类型
  * @param member 结构体成员
  * @param value 成员偏移
  */
#define TYPE_CHILDTYPE_MEMBER_CHECK_OFFSET(type, childtype, member, value) \
         extern int offset_of_##member##_in_##type##_##childtype##_is_error \
        [!!(__builtin_offsetof(type, childtype.member)==((size_t)(value))) - 1]
```

通过以上代码，就能解决这个问题，这个写法只占用文本大小，编译后不占内存！！！

用法：

```
typedef struct
{
    uint8_t testParam;
    uint8_t testParam2;
    uint8_t reserve[6];    // 预留
} TestParam_t;    /* 某模块参数 */
 
TYPE_CHECK_SIZE(TestParam_t, 8); // 检查结构体的大小是否符合预期
 
typedef struct
{
    uint8_t testParam;
    uint8_t testParam2;
    TestParam_t tTestParam;
    uint8_t reserve[54];   // 预留
} SystemParam_t; /* 系统参数 */
 
TYPE_CHECK_SIZE(SystemParam_t, 64); // 检查结构体的大小是否符合预期
TYPE_MEMBER_CHECK_OFFSET(SystemParam_t, tTestParam, 2); // 检查结构体成员tTestParam偏移是否符合预期
```

假设新增了参数，预留写错了，导致结构体的大小不符合，则编译时报错，且提示内容也能快速定位问题。  
![[../../_resources/关于 MCU 产品开发参数存储的几种方案_mcu 开发 eeprom 保存策略/60a1d975372bf6d6140afb5d1721fd2f_MD5.png]]

Chapter4 STM32 硬件 I2C 的一点心得 (AT24C32C 和 AT24C64C)
-------------------------------------------------

原文链接：[https://blog.csdn.net/whitefish520/article/details/110070972](https://blog.csdn.net/whitefish520/article/details/110070972)

从设备读写  
一般的 EEPROM，像 AT24C02 这种小容量的设备，地址都只需要 8 位，页大小一般是 16 字节一个页  
而像 AT24C32C、AT24C64C 这种 32K、64K 字节的大容量 EEPROM，8 位地址是不够的，使用了 16 位地址，页大小在这两个器件中也变成了 32 字节  
正是由于容量的不同，导致代码上需要做差异化处理，才能正确读取 EEPROM 芯片

以下代码可以参考，写的时候无法跨页，因此写大量数据的时候，只能一页一页的写，两次写之间保证 5MS 的间隔  
读没有跨页的影响，可以一次性把全部数据读出来，但是要注意，读和写之间，是要有 5MS 的间隔的，否则读不到数据。也就是说每次写完，延迟 5MS，就能保证后续的程序没有问题。

```
//#define I2C_MEMADD_SIZE		I2C_MEMADD_SIZE_8BIT			//小容量EEPROM芯片8位地址用此参数
#define I2C_MEMADD_SIZE		I2C_MEMADD_SIZE_16BIT		//大容量EEPROM芯片16位地址用此参数
#define ADDR_AT24C02_Write 0xA0		//EEPROM I2C写地址
#define ADDR_AT24C02_Read 0xA0+1	//EEPROM I2C读地址

typedef enum SYS_PARA_ENUM {
	LOCAL_IP = 0,
	UDP_LOCAL_PORT,
	UDP_PC_PORT,
	NETMASK,
	GATEWAY,
	SERVER_IP,
	SERVER_PORT,
	VERSION,
	SN_NUM,	//16字节
	SENSOR_TYPE = SN_NUM + 4,
	SENSOR_DATA_TYPE,
	SENSOR_INTERVAL,
	SYS_PARA_MAX,		//end
}sys_para_e;

//请注意，为了方便flash读写操作，此处的每一项均设为uint32_t类型
//如果不为uint32_t类型，则flash_write函数将出现错误
typedef struct SYS_PARA_TYPE {
	uint32_t local_ip;			//本机IP，大端模式
	uint32_t udp_local_port;	//本机端口号
	uint32_t udp_pc_port;		//PC端口号
	uint32_t netmask;			//本机子网掩码，大端模式
	uint32_t gateway;			//本机网关，大端模式
	uint32_t server_ip;			//服务器IP，大端模式
	uint32_t server_port;		//服务器端口号
	uint32_t version;			//stm32软件版本号
	uint32_t sn_num[4];			//SN号	5 6 7 8
	uint32_t sensor_type;		//传感器类型 9
	uint32_t sensor_data_type;	//传感器数据类型
	uint32_t sensor_interval;	//传感器采集时间间隔
}sys_para_t;

/* -----------------------------------------------------------------------------
函数名：  	i2c_write
作者：    	glx
日期：    	2020-11-10
功能：    	数据写入eeprom
输入参数：	pData：数据指针
返回值：  	类型：HAL_StatusTypeDef
			HAL_OK：操作成功
			HAL_ERROR：操作失败
修改记录：
------------------------------------------------------------------------------*/
HAL_StatusTypeDef i2c_write(void *pData)
{
	HAL_StatusTypeDef ret = HAL_ERROR;
	uint8_t i, page, pageSize;
	uint8_t *p = (uint8_t *)pData;
	if(pData != NULL)
	{
		if(I2C_MEMADD_SIZE == I2C_MEMADD_SIZE_8BIT)
		{
			pageSize = 16;			//一页16字节，不能跨页写
			page = SYS_PARA_MAX/4;
		}
		else if(I2C_MEMADD_SIZE == I2C_MEMADD_SIZE_16BIT)
		{
			pageSize = 32;			//一页32字节，不能跨页写
			page = SYS_PARA_MAX/8;
		}
		//写完整的页
		for(i = 0; i<page; i++)
		{
			ret = HAL_I2C_Mem_Write(&hi2c1, ADDR_AT24C02_Write, i*pageSize, I2C_MEMADD_SIZE, p, pageSize, 100);
			p += pageSize;
			if(ret != HAL_OK)
			{
				printf("I2C_Write Sys Para write error\r\n");
				return HAL_ERROR;
			}
			HAL_Delay(5);
		}
		//写残缺的页
		if(SYS_PARA_MAX > (page*4))	
		{
			ret = HAL_I2C_Mem_Write(&hi2c1, ADDR_AT24C02_Write, i*pageSize, I2C_MEMADD_SIZE, p, 4*SYS_PARA_MAX-pageSize*page, 100);
			if(ret != HAL_OK)
			{
				printf("I2C_Write Sys Para write error\r\n");
				return HAL_ERROR;
			}
			HAL_Delay(5);
		}
	}
	else
	{
		printf("I2C_Write pData NULL\r\n");
		return HAL_ERROR;
	}
	return HAL_OK;
}

/* -----------------------------------------------------------------------------
函数名：  	i2c_read
作者：    	glx
日期：    	2020-11-10
功能：    	读取eeprom数据
输入参数：	pData：数据指针
返回值：  	类型：HAL_StatusTypeDef
			HAL_OK：操作成功
			HAL_ERROR：操作失败
修改记录：
------------------------------------------------------------------------------*/
HAL_StatusTypeDef i2c_read(void *pData)
{
	HAL_StatusTypeDef ret = HAL_ERROR;
	if(pData != NULL)
	{
		ret = HAL_I2C_Mem_Read(&hi2c1, ADDR_AT24C02_Read, 0, I2C_MEMADD_SIZE, (uint8_t *)pData, SYS_PARA_MAX*4, 100);
		if(ret != HAL_OK)
		{
			printf("I2C_Read Sys Para read error\r\n");
			return HAL_ERROR;
		}		
	}
	else
	{
		printf("I2C_Read pData NULL\r\n");
		return HAL_ERROR;
	}
	return HAL_OK;
}
```

读写的代码

```
//用于保存系统参数
sys_para_t sys_para = {0};	
//用于保存系统默认参数
sys_para_t default_para = {
	.local_ip = 107<<24 | 10<<16 | 168<<8 | 192,
	.udp_local_port = 18080,
	.udp_pc_port = 18081,
	.netmask = 0<<24 | 255<<16 | 255<<8 | 255,
	.gateway = 1<<24 | 10<<16 | 168<<8 | 192,
	.server_ip = 9<<24 | 10<<16 | 168<<8 | 192,
	.server_port = 18082,
	.version = 20201124,
	.sn_num[0] = 0xffffffff,
	.sn_num[1] = 0xffffffff,
	.sn_num[2] = 0xffffffff,
	.sn_num[3] = 0xffffffff,
	.sensor_type = SENSOR_DOOR,
	.sensor_data_type = SENSOR_DATA_GPIO,
	.sensor_interval = 1000,
};

i2c_read(&sys_para);
i2c_write(&default_para);
```