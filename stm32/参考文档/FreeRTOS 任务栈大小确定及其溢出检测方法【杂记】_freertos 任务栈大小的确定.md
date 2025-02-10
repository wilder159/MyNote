> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/stephenbruce/article/details/133444362)

不管是裸机编程还是 [RTOS](https://so.csdn.net/so/search?q=RTOS&spm=1001.2101.3001.7020 "RTOS") 编程，栈的分配大小都非常重要。 局部变量，[函数调用](https://so.csdn.net/so/search?q=%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8&spm=1001.2101.3001.7020)时的现场保护和返回地址，函数的形参，进入中断函数前和中断嵌套等都需要栈空间，栈空间定义小了会造成系统崩溃。

> 一、裸机情况下，用户可以在这里配置栈大小
> --------------------

![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/4fb1510a823a80161321d8db32354713_MD5.png]]

         裸机情况下，局部变量，函数调用时的现场保护和返回地址，函数的形参，进入[中断函数](https://so.csdn.net/so/search?q=%E4%B8%AD%E6%96%AD%E5%87%BD%E6%95%B0&spm=1001.2101.3001.7020 "中断函数")前和中断嵌套等都使用的是上述栈空间（0x0000 0800）。

> 二、[FreeRTOS](https://so.csdn.net/so/search?q=FreeRTOS&spm=1001.2101.3001.7020 "FreeRTOS") 情况下，任务栈是从 FreeRTOSConfig.h 文件中定义的 HEAP 空间申请
> -------------------------------------------------------------------------------------------------------------------------------------

### 0.1.1 1、**FreeRTOS 情况下：任务栈设置** 

![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/c5c4f0310ab2d9baca92e75f69b70c17_MD5.png]]

         为什么是堆中的？因为我们采用的就是动态创建任务的方式。如果静态创建，就和我们自己开辟的空间有关，通常静态创建任务用数组作为容器，但是通常静态创建的方式我们都不使用。 

> FreeRTOS 的情况下，任务栈是从 FreeRTOSConfig.h 文件中定义的 HEAP 空间申请。

### 0.1.2  2、**FreeRTOS 情况下：系统栈设置**

        在 **FreeRTOS** 情况下，在 KEIL 中设置的栈大小有了一个新的名字叫**系统栈空间**（在 **FreeRTOS 情况下，任务栈是不使用这里的空间的**，任务栈使用的是 FreeRTOSConfig.h 文件中定义的 HEAP 空间）。

        任务栈不使用下图所示的栈空间（**系统栈空间**），谁使用这里的栈空间呢？答案就在**中断函数和中断嵌套**。   
![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/3c94d67b7cddd5888fc27b6e0395bcc7_MD5.png]]

> **在 FreeRTOS 情况下，中断函数和中断嵌套使用系统栈空间，任务栈使用堆 HEAP 空间。**

         由于 Cortex-M3 和 M4 内核具有双堆栈指针，MSP 主堆栈指针和 PSP 进程堆栈指针，或者叫 PSP 任务堆栈指针也是可以的。**在 FreeRTOS 操作系统中，主堆栈指针 MSP 是给系统栈空间使用的，进程堆栈指针 PSP 是给任务栈使用的。**

        也就是说，在 FreeRTOS 任务中，所有任务栈空间的使用都是通过 PSP 指针进行指向的。中断服务函数以及可能发生的中断嵌套都是用的 MSP 指针。这个知识点要记住它，当前可以不知道这是为什么，但是一定要记住。

        **实际应用中系统栈空间分配多大，主要是看可能发生的中断嵌套层数**，下面我们就按照最坏执行情况进行考虑，所有的寄存器都需要入栈，此时分为两种情况：

        （1）、64 字节

        对于 Cortex-M3 内核和未使用 FPU（浮点运算单元）功能的 Cortex-M4 内核在发生中断时需  
要将 16 个通用寄存器全部入栈，每个寄存器占用 4 个字节，也就是 16*4 = 64 字节的空间。  
可能发生几次中断嵌套就是要 64 乘以几即可。 当然，这种是最坏执行情况，也就是所有的寄存  
器都入栈。

        （注：任务执行的过程中发生中断的话，有 8 个寄存器是自动入栈的，这个栈是任务栈，进入中断以后其余寄存器入栈以及发生中断嵌套都是用的系统栈）

        （2）、200 字节

        对于具有 FPU（浮点运算单元）功能的 Cortex-M4 内核，如果在任务中进行了浮点运算，那么在发生中断的时候除了 16 个通用寄存器需要入栈，还有 34 个浮点寄存器也是要入栈的，也就是 (16+34)*4 = 200 字节的空间。当然，这种是最坏执行情况，也就是所有的寄存器都入栈。  
        （注：任务执行的过程中发送中断的话，有 8 个通用寄存器和 18 个浮点寄存器是自动入栈的，这个栈是任务栈，进入中断以后其余通用寄存器和浮点寄存器入栈以及发生中断嵌套都是用的系统栈）

