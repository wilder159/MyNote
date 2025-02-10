# 1 链表结构
链表根节点结构体(xLIST)：
```c
typedef struct xLIST
{
	UBaseType_t uxNumberOfItems; /*链表计数器，记录节点数量，第零节点不做计数*/
	ListItem_t * pxIndex;        /*链表节点索引指针，用于遍历节点*/
	MiniListItem_t xListEnd;     /*链表的第零节点*/
} List_t;
```
MiniListItem_t 第零节点结构体（链表节点结构体精简版）
```c
struct xMINI_LIST_ITEM
{
	TickType_t xItemValue;          /*零节点的辅助排序值最大 portMax_DELAY*/     
	struct xLIST_ITEM *  pxNext;    /*指向下一个节点*/
	struct xLIST_ITEM *  pxPrevious;/*指向上一个节点*/
};
typedef struct xMINI_LIST_ITEM MiniListItem_t;
```
链表节点结构体 (xLIST_ITEM):
```c
struct xLIST_ITEM
{	
	TickType_t xItemValue;			 /*辅助排序值*/
	struct xLIST_ITEM * pxNext;		 /*指向下一个节点*/
	struct xLIST_ITEM * pxPrevious;	 /*指向上一个节点*/
	void * pvOwner;					 /*指向任务控制块*/			
	void * pvContainer;				 /*指向该节点所在的链表根结构体*/
};
typedef struct xLIST_ITEM ListItem_t;					
```

**xLIST_ITEM中pvOwner它的主要作用是指向包含该链表项的对象，通常是任务控制块（TCB）。通过`pvOwner`，FreeRTOS可以从链表项中找到它所挂载的任务控制块，从而实现对任务的管理和调度**

[[freeretos链表结构.excalidraw]]

![[../_resources/freertos/0d6a76dfcafa0d670ead46bda5bccebf_MD5.jpeg]]
# 2 任务

静态任务栈
```c
#define portSTACK_TYPE uint32_t
typedef portSTACK_TYPE StackType_t;
#define TASK1_STACK_SIZE 128
/*任务栈大小，单位字（四个字节） 最小128字 512字节*/
StackType_t Task1Stack[TASK1_STACK_SIZE];
```
任务控制块，存储任务的所有信息，比如任务的栈指针，名称，形参
```c
typedef struct tskTaskControlBlock
{
	volatile StackType_t *pxTopOfStack;        /* 栈顶 */
	ListItem_t           xStateListItem;       /* 任务链表节点*/ 
	ListItem_t           xEventListItem;       /* 事件链表节点*/
	StackType_t          *pxStack;             /* 任务栈起始地址 */
	char                 pcTaskName[ configMAX_TASK_NAME_LEN ];/*任务名称*/
} tskTCB;
typedef tskTCB TCB_t;
```

任务创建函数，静态创建xTaskCreateStatic()
```c
typedef void * TaskHandle_t;         
typedef void (*TaskFunction_t)( void * );
TaskHandle_t xTaskCreateStatic(	
	TaskFunction_t pxTaskCode,      /*任务入口*/
	const char * const pcName,      /*任务名称*/
	const uint32_t ulStackDepth,    /*任务栈大小，单位字（四个字节） 最小128字 512字节*/
	void * const pvParameters,      /*任务行参*/
	UBaseType_t uxPriority,         /*任务优先级*/
	StackType_t * const puxStackBuffer,/*栈指针*/
	StaticTask_t * const pxTaskBuffer /*StaticTask_t与TCB_t结构体相同，就是任务控制块指针，目的隐藏TCB_t内部数据结构*/
){
    TCB_t *pxNewTCB;
    TaskHandle_t xReturn; /*任务句柄，返回结果*/
	pxNewTCB = ( TCB_t * ) pxTaskBuffer;
	pxNewTCB->pxStack = ( StackType_t * ) puxStackBuffer;
	prvInitialiseNewTask( pxTaskCode, pcName, ulStackDepth, pvParameters, uxPriority, &xReturn, pxNewTCB, NULL );
	prvAddNewTaskToReadyList( pxNewTCB );将任务块加入就行列表
	return xReturn;  /*如果任务创建成功，此时返回结果为任务块指针，失败为NULL指针*/
}
```
创建任务
```c
static void prvInitialiseNewTask(   
	TaskFunction_t pxTaskCode,
	const char * const pcName,
	const uint32_t ulStackDepth,
	void * const pvParameters,
	UBaseType_t uxPriority,
	TaskHandle_t * const pxCreatedTask,
	TCB_t *pxNewTCB,
	const MemoryRegion_t * const xRegions /*内存保护*/
){
	/*获取到栈顶*/
	pxTopOfStack = pxNewTCB->pxStack + ( ulStackDepth - ( uint32_t ) 1 );
	/*向下做8字节对齐*/
	pxTopOfStack = ( StackType_t * ) ( ( ( portPOINTER_SIZE_TYPE ) pxTopOfStack ) & ( ~( ( portPOINTER_SIZE_TYPE ) portBYTE_ALIGNMENT_MASK ) ) );
	/* 将任务的名字存储在 TCB 中 */
	for( x = ( UBaseType_t ) 0; x < ( UBaseType_t ) configMAX_TASK_NAME_LEN; x++ )
	{
		pxNewTCB->pcTaskName[ x ] = pcName[ x ];
		if( pcName[ x ] == 0x00 ){break;}else{mtCOVERAGE_TEST_MARKER();}
	}
	/* 任务名字的长度不能超过 configMAX_TASK_NAME_LEN */
	pxNewTCB->pcTaskName[ configMAX_TASK_NAME_LEN - 1 ] = '\0';
	/*初始化任务链表和事件链表*/
	vListInitialiseItem( &( pxNewTCB->xStateListItem ) );
    vListInitialiseItem( &( pxNewTCB->xEventListItem ) );
    /*将任务块指针添加进链表*/
    listSET_LIST_ITEM_OWNER( &( pxNewTCB->xStateListItem ), pxNewTCB );
    listSET_LIST_ITEM_OWNER( &( pxNewTCB->xEventListItem ), pxNewTCB );
    /*初始化任务栈,跟硬件有关，需要选择不同的port.c*/
    pxNewTCB->pxTop      OfStack = pxPortInitialiseStack( pxTopOfStack, pxTaskCode, pvParameters );
    /*返回结果*/
    if( ( void * ) pxCreatedTask != NULL )
    {
        /* Pass the handle out in an anonymous way.  The handle can be used to
        change the created task's priority, delete the created task, etc.*/
        *pxCreatedTask = ( TaskHandle_t ) pxNewTCB;
    }
    else
    {
        mtCOVERAGE_TEST_MARKER();
    }
```

