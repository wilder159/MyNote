# 1 链表结构
链表根节点结构体(xLIST)：
```c
typedef struct xLIST
{
	UBaseType_t uxNumberOfItems; /*链表计数器，记录节点数量，第零节点不做计数*/
	ListItem_t * pxIndex;        /*链表指针，始终指向链表的第零节点*/
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
	ListItem_t         xEventListItem;         /* 事件链表节点*/
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
	StackType_t * const puxStackBuffer,/*任务栈指针*/
	/*StaticTask_t与TCB_t结构体相同，就是任务控制块指针，目的隐藏TCB_t内部数据结构*/
	StaticTask_t * const pxTaskBuffer 
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