> **void vTaskStartScheduler(void);**  
> 函数描述：  
>         函数 vTaskStartScheduler 用于启动 FreeRTOS 调度器，即启动 FreeRTOS 的多任务执行。  
> 使用这个函数要注意以下几个问题：  
> 1. 空闲任务和可选的定时器任务是在调用这个函数后自动创建的。  
> 2. 正常情况下这个函数是不会返回的，运行到这里极有可能是用于定时器任务或者空闲任务的 heap 空间不足造成创建失败，此时需要加大 FreeRTOSConfig.h 文件中定义的 heap 大小：  
> **#define configTOTAL_HEAP_SIZE (( size_t) ( 17 * 1024 ) )**

![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/8a1027f4cfc0c67c2efb15ade1a089a1_MD5.png]]

> 三、任务栈大小的确定 
> -----------

        在基于 [RTOS](https://so.csdn.net/so/search?q=RTOS&spm=1001.2101.3001.7020) 的应用设计中，每个任务都需要自己的栈空间，应用不同，每个任务需要的栈大小也是不同的。 将如下的几个选项简单的累加就可以得到一个粗略的栈大小：

### 0.1.3 1、函数的嵌套调用

        针对每一级函数用到栈空间的有如下四项：

> （1）、**函数局部变量**。
> 
> （2）、**函数形参**，一般情况下函数的形参是直接使用的 CPU 寄存器，不需要使用栈空间，但是这个函数中如果还嵌套了一个函数的话，这个存储了函数形参的 CPU 寄存器内容是要入栈的。 所以建议大家也把这部分算在栈大小中。
> 
> （3）、**函数返回地址**，针对 M3 和 M4 内核的 MCU，一般函数的返回地址是专门保存到 LR（LinkRegister）寄存器里面的，如果这个函数里面还调用了一个函数的话，这个存储了函数返回地址的 LR 寄存器内容是要入栈的。 所以建议大家也把这部分算在栈大小中。
> 
> （4）、**函数内部的状态保存操作**也需要额外的栈空间。

### 0.1.4 2、任务切换

        任务切换时所有的寄存器都需要入栈，对于带 FPU 浮点处理单元的 M4 内核 MCU 来说，  
FPU 寄存器也是需要入栈的。

        针对 M3 内核和 M4 内核的 MCU 来说，在任务执行过程中，如果发生中断：

> （1）、M3 内核的 MCU 有 8 个寄存器是自动入栈的，这个栈是任务栈，进入中断以后其余寄存器入栈以及发生中断嵌套都是用的系统栈。
> 
> （2）、M4 内核的 MCU 有 8 个通用寄存器和 18 个浮点寄存器是自动入栈的，这个栈是任务栈，进入中断以后其余通用寄存器和浮点寄存器入栈以及发生中断嵌套都是用的系统栈。
> 
> （3）、进入中断以后使用的局部变量以及可能发生的中断嵌套都是用的系统栈，这点要注意。

        实际应用中将这些都加起来是一件非常麻烦的工作，上面这些栈空间加起来的总和只是栈的最小需求，实际分配的栈大小可以在最小栈需求的基础上乘以一个安全系数，一般取 1.5～2。上面的计算是我们用户可以确定的栈大小，项目应用中还存在无法确定的栈大小，比如调用 printf 函数就很难确定实际的栈消耗。  
        又比如通过函数指针实现函数的间接调用，因为函数指针不是固定的指向一个函数进行调用，而是根据不同的程序设计可以指向不同的函数，使得栈大小的计算变得比较麻烦。  
        另外还要注意一点，建议不要编写递归代码，因为我们不知道递归的层数，栈的大小也是不好确定的。

### 0.1.5  3、函数栈大小确定

        函数的栈大小计算起来是比较麻烦的， 那么有没有简单的办法来计算呢？ 有的，一般 IDE 开发环境都有这样的功能，比如 MDK 会生成一个 htm 文件，通过这个文件用户可以知道每个被调用函数的最大栈需求以及各个函数之间的调用关系。但是 MDK 无法确定通过函数指针实现函数调用时的栈需求。另外，发生中断或中断嵌套时的现场保护需要的栈空间也不会统计。

> 四、什么是栈溢出
> --------

        前面为大家讲解了如何确定任务栈的大小，那什么又是栈溢出呢？简单的说就是用户分配的栈空间不够用了，溢出了。 下面我们举一个简单的实例，栈生长方向从高地址向低地址生长（M4 和 M3 是这种方式）。

![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/6303f81f11ebccbd9bcdef9829c28c59_MD5.png]]

![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/a37d2f0463e78c206e9e3e1a04267535_MD5.png]]![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/aa92c13f65013d7b602b96ad3873f623_MD5.png]]

>  五、栈溢出如何检测
> ----------

 FreeRTOS 提供了两种栈溢出检测机制，这两种检测都是在任务切换时才会进行：