`portBYTE_ALIGNMENT` 是 FreeRTOS 中用于指定内存对齐方式的宏定义。内存对齐是指在内存中分配数据时，确保数据的起始地址是某个特定数值的倍数。这种做法可以提高数据访问的效率，并且在某些硬件平台上是必需的。

在 FreeRTOS 中，`portBYTE_ALIGNMENT` 的值决定了内存分配时的对齐要求。例如，如果 `portBYTE_ALIGNMENT` 被设置为 8，则所有分配的内存块的起始地址都必须是 8 的倍数。
这个值通常由硬件平台的  需求决定，并且在 FreeRTOS 配置文件中进行设置。
`portBYTE_ALIGNMENT_MASK` 是 FreeRTOS 中用于实现内存对齐的宏定义。它与 `portBYTE_ALIGNMENT` 配合使用，用于计算需要对齐的字节数。
`portBYTE_ALIGNMENT_MASK` 的值取决于 `portBYTE_ALIGNMENT` 的值。它们之间的对应关系如下：

| `portBYTE_ALIGNMENT` | `portBYTE_ALIGNMENT_MASK` |
| -------------------- | ------------------------- |
| 8                    | 0x0007                    |
| 4                    | 0x0003                    |
| 2                    | 0x0001                    |
| 1                    | 0x0000                    |
例如，如果 `portBYTE_ALIGNMENT` 被设置为 8，则 `portBYTE_ALIGNMENT_MASK` 应该被设置为 0x0007。
```c
/*栈顶存储要自动加载到 CPU 寄存器的内容，剩余空间给程序使用*/
StackType_t *pxPortInitialiseStack( 
	StackType_t *pxTopOfStack,/*栈顶*/
	TaskFunction_t pxCode,    /*代码入口*/
	void *pvParameters        /*传参*/
){
    pxTopOfStack--; /* 异常发生时，自动加载到 CPU 寄存器的内容*/
    *pxTopOfStack = portINITIAL_XPSR;   /* xPSR */
    pxTopOfStack--;
    *pxTopOfStack = ( ( StackType_t ) pxCode ) & portSTART_ADDRESS_MASK;
    pxTopOfStack--;
    *pxTopOfStack = ( StackType_t ) prvTaskExitError;   /* LR */
    pxTopOfStack -= 5;  /* R12, R3, R2 and R1. */
    *pxTopOfStack = ( StackType_t ) pvParameters;   /* R0 */
    pxTopOfStack -= 8;  /* R11, R10, R9, R8, R7, R6, R5 and R4. */
    return pxTopOfStack;
}
```
[[../_resources/freertos/de9048d635688706034d8c9a0c8ba514_MD5.jpeg|任务栈初始化完成后栈空间分布图]]
![[../_resources/freertos/de9048d635688706034d8c9a0c8ba514_MD5.jpeg]]
将任务插入到就绪列表中
```c
//在xTaskCreateStatic()函数中，初始化完任务节点和任务块后，调用prvAddNewTaskToReadyList()函数将任务插入就绪队列中
prvAddNewTaskToReadyList( pxNewTCB ){
	prvAddTaskToReadyList( pxNewTCB );
}
#define prvAddTaskToReadyList( pxTCB )                  \
    \*检查优先级值的合法性，超过最大值，改为最大值*\
    taskRECORD_READY_PRIORITY( ( pxTCB )->uxPriority ); \ 
    \*将任务插入各自就绪队列中，每个优先级有自己的队列*\
    vListInsertEnd( &( pxReadyTasksLists[ ( pxTCB )->uxPriority ] ), &( ( pxTCB )->xStateListItem ) ); \
```
# 3 调度器
```c
void vTaskStartScheduler( void ){
	/* 手动指定第一个运行的任务 */
	pxCurrentTCB = &Task1TCB;
	/* 启动调度器 */
	if ( xPortStartScheduler() != pdFALSE ){
		/* 调度器启动成功，则不会返回，即不会来到这里 */
	}
}
```
xPortStartScheduler()函数
```c
#define portNVIC_SYSPRI2_REG ( * ( ( volatile uint32_t * ) 0xe000ed20 ) )
#define portNVIC_PENDSV_PRI  (((uint32_t) configKERNEL_INTERRUPT_PRIORITY)<<16UL)
#define portNVIC_SYSTICK_PRI (((uint32_t) configKERNEL_INTERRUPT_PRIORITY)<<24UL)

BaseType_t xPortStartScheduler( void ){
	/*设置Systick和PendSV中断优先级为最低*/
    portNVIC_SYSPRI2_REG |= portNVIC_PENDSV_PRI;
    portNVIC_SYSPRI2_REG |= portNVIC_SYSTICK_PRI;
    /* 配置SysTick定时器，控制状态寄存器、重装载值*/
    vPortSetupTimerInterrupt();
    /*启动第一个任务,不再返回*/
    prvStartFirstTask();
    return 0;
    }
```
prvStartFirstTask()函数
```c
static void prvPortStartFirstTask( void )
{
__asm volatile(

	" ldr r0, =0xE000ED08   \n" /* 中断向量表地址存储在这个地址中*/
	" ldr r0, [r0]          \n"
	" ldr r0, [r0]          \n"
	" msr msp, r0           \n" /* 设置主堆栈地址*/
	" cpsie i               \n" /*开启中断*/
	" cpsie f               \n" /*开启异常*/
	" dsb                   \n"
	" isb                   \n"
	" svc 0                 \n" /* System call to start first task.  系统调用进入到SVC中断函数中*/
	" nop                   \n"
	);

}
```

