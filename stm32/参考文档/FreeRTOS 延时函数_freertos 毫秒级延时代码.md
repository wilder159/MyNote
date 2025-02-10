> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_44333597/article/details/107848797)

vTaskDelay() 相对延时函数
-------------------

vTaskDelay() 延时固定数量的 tick 中断，将调用任务置于阻塞状态。（**vTaskDelay() 函数只有在宏 INCLUDE_vTaskDelay 置 1 时才可用**）

```
void vTaskDelay( TickType_t xTicksToDelay );
```

**参数作用**：如果一个任务调用函数 vTaskDelay(100)，此时滴答中断计数为 10000，然后它将立即进入阻塞状态，并保持阻塞状态直到滴答中断计数达到 10100

**宏 pdMS_TO_TICKS() **可用于**将以毫秒为单位指定的时间转换为以 ticks 为单位指定的时间**。例如，调用 vTaskDelay(pdMS_TO_TICKS(100)) 将使任务保持阻塞状态 100 毫秒。

```
宏定义#define     pdMS_TO_TICKS( xTimeInMs )       ( ( TickType_t ) ( ( ( TickType_t ) ( xTimeInMs ) * ( TickType_t ) configTICK_RATE_HZ ) / ( TickType_t ) 1000 ) )
```

> ```
> 宏定义 #define    configTICK_RATE_HZ						(1000)           //时钟节拍频率，这里设置为1000，滴答定时器中断周期就是1ms
> ```
> 
> FreeRTOS 的系统时钟是由滴答定时器提供的，应根据 FreeRTOS 的系统时钟节拍来初始化滴答定时器。 FreeRTOS 的系统时钟节拍由宏 configTICK_RATE_HZ 来设置，这个值可自由设置，但是一旦设置好以后就要根据这个值来初始化滴答定时器，其实就是设置滴答定时器的中断周期。

**当任务处于阻塞状态时，它不占用 CPU。**  
**定义**：每次延时从任务执行 vTaskDelay() 函数时开始，延时一定的时间结束。  
![[../../_resources/FreeRTOS 延时函数_freertos 毫秒级延时代码/5d3589e9566046ee5235126abc79f61d_MD5.jpg]]  
**设定：TSET 任务的优先级最高**

**TEST 任务延时都是从调用延时函数 vTaskDelay() 开始算起的，延时从此刻算起**，执行 TEST 任务过程中产生中断或执行更高优先级任务时，两次 TEST 任务之间的时间周期会变长，**TSET 任务无法周期性地执行！**

```
void vTaskFunction( void *pvParameters )
{
	char *pcTaskName;
	const TickType_t xDelay250ms = pdMS_TO_TICKS( 250 );  //定义延时时长
	/*要打印的字符串通过指针传入，因此将其转换为字符指针*/
	pcTaskName = ( char * ) pvParameters;
	
	for( ;; )
	{
		vTaskDelay( xDelay250ms );  //调用延时，时长250ms
		vPrintString( pcTaskName ); //打印字符串
		
	}
}
```

**绝对延时函数 vTaskDelayUntil()**
----------------------------

**定义**：每隔指定的一段时间，执行一次调用 vTaskDelayUntil() 函数的任务

```
void vTaskDelayUntil( TickType_t * pxPreviousWakeTime, TickType_t xTimeIncrement );
```

*   参数：pxPreviousWakeTime 该参数保存此任务最后一次离开阻塞态（即延时结束被唤醒）的时间点，任务中第一次调用函数 vTaskDelayUntil 的话需要将 pxPreviousWakeTime 初始化为在进入 while 循环之前的滴答计数值。在以后的运行中函数 vTaskDelayUntil() 会自动更新 pxPreviousWakeTime 的值。
    
    ```
    void vTaskFunction( void *pvParameters )
    {
    	char *pcTaskName;
    	TickType_t xLastWakeTime;
        pcTaskName = ( char * ) pvParameters;
    	xLastWakeTime = xTaskGetTickCount();  //初始化为进入while之前的滴答计数值
        while(1)
        {
            vTaskDelayUntil( &xLastWakeTime, pdMS_TO_TICKS( 250 ) );  //延时
            vPrintString( pcTaskName );
          
        }
    }
    ```
    
*   xTimeIncrement 定期延时的时长，同 vTaskDelay() API 的函数参数用法相同。
    

![[../../_resources/FreeRTOS 延时函数_freertos 毫秒级延时代码/b17647164d2acb2b0c475018708876e7_MD5.jpg]]

**设置：TSET 任务具有最高优先级！**

从第一次上电调用绝对[延时函数](https://so.csdn.net/so/search?q=%E5%BB%B6%E6%97%B6%E5%87%BD%E6%95%B0&spm=1001.2101.3001.7020)起，首先执行其他任务，然后 TEST 任务主体周期性地执行，即使在执行 TEST 任务过程中发生中断，也不会影响 TEST 任务的执行周期，但会缩短其他任务的执行时间。（注意：**周期性延时时间必须大于任务主体代码执行时间**, 才会将任务挂接到延时列表）