### 0.1.6 1、方法一 

        在任务切换时检测任务栈指针是否过界了，如果过界了，在任务切换的时候会触发栈溢出钩子函数。

```c
void vApplicationStackOverflowHook( TaskHandle_t xTask,
signed char *pcTaskName );
```

        用户可以在钩子函数里面做一些处理。这种方法不能保证所有的栈溢出都能检测到。比如任务在执行的过程中出现过栈溢出，任务切换前栈指针又恢复到了正常水平，这种情况在任务切换的时候是检测不到的。又比如任务栈溢出后，把这部分栈区的数据修改了，这部分栈区的数据不重要或者暂时没有用到还好，但如果是重要数据被修改将直接导致系统进入硬件异常，这种情况下，栈溢出检测功能也是检测不到的。  
        使用方法一需要用户在 FreeRTOSConfig.h 文件中配置如下宏定义：

```c
static void StackOverflowTest(void)
{
	int16_t i;
	uint8_t buf[4906];
	
	(void)buf; /* 防止警告 */
	
	/*
	  1. 为了能够模拟任务栈溢出，并触发任务栈溢出函数，这里强烈建议使用数组的时候逆着赋值。
	     因为对于M3和M4内核的MCU，堆栈生长方向是向下生长的满栈。即高地址是buf[2047], 低地址
	     是buf[0]。如果任务栈溢出了，也是从高地址buf[2047]到buf[0]的某个地址开始溢出。
	        因此，如果用户直接修改的是buf[0]开始的数据且这些溢出部分的数据比较重要，会直接导致
	     进入到硬件异常。
	  2. 栈溢出检测是在任务切换的时候执行的，我们这里加个延迟函数，防止修改了重要的数据导致直接
	     进入硬件异常。
	  3. 任务vTaskTaskUserIF的栈空间大小是2048字节，在此任务的入口已经申请了栈空间大小
		 ------uint8_t ucKeyCode;
	     ------uint8_t pcWriteBuffer[500];
	     这里再申请如下这么大的栈空间
	     -------int16_t i;
		 -------uint8_t buf[2048];
	     必定溢出。
	*/
	for(i = 4095; i >= 0; i--)
	{
		buf[i] = 0x55;
		vTaskDelay(1);
	}
}
```

### 0.1.7 2、方法二  

        任务创建的时候将任务栈所有数据初始化为 0xa5，任务切换时进行任务栈检测的时候会检测末尾的 16 个字节是否都是 0xa5，通过这种方式来检测任务栈是否溢出了。相比方法一，这种方法的速度稍慢些，但是这样就有效地避免了方法一里面的部分情况。 不过依然不能保证所有的栈溢出都能检测到，比如任务栈末尾的 16 个字节没有用到，即没有被修改，但是任务栈已经溢出了，这种情况是检测不到的。 另外任务栈溢出后，任务栈末尾的 16 个字节没有修改，但是溢出部分的栈区数据被修改了，这部分栈区的数据不重要或者暂时没有用到还好，但如果是重要数据被修改将直接导致系统进入硬件异常，这种情况下，栈溢出检测功能也是检测不到的。  
        使用方法二需要用户在 FreeRTOSConfig.h 文件中配置如下宏定义：