> DMB 数据存储器屏障。确保在执行新的存储器访问前所有的存储器访问都已经完成
> DSB  数据同步屏障。确保在下一条指令执行前所有的存储器访问都已经完成
> ISB   指令同步屏障。清空流水线，确保在执行新的指令前，之前所有的指令都已完成

| 名字        | 功能描述                                                                                                           |
| --------- | -------------------------------------------------------------------------------------------------------------- |
| PRIMASK   | 这是个只有单一比特的寄存器。 在它被置 1 后，就关掉所有可屏蔽的异常，只剩下 NMI 和硬 FAULT 可以响应。它的缺省值是 0，表示没有关中断。                                    |
| FAULTMASK | 这是个只有 1 个位的寄存器。当它置 1 时，只有 NMI 才能响应，所有其它的异常，甚至是硬 FAULT，也通通闭嘴。它的缺省值也是 0，表示没有关异常。                                 |
| BASEPRI   | 这个寄存器最多有 9 位（ 由表达优先级的位数决定）。它定义了被屏蔽优先级的阈值。当它被设成某个值后，所有优先级号大于等于此值的中断都被关（优先级号越大，优先级越低）。但若被设成 0，则不关闭任何中断， 0 也是缺省值。 |
```c
void vPortSVCHandler( void )
{
__asm volatile (
	"   ldr r3, pxCurrentTCBConst2      \n"
	"   ldr r1, [r3]                    \n" 
	"   ldr r0, [r1]                    \n" /*获取当前任务的任务控制块栈顶*/
	"   ldmia r0!, {r4-r11}             \n" /*向上将栈中8个字的内容加载到CPU寄存器r4~r11中，r0也会自增*/
	"   msr psp, r0                     \n" /*还原任务栈指针 */
	"   isb                             \n"
	"   mov r0, #0                      \n" /*r0清零*/
	"   msr basepri, r0                 \n" /*打开所有中断*/
	"   orr r14, #0xd                   \n" 
	"   bx r14                          \n" /*异常返回，栈指针使用的是PSP，并自动将剩余内容加载到CPU寄存器，同时 PSP 的值也将更新，即指向任务栈的栈顶。*/
	"                                   \n"
	"   .align 4                        \n"
	"pxCurrentTCBConst2: .word pxCurrentTCB \n"
}
```

```c
void xPortPendSVHandler( void )
{
__asm volatile
(
/*当进入 PendSVC Handler 时，上一个任务运行的环境即： xPSR，PC（任务入口地址），R14，R12，R3，R2，R1，R0（任务的形参）这些 CPU 寄存器的值会自动存储到任务的栈中，剩下的 r4~r11 需要手动保存，同时PSP 会自动更新（在更新之前 PSP 指向任务栈的栈顶）*/
	"   mrs r0, psp                         \n" /* 将PSP值存储到r0 */
	"   isb                                 \n"	
	"                                       \n"
	"   ldr r3, pxCurrentTCBConst           \n" /* 将当前任务块地址给r3 */
	"   ldr r2, [r3]                        \n" /* 加载 r3 指向的内容到 r2，即 r2 等于 pxCurrentTCB */
	"                                       \n"
	"   stmdb r0!, {r4-r11}                 \n" /* Save the remaining registers. */
	"   str r0, [r2]                        \n" /* Save the new top of stack into the first member of the TCB. */
	"                                       \n"
	"   stmdb sp!, {r3, r14}                \n" /* 保持R3 R14 保存的是当前正在运行的任务 */
	"   mov r0, %0                          \n"
	"   msr basepri, r0                     \n" /* 关闭中断*/
	"   bl vTaskSwitchContext               \n" /* 更新pxCurrentTCB，选择优先级最高的任务 */
	"   mov r0, #0                          \n"
	"   msr basepri, r0                     \n" /* 开启中断 */
	"   ldmia sp!, {r3, r14}                \n" /* 还原R3 R14 */
	"                                       \n" 
	"   ldr r1, [r3]                        \n" 
	"   ldr r0, [r1]                        \n" /* 将要运行的任务控制块 */
	"   ldmia r0!, {r4-r11}                 \n" /* 载入r4~r11寄存器 */
	"   msr psp, r0                         \n" /* 更新PSP值 */
	"   isb                                 \n"
	"   bx r14                              \n" /*  */
	"                                       \n"
	"   .align 4                            \n"
	"pxCurrentTCBConst: .word pxCurrentTCB  \n"
	::"i"(configMAX_SYSCALL_INTERRUPT_PRIORITY)
);
}
```

# 4 临界段保护
临界段用一句话概括就是一段在执行的时候不能被中断的代码段。
```c
/*任务级临界区 中断值不保护 不能嵌套，不能在中断中使用*/
#define taskENTER_CRITICAL() portENTER_CRITICAL()
#define taskEXIT_CRITICAL() portEXIT_CRITICAL()
/*中断级临界区 中断值保护 能嵌套，在中断中使用*/
#define taskENTER_CRITICAL_FROM_ISR() portSET_INTERRUPT_MASK_FROM_ISR()
#define taskEXIT_CRITICAL_FROM_ISR( x ) portCLEAR_INTERRUPT_MASK_FROM_ISR( x )
```
# 5 空闲任务与阻塞延时实现
## 5.1 空闲任务

## 5.2 阻塞延时

freertos未实现此方法，接 **7 任务延时函数**
```c 
/*延时阻塞函数*/
void vTaskDelay( const TickType_t xTicksToDelay ){
	TCB_t *pxTCB = NULL;
	pxTCB = pxCurrentTCB;//获取当前任务的TCB
	pxTCB->xTicksToDelay = xTicksToDelay;//设置延时函数
	taskYIELD();         //任务调度->产生PendSV，进行任务切换
}
```
SysTick中断服务函数，通过SysTick中断对任务进行调度
```c
#define xPortSysTickHandler SysTick_Handler
void xPortSysTickHandler( void ){
	vPortRaiseBASEPRI();/*关闭中断*/
	xTaskIncrementTick();/*更新时基*/
	vPortClearBASEPRIFromISR();/*开启中断*/
}
```

