灵活的静态存储控制器，stm32内置的外设，主要用于驱动SRAM和Flash类型的存储器
可以将外接的存储单元映射到stm32内部的寻址空间中
可以模拟8080协议驱动LCD屏
# 1 主要信号线：
片选
地址总线
掩码信号(两条)
数据总线
读使能
写使能
# 2 流程：
1. 配置时钟 GPIO口
2. 配置FSMC（**读写时序**、要映射bank区域，存储器类型，数据宽度、写使能 ）
3. 使能 FSMC_Bank1_NORSRAM4 
```c
/**
* @brief  FSMC NOR/SRAM Init structure definition
*/
typedef struct
{
uint32_t FSMC_Bank;                /*设置要控制的Bank区域 */
uint32_t FSMC_DataAddressMux;      /*设置地址总线与数据总线是否复用 */
uint32_t FSMC_MemoryType;          /*设置存储器的类型 */
uint32_t FSMC_MemoryDataWidth;     /*设置存储器的数据宽度*/
uint32_t FSMC_BurstAccessMode; /*设置是否支持突发访问模式，只支持同步类型的存储器 */
uint32_t FSMC_AsynchronousWait;     /*设置是否使能在同步传输时的等待信号，*/
uint32_t FSMC_WaitSignalPolarity;  /*设置等待信号的极性*/
uint32_t FSMC_WrapMode;            /*设置是否支持对齐的突发模式 */
uint32_t FSMC_WaitSignalActive; /*配置等待信号在等待前有效还是等待期间有效 */
uint32_t FSMC_WriteOperation;      /*设置是否写使能 */
uint32_t FSMC_WaitSignal;          /*设置是否使能等待状态插入 */
uint32_t FSMC_ExtendedMode;        /*设置是否使能扩展模式 置为1时序有可以设置A、B、C、D模式 置为0可以设置1、2模式*/
uint32_t FSMC_WriteBurst;          /*设置是否使能写突发操作*/
/*当不使用扩展模式时，本参数用于配置读写时序，否则用于配置读时序*/
FSMC_NORSRAMTimingInitTypeDef* FSMC_ReadWriteTimingStruct;
/*当使用扩展模式时，本参数用于配置写时序*/
FSMC_NORSRAMTimingInitTypeDef* FSMC_WriteTimingStruct;
}FSMC_NORSRAMInitTypeDef;
```
# 3 时序结构体赋值

```c
typedef struct
{
  //地址建立时间，不同的存储器芯片可能有不同的时序要求，ADDSET时间是其中一个关键参数。它确保在选通信号（如读/写使能信号NOE/NWE）激活之前，地址总线上的地址已经稳定，从而满足存储器芯片的时序需求。
  uint32_t FSMC_AddressSetupTime;
  //
  uint32_t FSMC_AddressHoldTime;

  uint32_t FSMC_DataSetupTime;
  
  uint32_t FSMC_BusTurnAroundDuration;

  uint32_t FSMC_CLKDivision;

  uint32_t FSMC_DataLatency;
  //时序模式，模式1为SRAM，模式2为nor flash
  uint32_t FSMC_AccessMode;

}FSMC_NORSRAMTimingInitTypeDef;
```

模式1
读时序：
1. 地址发出，要注意器件识别到地址的时间,同时读操作线置为低电平
2. 读操作线置为低电平后，ADDSET+1+DATAST+1后，FSMC会采集数据线上的数据，要注意采集的时间不能比数据生成的时间要快
3. ADDSET+1 +DATAST+1+ 2 HCLK cycles（周期），不能小于器件规定的最小周期

$1 / HCLK *（ADDSET+1+DATAST+1+2）>55ns$
$1/HCLK* (ADDSET+1+DARAST+1)>55ns$
$ADDSET+1+DARAST+1>3.96$
写时序：
1. 地址发出，要注意器件识别到地址的时间。（ADDSET+1）后写操作线置为低电平
2. 写操作置为低电平，同时数据线输入数据。（DATAST）时间后写操作线置为高电平，1个周期后结束写时序。要注意数据保持时间要比器件要求的数据保持时间要长
$1/HCLK* (ADDSET+1+DARAST+1)>55ns$
$1/HCLK* (ADDSET+1)>0ns$
$1/HCLK* (ADDSET+1+DATASRT+1)>45ns$   tPWB>45   高字节与低字节线时序要求
$1/HCLK* (DATAST)>40ns$   tHZWE 写操作到数据要持续的时间  DATAST>72\*40/1000=2.88   
模式A
读时序：
1. 地址发出ADDSET+1时间后，读操作线置为低电平，要注意器件识别到地址的时间，置为低电平时要保障器件识别到及地址 1 / HCLK *（ADDSET+1+DATAST+1）>55ns
2. 读操作线置为低电平后，DATAST+1后，FSMC会采集数据线上的数据，要注意采集的时间不能比数据生成的时间要快  1/HCLK* (DATAST+1)>25ns
3. ADDSET+1 +DATAST+1+ 2 HCLK cycles（周期），不能小于器件规定的最小周期
 $1 / HCLK *（ADDSET+1+DATAST+1）>55ns$
$1 / HCLK *（ADDSET+1+DATAST+1+2）>55ns$
$1/HCLK* (DATAST+1)>25ns$
1/HCLK

1000/72

ADDSET+1>0
ADDSET+1+DATAST+1>55
45+0>DATAST+1>45-25