```c
#include "FreeRTOS.h"
#include "task.h"
//------------------------------------------------------------------------------------
#include "stm32f10x.h"
#include "BSP_GPIO.h"
#include "BSP_NVIC.h"
#include "BSP_TIMER.h"
 
 
static TaskHandle_t xHandleTaskStart = NULL;
static TaskHandle_t xHandleTask1 = NULL;
 
 
static void vTaskStart(void *pvParameters);
static void vTask1(void *pvParameters);
static void AppTaskCreate (void); 
 
 
 
#ifdef __DEBUG_STM32F103VC__
	uint32_t volatile StackOverflowHook = 0x00;
	uint32_t volatile StackOverflowHook_En = 0x00;
#endif
 
 
 
 
/*
**************************************************************************************
*	函 数 名: StackOverflowTest
*	功能说明: 任务栈溢出测试
*	形    参: 无
*	返 回 值: 无
**************************************************************************************
*/
static void StackOverflowTest(void)
{
	int16_t i;
	uint8_t buf[4906];
	
	(void)buf; /* 防止警告 */
	
	/*
	  1. 为了能够模拟任务栈溢出，并触发任务栈溢出函数，这里强烈建议使用数组的时候逆着赋值。
	     因为对于M3和M4内核的MCU，堆栈生长方向是向下生长的满栈。即高地址是buf[2047], 低地址
	     是buf[0]。如果任务栈溢出了，也是从高地址buf[2047]到buf[0]的某个地址开始溢出。
	        因此，如果用户直接修改的是buf[0]开始的数据且这些溢出部分的数据比较重要，会直接导致
	     进入到硬件异常。
	  2. 栈溢出检测是在任务切换的时候执行的，我们这里加个延迟函数，防止修改了重要的数据导致直接
	     进入硬件异常。
	  3. 任务vTaskTaskUserIF的栈空间大小是2048字节，在此任务的入口已经申请了栈空间大小
		 ------uint8_t ucKeyCode;
	     ------uint8_t pcWriteBuffer[500];
	     这里再申请如下这么大的栈空间
	     -------int16_t i;
		 -------uint8_t buf[2048];
	     必定溢出。
	*/
	for(i = 4095; i >= 0; i--)
	{
		buf[i] = 0x55;
		vTaskDelay(1);
	}
}
 
 
 
void vTask1(void *pvParameters)
{	
    while(1)
    {
		if (StackOverflowHook_En == 1)
		{
			StackOverflowTest();    //软件模拟方式让堆栈溢出
		}
		
		GPIO_LED1_ON();
		vTaskDelay(250);
		GPIO_LED1_OFF();
		vTaskDelay(250);	
    }
}
 
 
 
/*
*************************************************************************************
*	函 数 名: vApplicationStackOverflowHook
*	功能说明: 栈溢出的钩子函数
*	形    参: xTask        任务句柄
*             pcTaskName   任务名
*	返 回 值: 无
*************************************************************************************
*/
void vApplicationStackOverflowHook( TaskHandle_t xTask, char *pcTaskName )
{
	//printf("任务：%s 发现栈溢出\r\n", pcTaskName);
	
	//用户程序代码，记录溢出次数
	StackOverflowHook++;	
}
				
 
 
/*
***********************************************************************************
*	函 数 名: AppTaskCreate
*	功能说明: 创建应用任务
*	形    参：无
*	返 回 值: 无
***********************************************************************************
*/
static void AppTaskCreate (void)
{
 
	xTaskCreate( vTaskStart,     			// 任务函数  
                 "vTaskStart",   			// 任务名    
                 512,            			// 任务栈大小，单位word，也就是4字节 
                 NULL,           			// 任务参数  
                 3, 						// 任务优先级
                 &xHandleTaskStart);  		// 任务句柄 	
 
}
 
 
 
static void vTaskStart(void *pvParameters)
{
	
	taskENTER_CRITICAL(); 					//进入临界区
 
	
	xTaskCreate( vTask1,     				// 任务函数  
                 "vTask1",   				// 任务名    
                 512,            			// 任务栈大小，单位word，也就是4字节 
                 NULL,           			// 任务参数  
                 2, 						// 任务优先级
                 &xHandleTask1);   			// 任务句柄  			 	
	
	vTaskDelete(xHandleTaskStart); 			//删除开始任务
	taskEXIT_CRITICAL(); 					//退出临界区
}
 
 
 
int main(void)
{
	__set_PRIMASK(1); 	
	GPIO_Configuration();	
	NVIC_Configuration();
 
	//创建任务
	AppTaskCreate();
	//启动调度器
	vTaskStartScheduler();
    while (1);	
}
```

        对于上述 2 种溢出检测，我测试了几种情况，确实不容易检测到，多半进入硬件错误中断，所以以后要是遇到程序硬件中断死循环，清注意检测堆栈大小。不确定的时候，先使用 printf 调试打印任务栈的剩余，选择一个合理安全的值作为堆栈设置，这确实是个难以简单计算的东西。

> 六、栈溢出方法一模拟检测
> ------------

 1、在 **FreeRTOSConfig.h** 文件中添加如下配置宏定义

```c
static void StackOverflowTest(void)
{
	int16_t i;
	uint8_t buf[4906];
	
	(void)buf; /* 防止警告 */
	
	/*
	  1. 为了能够模拟任务栈溢出，并触发任务栈溢出函数，这里强烈建议使用数组的时候逆着赋值。
	     因为对于M3和M4内核的MCU，堆栈生长方向是向下生长的满栈。即高地址是buf[2047], 低地址
	     是buf[0]。如果任务栈溢出了，也是从高地址buf[2047]到buf[0]的某个地址开始溢出。
	        因此，如果用户直接修改的是buf[0]开始的数据且这些溢出部分的数据比较重要，会直接导致
	     进入到硬件异常。
	  2. 栈溢出检测是在任务切换的时候执行的，我们这里加个延迟函数，防止修改了重要的数据导致直接
	     进入硬件异常。
	  3. 任务vTaskTaskUserIF的栈空间大小是2048字节，在此任务的入口已经申请了栈空间大小
		 ------uint8_t ucKeyCode;
	     ------uint8_t pcWriteBuffer[500];
	     这里再申请如下这么大的栈空间
	     -------int16_t i;
		 -------uint8_t buf[2048];
	     必定溢出。
	*/
	for(i = 4095; i >= 0; i--)
	{
		buf[i] = 0x55;
		vTaskDelay(1);
	}
}
```

![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/8a2267836b6481c7b7329715281fc517_MD5.png]]

 2、在 **main.c** 文件中添加如下钩子函数