TaskIncrementTick()函数
```c
void xTaskIncrementTick( void ){
	TCB_t *pxTCB = NULL;
	BaseType_t i = 0;
	/* 更新系统时基计数器 xTickCount，xTickCount 是一个在 port.c 中定义的全局变量 */
	const TickType_t xConstTickCount = xTickCount + 1;
	xTickCount = xConstTickCount;
	 /* 扫描就绪列表中所有任务的 xTicksToDelay，如果不为 0，则减 1 */
	for (i=0; i<configMAX_PRIORITIES; i++)
	{
		pxTCB = ( TCB_t * ) listGET_OWNER_OF_HEAD_ENTRY( ( &pxReadyTasksLists[i] ) );
		if (pxTCB->xTicksToDelay > 0)
		{
			pxTCB->xTicksToDelay--;
		}
	}
	/* 任务切换 */
	portYIELD();
}
```


# 6 多优先级

值越大优先级越大
# 7 任务延时函数

延时函数列表

```c
/*定义了两个延时列表，当任务延时时间加上系统时基计数器（xTickCount）没有溢出时，将任务加入pxDelayedTaskList指向的列表中，有溢出加入到pxOverflowDelayedTaskList指向的列表中。当系统时基计数器（xTickCount）溢出后，交换两个列表交换的指针*/		
static List_t xDelayedTaskList1;						
static List_t xDelayedTaskList2;
static List_t * volatile pxDelayedTaskList;
static List_t * volatile pxOverflowDelayedTaskList;
```

任务延时列表初始化
```c
/* 初始化任务相关的列表 */
void prvInitialiseTaskLists( void )
{
	UBaseType_t uxPriority;
	/* 初始化就绪列表 */
	for (uxPriority = ( UBaseType_t ) 0U;
		uxPriority < ( UBaseType_t ) configMAX_PRIORITIES;
		uxPriority++ )
	{
		vListInitialise( &( pxReadyTasksLists[ uxPriority ] ) );
	}
	vListInitialise( &xDelayedTaskList1 );
	vListInitialise( &xDelayedTaskList2 );
	pxDelayedTaskList = &xDelayedTaskList1;
	pxOverflowDelayedTaskList = &xDelayedTaskList2;
}
```

**定义xNextTaskUnblockTime**：xNextTaskUnblockTime 是一个在 task.c 中定义的静态变量，用于表示下一个任务的解锁时刻。xNextTaskUnblockTime 的值等于系统时基计数器的值 xTickCount 加上任务需要延时值 xTicksToDelay。当系统时基计数器 xTickCount 的值与 xNextTaskUnblockTime 相等时，就表示有任务延时到期了，需要将该任务就绪。
**xNextTaskUnblockTime 在 vTaskStartScheduler()函数中初始化为 portMAX_DELAY, 是一个 portmacro.h 中定义的宏，默认为 0xffffffffUL）**

vTaskDelay()函数
```c
void vTaskDelay( const TickType_t xTicksToDelay )
{
	vTaskSuspendAll();          /*暂停任务调度*/
	prvAddCurrentTaskToDelayedList( xTicksToDelay ); /* 将任务插入到延时列表 */ 
	xAlreadyYielded = xTaskResumeAll(); /*继续任务调度*/
 }
```
prvAddCurrentTaskToDelayedList( xTicksToDelay )函数
```c
static void prvAddCurrentTaskToDelayedList( TickType_t xTicksToWait )
{
	TickType_t xTimeToWake;
	/* 获取系统时基计数器 xTickCount 的值 */
	const TickType_t xConstTickCount = xTickCount; 

	/* 将任务从就绪列表中移除 */
	if ( uxListRemove( &( pxCurrentTCB->xStateListItem ) )== ( UBaseType_t ) 0 )
	 {
		 /* 将任务在优先级位图中对应的位清除 */
		 portRESET_READY_PRIORITY( pxCurrentTCB->uxPriority,uxTopReadyPriority );
	 }
	 
	 /* 计算任务延时到期时，系统时基计数器 xTickCount 的值是多少 */
	 xTimeToWake = xConstTickCount + xTicksToWait;
	 
	 /* 将延时到期的值设置为节点的排序值 */ 
	 listSET_LIST_ITEM_VALUE( &( pxCurrentTCB->xStateListItem ),xTimeToWake );
	 
	 /* 溢出 */ 
	 if ( xTimeToWake < xConstTickCount )
	 {
		 vListInsert( pxOverflowDelayedTaskList,&(pxCurrentTCB->xStateListItem));
	 }
	 else /* 没有溢出 */
	 {
		 vListInsert( pxDelayedTaskList,&( pxCurrentTCB->xStateListItem ) );
		 /* 更新下一个任务解锁时刻变量 xNextTaskUnblockTime 的值 */ 
		 if ( xTimeToWake < xNextTaskUnblockTime )
		 {
			 xNextTaskUnblockTime = xTimeToWake;
		 }
	 }
 }
```