```c
#include "FreeRTOS.h"
#include "task.h"
//------------------------------------------------------------------------------------
#include "stm32f10x.h"
#include "BSP_GPIO.h"
#include "BSP_NVIC.h"
#include "BSP_TIMER.h"
 
 
static TaskHandle_t xHandleTaskStart = NULL;
static TaskHandle_t xHandleTask1 = NULL;
 
 
static void vTaskStart(void *pvParameters);
static void vTask1(void *pvParameters);
static void AppTaskCreate (void); 
 
 
 
#ifdef __DEBUG_STM32F103VC__
	uint32_t volatile StackOverflowHook = 0x00;
	uint32_t volatile StackOverflowHook_En = 0x00;
#endif
 
 
 
 
/*
**************************************************************************************
*	函 数 名: StackOverflowTest
*	功能说明: 任务栈溢出测试
*	形    参: 无
*	返 回 值: 无
**************************************************************************************
*/
static void StackOverflowTest(void)
{
	int16_t i;
	uint8_t buf[4906];
	
	(void)buf; /* 防止警告 */
	
	/*
	  1. 为了能够模拟任务栈溢出，并触发任务栈溢出函数，这里强烈建议使用数组的时候逆着赋值。
	     因为对于M3和M4内核的MCU，堆栈生长方向是向下生长的满栈。即高地址是buf[2047], 低地址
	     是buf[0]。如果任务栈溢出了，也是从高地址buf[2047]到buf[0]的某个地址开始溢出。
	        因此，如果用户直接修改的是buf[0]开始的数据且这些溢出部分的数据比较重要，会直接导致
	     进入到硬件异常。
	  2. 栈溢出检测是在任务切换的时候执行的，我们这里加个延迟函数，防止修改了重要的数据导致直接
	     进入硬件异常。
	  3. 任务vTaskTaskUserIF的栈空间大小是2048字节，在此任务的入口已经申请了栈空间大小
		 ------uint8_t ucKeyCode;
	     ------uint8_t pcWriteBuffer[500];
	     这里再申请如下这么大的栈空间
	     -------int16_t i;
		 -------uint8_t buf[2048];
	     必定溢出。
	*/
	for(i = 4095; i >= 0; i--)
	{
		buf[i] = 0x55;
		vTaskDelay(1);
	}
}
 
 
 
void vTask1(void *pvParameters)
{	
    while(1)
    {
		if (StackOverflowHook_En == 1)
		{
			StackOverflowTest();    //软件模拟方式让堆栈溢出
		}
		
		GPIO_LED1_ON();
		vTaskDelay(250);
		GPIO_LED1_OFF();
		vTaskDelay(250);	
    }
}
 
 
 
/*
*************************************************************************************
*	函 数 名: vApplicationStackOverflowHook
*	功能说明: 栈溢出的钩子函数
*	形    参: xTask        任务句柄
*             pcTaskName   任务名
*	返 回 值: 无
*************************************************************************************
*/
void vApplicationStackOverflowHook( TaskHandle_t xTask, char *pcTaskName )
{
	//printf("任务：%s 发现栈溢出\r\n", pcTaskName);
	
	//用户程序代码，记录溢出次数
	StackOverflowHook++;	
}
				
 
 
/*
***********************************************************************************
*	函 数 名: AppTaskCreate
*	功能说明: 创建应用任务
*	形    参：无
*	返 回 值: 无
***********************************************************************************
*/
static void AppTaskCreate (void)
{
 
	xTaskCreate( vTaskStart,     			// 任务函数  
                 "vTaskStart",   			// 任务名    
                 512,            			// 任务栈大小，单位word，也就是4字节 
                 NULL,           			// 任务参数  
                 3, 						// 任务优先级
                 &xHandleTaskStart);  		// 任务句柄 	
 
}
 
 
 
static void vTaskStart(void *pvParameters)
{
	
	taskENTER_CRITICAL(); 					//进入临界区
 
	
	xTaskCreate( vTask1,     				// 任务函数  
                 "vTask1",   				// 任务名    
                 512,            			// 任务栈大小，单位word，也就是4字节 
                 NULL,           			// 任务参数  
                 2, 						// 任务优先级
                 &xHandleTask1);   			// 任务句柄  			 	
	
	vTaskDelete(xHandleTaskStart); 			//删除开始任务
	taskEXIT_CRITICAL(); 					//退出临界区
}
 
 
 
int main(void)
{
	__set_PRIMASK(1); 	
	GPIO_Configuration();	
	NVIC_Configuration();
 
	//创建任务
	AppTaskCreate();
	//启动调度器
	vTaskStartScheduler();
    while (1);	
}
```

![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/cec9eeacaaa8bb5e46e83818bbc35a1a_MD5.png]]

3、在 **main.c** 文件编写一个测试函数 static void StackOverflowTest(void)，在该函数内定义一个特别大的局部数组，远远超出当初分配给 Task1 任务的栈空间，然后在该函数中读写局部数组。