xTaskIncrementTick()函数 系统时基函数
```c
void xTaskIncrementTick( void )
{
	TCB_t * pxTCB;
	TickType_t xItemValue;
	
	const TickType_t xConstTickCount = xTickCount + 1;
	xTickCount = xConstTickCount;
	
	/* 如果 xConstTickCount 溢出，则切换延时列表 */
	if ( xConstTickCount == ( TickType_t ) 0U ) 
	{ 
		taskSWITCH_DELAYED_LISTS(); /*切换延时列表*/
	} 
	
	/* 最近的延时任务延时到期 */
	if ( xConstTickCount >= xNextTaskUnblockTime ) 
	{ 
		for ( ;; ) 
		{ 
			if ( listLIST_IS_EMPTY( pxDelayedTaskList ) != pdFALSE ) 
			{ 
			/* 延时列表为空，设置 xNextTaskUnblockTime 为可能的最大值 */ 
			xNextTaskUnblockTime = portMAX_DELAY; 
			break; 
			} 
			else /* 延时列表不为空 */
			{ 
				pxTCB = (TCB_t *)listGET_OWNER_OF_HEAD_ENTRY(pxDelayedTaskList); 
				xItemValue = listGET_LIST_ITEM_VALUE(&(pxTCB->xStateListItem) ); 
				/* 直到将延时列表中所有延时到期的任务移除才跳出 for 循环 */
				if ( xConstTickCount < xItemValue ) 
				{ 
					xNextTaskUnblockTime = xItemValue; 
					break; 
				}
				/* 将任务从延时列表移除，消除等待状态 */ 
				( void ) uxListRemove( &( pxTCB->xStateListItem ) ); 
			
				/* 将解除等待的任务添加到就绪列表 */ 
				prvAddTaskToReadyList( pxTCB );
			} 
		} 
	}/* xConstTickCount >= xNextTaskUnblockTime */ 	
	/* 任务切换 */
	portYIELD();
}
```
taskSWITCH_DELAYED_LISTS() 切换延时列表
```c
#define taskSWITCH_DELAYED_LISTS()                                                \
	{                                                                             \
		List_t *pxTemp;                                                           \
		/* The delayed tasks list should be empty when the lists are switched. */ \
		configASSERT((listLIST_IS_EMPTY(pxDelayedTaskList)));                     \
		pxTemp = pxDelayedTaskList;                                               \
		pxDelayedTaskList = pxOverflowDelayedTaskList;                            \
		pxOverflowDelayedTaskList = pxTemp;                                       \
		xNumOfOverflows++;                                                        \
		prvResetNextTaskUnblockTime();                                            \
	}
```
系统时基计数器 xTickCount 溢出后更新xNextTaskUnblockTime（下一个任务的解锁时刻）时间
prvResetNextTaskUnblockTime()
```c
static void prvResetNextTaskUnblockTime( void )
{
	TCB_t *pxTCB;
	
	if ( listLIST_IS_EMPTY( pxDelayedTaskList ) != pdFALSE )
	{
		/* 当前延时列表为空，则设置 xNextTaskUnblockTime 等于最大值 */
		xNextTaskUnblockTime = portMAX_DELAY;
	}
	else
	{
		/* 当前列表不为空，则有任务在延时，则获取当前列表下第一个节点的排序值
		然后将该节点的排序值更新到 xNextTaskUnblockTime */
		( pxTCB ) = ( TCB_t *)listGET_OWNER_OF_HEAD_ENTRY(pxDelayedTaskList);
		xNextTaskUnblockTime = listGET_LIST_ITEM_VALUE(&((pxTCB)->xStateListItem));
	}
 }
```
FreeRTOS 中程序运行的上下文包括：
● 中断服务函数。
● 普通任务。
● 空闲任务。
空闲任务（idle 任务）是 FreeRTOS 系统中没有其他工作进行时自动进入的系统任务。
因为处理器总是需要代码来执行——所以至少要有一个任务处于运行态。FreeRTOS 为了保证这一点，当调用 vTaskStartScheduler()时，调度器会自动创建一个空闲任务，空闲任务是一个非常短小的循环。用户可以通过空闲任务钩子方式，在空闲任务上钩入自己的功能函数。通常这个空闲任务钩子能够完成一些额外的特殊功能，例如系统运行状态的指示，系统省电模式等。除了空闲任务，FreeRTOS 系统还把空闲任务用于一些其他的功能，比如当系统删除一个任务或一个动态任务运行结束时，在执行删除任务的时候，并不会释放任务的内存空间，只会将任务添加到结束列表中，真正的系统资源回收工作在空闲任务完成，空闲任务是唯一一个不允许出现阻塞情况的任务，因为 FreeRTOS 需要保证系统永远都有一个可运行的任务。

对于空闲任务钩子上挂接的空闲钩子函数，它应该满足以下的条件：
● 永远不会挂起空闲任务；
● 不应该陷入死循环，需要留出部分时间用于系统处理系统资源回收。

在系统设计的时候这两个时间候我们都需要考虑，例如，对于事件 A 对应的服务任务Ta，系统要求的实时响应指标是 10ms，而 Ta 的最大运行时间是 1ms，那么 10ms 就是任务Ta 的周期了，1ms 则是任务的运行时间，简单来说任务 Ta 在 10ms 内完成对事件 A 的响应即可。此时，系统中还存在着以 50ms 为周期的另一任务 Tb，它每次运行的最大时间长度是 100us。在这种情况下，即使把任务 Tb 的优先级抬到比 Ta 更高的位置，对系统的实时性指标也没什么影响，因为即使在 Ta 的运行过程中，Tb 抢占了 Ta 的资源，等到 Tb 执行完毕，消耗的时间也只不过是 100us，还是在事件 A 规定的响应时间内(10ms)，Ta 能够安全完成对事件 A 的响应。但是假如系统中还存在任务 Tc，其运行时间为 20ms，假如将 Tc的优先级设置比 Ta 更高，那么在 Ta 运行的时候，突然间被 Tc 打断，等到 Tc 执行完毕，那 Ta 已经错过对事件 A（10ms）的响应了，这是不允许的。所以在我们设计的时候，必须考虑任务的时间，一般来说处理时间更短的任务优先级应设置更高一些。
# 8 消息队列
[[../_resources/freertos/ae7003fb00cb561c16a9f8b57c8c4047_MD5.jpeg|Open: Pasted image 20250109153405.png]]
![[../_resources/freertos/ae7003fb00cb561c16a9f8b57c8c4047_MD5.jpeg]]