```c
static void StackOverflowTest(void)
{
	int16_t i;
	uint8_t buf[4906];
	
	(void)buf; /* 防止警告 */
	
	/*
	  1. 为了能够模拟任务栈溢出，并触发任务栈溢出函数，这里强烈建议使用数组的时候逆着赋值。
	     因为对于M3和M4内核的MCU，堆栈生长方向是向下生长的满栈。即高地址是buf[2047], 低地址
	     是buf[0]。如果任务栈溢出了，也是从高地址buf[2047]到buf[0]的某个地址开始溢出。
	        因此，如果用户直接修改的是buf[0]开始的数据且这些溢出部分的数据比较重要，会直接导致
	     进入到硬件异常。
	  2. 栈溢出检测是在任务切换的时候执行的，我们这里加个延迟函数，防止修改了重要的数据导致直接
	     进入硬件异常。
	  3. 任务vTaskTaskUserIF的栈空间大小是2048字节，在此任务的入口已经申请了栈空间大小
		 ------uint8_t ucKeyCode;
	     ------uint8_t pcWriteBuffer[500];
	     这里再申请如下这么大的栈空间
	     -------int16_t i;
		 -------uint8_t buf[2048];
	     必定溢出。
	*/
	for(i = 4095; i >= 0; i--)
	{
		buf[i] = 0x55;
		vTaskDelay(1);
	}
}
```

4、定义一个全局变量 StackOverflowHook_En，在任务 Task1 中触发测试函数 static void StackOverflowTest(void) 立即运行。

```c
uint32_t volatile StackOverflowHook_En = 0x00;
```

 5、定义一个全局变量 StackOverflowHook，记录进入钩子函数的次数（表示：溢出的次数）

```c
uint32_t volatile StackOverflowHook = 0x00;
```

 6、完整的主程序如下：

```c
#include "FreeRTOS.h"
#include "task.h"
//------------------------------------------------------------------------------------
#include "stm32f10x.h"
#include "BSP_GPIO.h"
#include "BSP_NVIC.h"
#include "BSP_TIMER.h"
 
 
static TaskHandle_t xHandleTaskStart = NULL;
static TaskHandle_t xHandleTask1 = NULL;
 
 
static void vTaskStart(void *pvParameters);
static void vTask1(void *pvParameters);
static void AppTaskCreate (void); 
 
 
 
#ifdef __DEBUG_STM32F103VC__
	uint32_t volatile StackOverflowHook = 0x00;
	uint32_t volatile StackOverflowHook_En = 0x00;
#endif
 
 
 
 
/*
**************************************************************************************
*	函 数 名: StackOverflowTest
*	功能说明: 任务栈溢出测试
*	形    参: 无
*	返 回 值: 无
**************************************************************************************
*/
static void StackOverflowTest(void)
{
	int16_t i;
	uint8_t buf[4906];
	
	(void)buf; /* 防止警告 */
	
	/*
	  1. 为了能够模拟任务栈溢出，并触发任务栈溢出函数，这里强烈建议使用数组的时候逆着赋值。
	     因为对于M3和M4内核的MCU，堆栈生长方向是向下生长的满栈。即高地址是buf[2047], 低地址
	     是buf[0]。如果任务栈溢出了，也是从高地址buf[2047]到buf[0]的某个地址开始溢出。
	        因此，如果用户直接修改的是buf[0]开始的数据且这些溢出部分的数据比较重要，会直接导致
	     进入到硬件异常。
	  2. 栈溢出检测是在任务切换的时候执行的，我们这里加个延迟函数，防止修改了重要的数据导致直接
	     进入硬件异常。
	  3. 任务vTaskTaskUserIF的栈空间大小是2048字节，在此任务的入口已经申请了栈空间大小
		 ------uint8_t ucKeyCode;
	     ------uint8_t pcWriteBuffer[500];
	     这里再申请如下这么大的栈空间
	     -------int16_t i;
		 -------uint8_t buf[2048];
	     必定溢出。
	*/
	for(i = 4095; i >= 0; i--)
	{
		buf[i] = 0x55;
		vTaskDelay(1);
	}
}
 
 
 
void vTask1(void *pvParameters)
{	
    while(1)
    {
		if (StackOverflowHook_En == 1)
		{
			StackOverflowTest();    //软件模拟方式让堆栈溢出
		}
		
		GPIO_LED1_ON();
		vTaskDelay(250);
		GPIO_LED1_OFF();
		vTaskDelay(250);	
    }
}
 
 
 
/*
*************************************************************************************
*	函 数 名: vApplicationStackOverflowHook
*	功能说明: 栈溢出的钩子函数
*	形    参: xTask        任务句柄
*             pcTaskName   任务名
*	返 回 值: 无
*************************************************************************************
*/
void vApplicationStackOverflowHook( TaskHandle_t xTask, char *pcTaskName )
{
	//printf("任务：%s 发现栈溢出\r\n", pcTaskName);
	
	//用户程序代码，记录溢出次数
	StackOverflowHook++;	
}
				
 
 
/*
***********************************************************************************
*	函 数 名: AppTaskCreate
*	功能说明: 创建应用任务
*	形    参：无
*	返 回 值: 无
***********************************************************************************
*/
static void AppTaskCreate (void)
{
 
	xTaskCreate( vTaskStart,     			// 任务函数  
                 "vTaskStart",   			// 任务名    
                 512,            			// 任务栈大小，单位word，也就是4字节 
                 NULL,           			// 任务参数  
                 3, 						// 任务优先级
                 &xHandleTaskStart);  		// 任务句柄 	
 
}
 
 
 
static void vTaskStart(void *pvParameters)
{
	
	taskENTER_CRITICAL(); 					//进入临界区
 
	
	xTaskCreate( vTask1,     				// 任务函数  
                 "vTask1",   				// 任务名    
                 512,            			// 任务栈大小，单位word，也就是4字节 
                 NULL,           			// 任务参数  
                 2, 						// 任务优先级
                 &xHandleTask1);   			// 任务句柄  			 	
	
	vTaskDelete(xHandleTaskStart); 			//删除开始任务
	taskEXIT_CRITICAL(); 					//退出临界区
}
 
 
 
int main(void)
{
	__set_PRIMASK(1); 	
	GPIO_Configuration();	
	NVIC_Configuration();
 
	//创建任务
	AppTaskCreate();
	//启动调度器
	vTaskStartScheduler();
    while (1);	
}
```

 7、KEIL 软件追踪运行

![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/e1d9bbf78f46058449aa6300d7d36d43_MD5.png]]

运行一段时间后，将全局变量 StackOverflowHook_En 在 KEIL 上强制为 1

![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/dd2316471fa08504372bc1e28bc8da41_MD5.png]]

此时，程序立即跳转到钩子函数 ApplicationStackOverflowHook() 内部，说明任务堆栈已经溢出

![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/6d988b77a49ab9f84d876162f67b0d76_MD5.png]]

方法一堆栈溢出检测完毕。

> ### 七、栈溢出方法二模拟检测

1、在 **FreeRTOSConfig.h** 文件中添加如下配置宏定义

```c
#define configCHECK_FOR_STACK_OVERFLOW  2
```

![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/add5b2959b0864e3ad18183df91c33c3_MD5.png]]

2、在 **main.c** 文件中添加如下钩子函数

```c
void vApplicationStackOverflowHook( TaskHandle_t xTask, char *pcTaskName )
```

![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/cec9eeacaaa8bb5e46e83818bbc35a1a_MD5.png]]

3、在 **main.c** 文件编写一个测试函数 static void StackOverflowTest(void)，在该函数内定义一个特别大的局部数组，远远超出当初分配给 Task1 任务的栈空间，然后在该函数中读写局部数组。

```c
static void StackOverflowTest(void)
{
	int16_t i;
	uint8_t buf[4906];
	
	(void)buf; /* 防止警告 */
	
	/*
	  1. 为了能够模拟任务栈溢出，并触发任务栈溢出函数，这里强烈建议使用数组的时候逆着赋值。
	     因为对于M3和M4内核的MCU，堆栈生长方向是向下生长的满栈。即高地址是buf[2047], 低地址
	     是buf[0]。如果任务栈溢出了，也是从高地址buf[2047]到buf[0]的某个地址开始溢出。
	        因此，如果用户直接修改的是buf[0]开始的数据且这些溢出部分的数据比较重要，会直接导致
	     进入到硬件异常。
	  2. 栈溢出检测是在任务切换的时候执行的，我们这里加个延迟函数，防止修改了重要的数据导致直接
	     进入硬件异常。
	  3. 任务vTaskTaskUserIF的栈空间大小是2048字节，在此任务的入口已经申请了栈空间大小
		 ------uint8_t ucKeyCode;
	     ------uint8_t pcWriteBuffer[500];
	     这里再申请如下这么大的栈空间
	     -------int16_t i;
		 -------uint8_t buf[2048];
	     必定溢出。
	*/
	for(i = 4095; i >= 0; i--)
	{
		buf[i] = 0x55;
		vTaskDelay(1);
	}
}
```

 4、定义一个全局变量 StackOverflowHook_En，在任务 Task1 中触发测试函数 static void StackOverflowTest(void) 立即运行。

```c
uint32_t volatile StackOverflowHook_En = 0x00;
```

 5、定义一个全局变量 StackOverflowHook，记录进入钩子函数的次数（表示：溢出的次数）

```c
uint32_t volatile StackOverflowHook = 0x00;
```

6、完整的主程序如下：