在发送消息操作的时候，为了保护数据，当且仅当队列允许入队的时候，发送者才能成功发送息；队列中无可用消息空间时，说明消息队列已满，此时，系统会根据用户指定的阻塞超时时间将任务阻塞，在指定的超时时间内如果还不能完成入队操作，发送消息的任务或者中断服务程序会收到一个错误码 errQUEUE_FULL，然后解除阻塞状态；当然，只有在任务中发送消息才允许进行阻塞状态，而在中断中发送消息不允许带有阻塞机制的，需要调用在中断中发送消息的 API 函数接口，因为发送消息的上下文环境是在中断中，不允许有阻塞的情况。假如有多个任务阻塞在一个消息队列中，那么这些阻塞的任务将按照任务优先级进行排序，优先级高的任务将优先获得队列的访问权。
## 8.1 消息队列控制块

```c
typedef struct QueuePointers
{
	int8_t *pcTail; /* 指向队列存储区域末尾的字节。一旦分配的字节数超过了存储队列项所需的字节数，就会将其用作标记 */
	int8_t *pcReadFrom;	/* 指向最后一个读取队列项的位置。 */
} QueuePointers_t;

typedef struct SemaphoreData
{
	TaskHandle_t xMutexHolder;		 /*< The handle of the task that holds the mutex. */
	UBaseType_t uxRecursiveCallCount;/*< Maintains a count of the number of times a recursive mutex has been recursively 'taken' when the structure is used as a mutex. */
} SemaphoreData_t;

typedef struct QueueDefinition
{
	int8_t *pcHead;    /* 队头 */
	int8_t *pcWriteTo; /* 可用空闲的消息空间 */
	union
	{
		QueuePointers_t xQueue;/* 此结构用作队列时专门需要的数据。*/
		SemaphoreData_t xSemaphore;/* 当此结构用作信号量时，只需要数据。*/
	} u;

	List_t xTasksWaitingToSend;     /* 等待向队列发送消息的任务 */
	List_t xTasksWaitingToReceive;	/* 等待读取队列消息的任务 */

	volatile UBaseType_t uxMessagesWaiting;/* 当前在队列中消息的数量 */
	UBaseType_t uxLength;			/* 队列长度，可以存储消息的数量 */
	UBaseType_t uxItemSize;			/* 每个消息的大小 */

	volatile int8_t cRxLock;        /* 队列上锁后，储存从队列收到的列表项数目，也就是出队的数量，如果队列没有上锁，设置为 queueUNLOCKED */
	volatile int8_t cTxLock;        /* 队列上锁后，储存发送到队列的列表项数目，也就是入队的数量，如果队列没有上锁，设置为 queueUNLOCKED。*/
	#if( ( configSUPPORT_STATIC_ALLOCATION == 1 ) && ( configSUPPORT_DYNAMIC_ALLOCATION == 1 ) )
		uint8_t ucStaticallyAllocated;	/*< Set to pdTRUE if the memory used by the queue was statically allocated to ensure no attempt is made to free the memory. */
	#endif

	#if ( configUSE_QUEUE_SETS == 1 )
		struct QueueDefinition *pxQueueSetContainer;
	#endif

	#if ( configUSE_TRACE_FACILITY == 1 )
		UBaseType_t uxQueueNumber;
		uint8_t ucQueueType;
	#endif
} xQUEUE;

typedef xQUEUE Queue_t;
```
重置消息队列
```c
BaseType_t xQueueGenericReset( QueueHandle_t xQueue, BaseType_t xNewQueue )
{
Queue_t * const pxQueue = xQueue;

	configASSERT( pxQueue );

	taskENTER_CRITICAL();
	{
		pxQueue->u.xQueue.pcTail = pxQueue->pcHead + ( pxQueue->uxLength * pxQueue->uxItemSize ); /*lint !e9016 Pointer arithmetic allowed on char types, especially when it assists conveying intent. */
		pxQueue->uxMessagesWaiting = ( UBaseType_t ) 0U;
		pxQueue->pcWriteTo = pxQueue->pcHead;
		pxQueue->u.xQueue.pcReadFrom = pxQueue->pcHead + ( ( pxQueue->uxLength - 1U ) * pxQueue->uxItemSize ); /*lint !e9016 Pointer arithmetic allowed on char types, especially when it assists conveying intent. */
		pxQueue->cRxLock = queueUNLOCKED;
		pxQueue->cTxLock = queueUNLOCKED;

		if( xNewQueue == pdFALSE )
		{
			/* 如果不是新建一个消息队列，那么之前的消息队列可能阻塞了一些任务，需要将其解除阻塞。如果有发送消息任务被阻塞，那么需要将它恢复，而如果任务是因为读取消息而阻塞，那么重置之后的消息队列也是空的，则无需被恢复。 */
			if( listLIST_IS_EMPTY( &( pxQueue->xTasksWaitingToSend ) ) == pdFALSE )
			{
				if( xTaskRemoveFromEventList( &( pxQueue->xTasksWaitingToSend ) ) != pdFALSE )
				{
					queueYIELD_IF_USING_PREEMPTION();
				}
				else
				{
					mtCOVERAGE_TEST_MARKER();
				}
			}
			else
			{
				mtCOVERAGE_TEST_MARKER();
			}
		}
		else
		{
			/* Ensure the event queues start in the correct state. */
			vListInitialise( &( pxQueue->xTasksWaitingToSend ) );
			vListInitialise( &( pxQueue->xTasksWaitingToReceive ) );
		}
	}
	taskEXIT_CRITICAL();

	/* A value is returned for calling semantic consistency with previous
	versions. */
	return pdPASS;
}
```

**消息队列使用注意事项**
在使用 FreeRTOS 提供的消息队列函数的时候，需要了解以下几点：
1. 使用 xQueueSend()、xQueueSendFromISR()、xQueueReceive()等这些函数之前应先创建需消息队列，并根据队列句柄进行操作。
2. 队列读取采用的是先进先出（FIFO）模式，会先读取先存储在队列中的数据。当然也 FreeRTOS 也支持后进先出（LIFO）模式，那么读取的时候就会读取到后进队列的数据。
3. 在获取队列中的消息时候，我们必须要定义一个存储读取数据的地方，并且该数据区域大小不小于消息大小，否则，很可能引发地址非法的错误。
4. 无论是发送或者是接收消息都是以拷贝的方式进行，如果消息过于庞大，可以将消息的地址作为消息进行发送、接收。
5. 队列是具有自己独立权限的内核对象，并不属于任何任务。所有任务都可以向同一队列写入和读出。一个队列由多任务或中断写入是经常的事，但由多个任务读出倒是用的比较少。