```c
#include "FreeRTOS.h"
#include "task.h"
//------------------------------------------------------------------------------------
#include "stm32f10x.h"
#include "BSP_GPIO.h"
#include "BSP_NVIC.h"
#include "BSP_TIMER.h"
 
 
static TaskHandle_t xHandleTaskStart = NULL;
static TaskHandle_t xHandleTask1 = NULL;
 
 
static void vTaskStart(void *pvParameters);
static void vTask1(void *pvParameters);
static void AppTaskCreate (void); 
 
 
 
#ifdef __DEBUG_STM32F103VC__
	uint32_t volatile StackOverflowHook = 0x00;
	uint32_t volatile StackOverflowHook_En = 0x00;
#endif
 
 
 
 
/*
**************************************************************************************
*	函 数 名: StackOverflowTest
*	功能说明: 任务栈溢出测试
*	形    参: 无
*	返 回 值: 无
**************************************************************************************
*/
static void StackOverflowTest(void)
{
	int16_t i;
	uint8_t buf[4906];
	
	(void)buf; /* 防止警告 */
	
	/*
	  1. 为了能够模拟任务栈溢出，并触发任务栈溢出函数，这里强烈建议使用数组的时候逆着赋值。
	     因为对于M3和M4内核的MCU，堆栈生长方向是向下生长的满栈。即高地址是buf[2047], 低地址
	     是buf[0]。如果任务栈溢出了，也是从高地址buf[2047]到buf[0]的某个地址开始溢出。
	        因此，如果用户直接修改的是buf[0]开始的数据且这些溢出部分的数据比较重要，会直接导致
	     进入到硬件异常。
	  2. 栈溢出检测是在任务切换的时候执行的，我们这里加个延迟函数，防止修改了重要的数据导致直接
	     进入硬件异常。
	  3. 任务vTaskTaskUserIF的栈空间大小是2048字节，在此任务的入口已经申请了栈空间大小
		 ------uint8_t ucKeyCode;
	     ------uint8_t pcWriteBuffer[500];
	     这里再申请如下这么大的栈空间
	     -------int16_t i;
		 -------uint8_t buf[2048];
	     必定溢出。
	*/
	for(i = 4095; i >= 0; i--)
	{
		buf[i] = 0x55;
		vTaskDelay(1);
	}
}
 
 
 
void vTask1(void *pvParameters)
{	
    while(1)
    {
		if (StackOverflowHook_En == 1)
		{
			StackOverflowTest();    //软件模拟方式让堆栈溢出
		}
		
		GPIO_LED1_ON();
		vTaskDelay(250);
		GPIO_LED1_OFF();
		vTaskDelay(250);	
    }
}
 
 
 
/*
*************************************************************************************
*	函 数 名: vApplicationStackOverflowHook
*	功能说明: 栈溢出的钩子函数
*	形    参: xTask        任务句柄
*             pcTaskName   任务名
*	返 回 值: 无
*************************************************************************************
*/
void vApplicationStackOverflowHook( TaskHandle_t xTask, char *pcTaskName )
{
	//printf("任务：%s 发现栈溢出\r\n", pcTaskName);
	
	//用户程序代码，记录溢出次数
	StackOverflowHook++;	
}
				
 
 
/*
***********************************************************************************
*	函 数 名: AppTaskCreate
*	功能说明: 创建应用任务
*	形    参：无
*	返 回 值: 无
***********************************************************************************
*/
static void AppTaskCreate (void)
{
 
	xTaskCreate( vTaskStart,     			// 任务函数  
                 "vTaskStart",   			// 任务名    
                 512,            			// 任务栈大小，单位word，也就是4字节 
                 NULL,           			// 任务参数  
                 3, 						// 任务优先级
                 &xHandleTaskStart);  		// 任务句柄 	
 
}
 
 
 
static void vTaskStart(void *pvParameters)
{
	
	taskENTER_CRITICAL(); 					//进入临界区
 
	
	xTaskCreate( vTask1,     				// 任务函数  
                 "vTask1",   				// 任务名    
                 512,            			// 任务栈大小，单位word，也就是4字节 
                 NULL,           			// 任务参数  
                 2, 						// 任务优先级
                 &xHandleTask1);   			// 任务句柄  			 	
	
	vTaskDelete(xHandleTaskStart); 			//删除开始任务
	taskEXIT_CRITICAL(); 					//退出临界区
}
 
 
 
int main(void)
{
	__set_PRIMASK(1); 	
	GPIO_Configuration();	
	NVIC_Configuration();
 
	//创建任务
	AppTaskCreate();
	//启动调度器
	vTaskStartScheduler();
    while (1);	
}
```

 7、KEIL 软件追踪运行

![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/e1d9bbf78f46058449aa6300d7d36d43_MD5.png]]

运行一段时间后，将全局变量 StackOverflowHook_En 在 KEIL 上强制为 1

![[../../_resources/FreeRTOS 任务栈大小确定及其溢出检测方法【杂记】_freertos 任务栈大小的确定/dd2316471fa08504372bc1e28bc8da41_MD5.png]]

程序死机（出现硬件死机），说明任务堆栈溢出。

方法二堆栈溢出检测完毕。