函数
```c
// 创建队列
QueueHandle_t xQueueCreate( UBaseType_t uxQueueLength,UBaseType_t uxItemSize );
//静态创建队列
QueueHandle_t xQueueCreateStatic(
		UBaseType_t uxQueueLength,
		UBaseType_t uxItemSize,
		uint8_t *pucQueueStorageBuffer, //储存队列消息
		StaticQueue_t *pxQueueBuffer    //存储队列控制块
		);
// 删除消息队列
void vQueueDelete( QueueHandle_t xQueue);
// 队列发送
BaseType_t xQueueSend(QueueHandle_t xQueue,const void * pvItemToQueue,TickType_t xTicksToWait);
/* xTicksToWait队列满时，等待队列空闲的最大超时时间。如果队列满并且xTicksToWait 被设置成 0，函数立刻返回。超时时间的单位为系统节拍周期，常量 portTICK_PERIOD_MS 用于辅助计算真实的时间，单位为 ms。如果 INCLUDE_vTaskSuspend 设置成 1，并且指定延时为 portMAX_DELAY 将导致任务挂起（没有超时）. */
// 在中断服务程序中用于向队列尾部发送一个消息。
BaseType_t xQueueSendFromISR(QueueHandle_t xQueue,const void *pvItemToQueue,
BaseType_t *pxHigherPriorityTaskWoken);
// 向队首发送的消息

BaseType_t xQueueSendToFront( QueueHandle_t xQueue,const void * pvItemToQueue,
TickType_t xTicksToWait );
// 在中断服务程序中向消息队列队首发送一个消息。
BaseType_t xQueueSendToFrontFromISR(QueueHandle_t xQueue,const void *pvItemToQueue,
BaseType_t *pxHigherPriorityTaskWoken);

// 用于从一个队列中接收消息，并把接收的消息从队列中删除。
BaseType_t xQueueReceive(QueueHandle_t xQueue,void *pvBuffer,TickType_t xTicksToWait);
// 在中断中从一个队列中接收消息，并从队列中删除该消息。
BaseType_t xQueueReceiveFromISR(QueueHandle_t xQueue,void *pvBuffer,BaseType_t *pxHigherPriorityTaskWoken);

// 用于从一个队列中接收消息，不删除对应的消息。
BaseType_t xQueuePeek( QueueHandle_t xQueue,void *pvBuffer,TickType_t xTicksToWait
);
// 在中断中从一个队列中接收消息，但并不会把消息从该队列中移除。
BaseType_t xQueuePeekFromISR(QueueHandle_t xQueue,void *pvBuffer);

```

## 8.2 `pxHigherPriorityTaskWoken` 的含义

`pxHigherPriorityTaskWoken` 是 FreeRTOS 中用于在中断服务例程（ISR）中进行任务调度的一个参数。它是一个指向 `BaseType_t` 类型变量的指针。其主要作用是指示在 ISR 中是否有一个更高优先级的任务被唤醒。

### 8.2.1 具体说明

- **类型**: `BaseType_t` 是 FreeRTOS 中定义的一种基础数据类型，通常用于表示布尔值（`pdTRUE` 或 `pdFALSE`）。
- **初始值**: 在调用相关函数（如 `xQueueReceiveFromISR` 或 `xQueueSendFromISR`）之前，需要将 `pxHigherPriorityTaskWoken` 指向的变量初始化为 `pdFALSE`。
- **功能**: 当在 ISR 中进行队列操作（如接收或发送数据）时，如果该操作导致某个高优先级的任务可以运行，则 `pxHigherPriorityTaskWoken` 指向的变量会被置为 `pdTRUE`。否则，该变量保持为 `pdFALSE`。
- **用途**: 在 ISR 结束前，可以通过检查 `pxHigherPriorityTaskWoken` 的值来决定是否需要进行任务调度。如果 `pxHigherPriorityTaskWoken` 的值为 `pdTRUE`，则调用 `portYIELD_FROM_ISR()` 函数来进行任务切换，以便尽快执行高优先级的任务。

### 8.2.2 示例代码

以下是一个使用 `pxHigherPriorityTaskWoken` 的示例代码：

```c
void vAnISR(void) { 
	QueueHandle_t xQueue; // 假设已经创建并初始化了队列 
	uint32_t ulReceivedValue; 
	BaseType_t xHigherPriorityTaskWoken; // 初始化 
	pxHigherPriorityTaskWoken xHigherPriorityTaskWoken = pdFALSE; 
	// 从队列中接收数据 
	xQueueReceiveFromISR(xQueue, &ulReceivedValue, &xHigherPriorityTaskWoken); 
	// 检查是否有更高优先级的任务被唤醒 
	if (xHigherPriorityTaskWoken == pdTRUE) { // 进行任务切换 
		portYIELD_FROM_ISR(); 
	} 
}
```
# 9 信号量

用于标记某个事件是否发生，或者标志一下某个东西是否正在被使用，如果是被占用了的或者没发生，我们就不对它进行操作。
抽象的说，信号量就是一个非负整数，所有获取它的任务都会将它的值减1，当这个值为0的时候，资源就没了，后续想要获取这个资源的任务就要阻塞在这里。
信号量的种类：
1. 二值信号量。常用于任务同步与临界资源管理，是任务间、任务与中断间同步的重要手段
2. 计数信号量。常用于事件计数与资源管理，二进制信号量可以被认为是长度为 1 的队列，而计数信号量则可以被认为长度大于 1的队列，信号量使用者依然不必关心存储在队列中的消息，只需关心队列是否有消息即可。
3. 互斥信号量。常用于临界资源的管理，具有优先级继承机制
4. 递归信号量。对于已经获取递归互斥量的任务可以重复获取该递归互斥量，该任务拥有递归信号量的所有权。任务成功获取几次递归互斥量，就要返还几次，在此之前递归互斥量都处于无效状态，其他任务无法获取，只有持有递归信号量的任务才能获取与释放。
**需要注意的是互斥量不能在中断服务函数中使用，因为其特有的优先级继承机制只在任务起作用，在中断的上下文环境毫无意义。**
[[../_resources/freertos/92ec199ae7333d4feb09e2a661b9d62a_MD5.jpeg|Open: Pasted image 20250110160938.png]]
![[../_resources/freertos/92ec199ae7333d4feb09e2a661b9d62a_MD5.jpeg]]

[[../_resources/freertos/33e479aff5652a9661d6b962170aee19_MD5.jpeg|Open: Pasted image 20250110161002.png]]
![[../_resources/freertos/33e479aff5652a9661d6b962170aee19_MD5.jpeg]]
```c
SemaphoreHandle_t xSemaphoreCreateBinary( void );
SemaphoreHandle_t xSemaphoreCreateBinaryStatic(StaticSemaphore_t *pxSemaphoreBuffer );
vSemaphoreCreateBinary( SemaphoreHandle_t xSemaphore )
xSemaphoreCreateCounting
xSemaphoreCreateCountingStatic
xSemaphoreCreateMutex
xSemaphoreCreateMutexStatic
xSemaphoreCreateRecursiveMutex
xSemaphoreCreateRecursiveMutexStatic
vSemaphoreDelete
xSemaphoreGetMutexHolder
xSemaphoreTake
xSemaphoreTakeFromISR
xSemaphoreTakeRecursive
xSemaphoreGive
xSemaphoreGiveRecursive
xSemaphoreGiveFromISR
uxSemaphoreGetCount
```
## 9.1 递归信号量的使用场景
 递归信号量（Recursive Semaphore）是一种特殊的信号量，允许同一个任务多次获取信号量而不导致死锁。它通常用于以下几种场景：
1. 递归函数调用。在递归函数调用中，可能会多次进入同一个临界区。使用普通的互斥锁会导致死锁，因为同一个任务已经持有锁，再次尝试获取锁时会被阻塞。递归信号量允许同一个任务多次获取锁，从而避免死锁。
2. 多层函数调用。在多层函数调用中，可能会在不同的函数层级中进入同一个临界区。使用普通的互斥锁也会导致死锁，因为同一个任务已经持有锁，再次尝试获取锁时会被阻塞。递归信号量允许同一个任务多次获取锁，从而避免死锁。
3. GUI应用程序。在GUI应用程序中，事件处理函数可能会多次进入同一个临界区。例如，一个按钮点击事件可能会触发多个操作，这些操作可能会多次进入同一个临界区。使用递归信号量可以避免死锁，确保程序的正常运行。
4. 复杂的数据结构操作。在操作复杂的数据结构时，可能会多次进入同一个临界区。例如，在操作树形结构时，可能会在不同的节点上进行多次操作，这些操作可能会多次进入同一个临界区。使用递归信号量可以避免死锁，确保数据结构的一致性。
```c
#include "FreeRTOS.h"
#include "task.h"
#include "semphr.h"
 
// 创建递归信号量 
SemaphoreHandle_t xRecursiveSemaphore;
 
void vTaskFunction(void *pvParameters)
{
    for(;;)
    {
        // 获取递归信号量 
        if (xSemaphoreTakeRecursive(xRecursiveSemaphore, portMAX_DELAY) == pdTRUE)
        {
            // 进入临界区 
            // 进行一些操作 
 
            // 释放递归信号量 
            xSemaphoreGiveRecursive(xRecursiveSemaphore);
        }
    }
}
 
int main(void)
{
    // 创建递归信号量 
    xRecursiveSemaphore = xSemaphoreCreateRecursiveMutex();
 
    // 创建任务 
    xTaskCreate(vTaskFunction, "Task 1", configMINIMAL_STACK_SIZE, NULL, tskIDLE_PRIORITY, NULL);
    xTaskCreate(vTaskFunction, "Task 2", configMINIMAL_STACK_SIZE, NULL, tskIDLE_PRIORITY, NULL);
 
    // 启动调度器 
    vTaskStartScheduler();
 
    // 永远不会到达这里 
    for(;;);
}

```
# 10 事件

在某些场合，可能需要多个时间发生了才能进行下一步操作，比如一些危险机器的启动，需要检查各项指标，当指标不达标的时候，无法启动，但是检查各个指标的时候，不能一下子检测完毕啊，所以，需要事件来做统一的等待，当所有的事件都完成了，那么机器才允许启动，这只是事件的其中一个应用。
事件可使用于多种场合，它能够在一定程度上替代信号量，用于任务与任务间，中断与任务间的同步。一个任务或中断服务例程发送一个事件给事件对象，而后等待的任务被唤醒并对相应的事件进行处理。但是它与信号量不同的是，事件的发送操作是不可累计的，而信号量的释放动作是可累计的。事件另外一个特性是，接收任务可等待多种事件，即多个事件对应一个任务或多个任务。同时按照任务等待的参数，可选择是“逻辑或”触发还是“逻辑与”触发。这个特性也是信号量等所不具备的，信号量只能识别单一同步动作，而不能同时等待多个事件的同步。各个事件可分别发送或一起发送给事件对象，而任务可以等待多个事件，任务仅对感兴趣的事件进行关注。当有它们感兴趣的事件发生时并且符合感兴趣的条件，任务将被唤醒并进行后续的处理动作。
```c
xEventGroupCreate
xEventGroupCreateStatic
xEventGroupWaitBits
xEventGroupSetBits
xEventGroupSetBitsFromISR
xEventGroupClearBits
xEventGroupClearBitsFromISR
xEventGroupGetBits
xEventGroupGetBitsFromISR
xEventGroupGetStaticBuffer
xEventGroupSync
vEventGroupDelete
```