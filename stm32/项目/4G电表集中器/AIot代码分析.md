
# 1 main.c
1.  硬件初始化（HAL库初始化、外设初始化、初始化串口DMA）
2. 创建任务默认任务defaultTask没什么用
3. 创建start link任务（主要启动连接服务）StartLinkkitTask
StartLinkkitTask- >link_main(0, NULL);
# 2 link_main LinkSDK测试Demo入口
## 2.1 at_hal_init 硬件AT模组初始化 
### 2.1.1 aiot_install_os_api() 设置系统接口
```c
aiot_os_al_t  g_aiot_freertos_api = {
    .malloc = __malloc,
    .free = __free,
    .time = __time,
    .sleep = __sleep,
    .rand = __rand,
    .mutex_init = __mutex_init,
    .mutex_lock = __mutex_lock,
    .mutex_unlock = __mutex_unlock,
    .mutex_deinit = __mutex_deinit,
};
// 面向应用的系统接口
aiot_os_al_t  *os_api;
```
### 2.1.2 aiot_install_net_api() 设置网络接口
```c
// 初始化网络接口


aiot_net_al_t  g_aiot_net_at_api = {
    .establish = __at_establish,
    .recv = __at_recv,
    .send = __at_send,
    .close = __at_close,
};
// 面向应用层的网络接口
aiot_net_al_t *net_api;
```
### 2.1.3 开启DMA
```c
// 环形队列
typedef struct {
    uint8_t  data[RING_BUFFER_SIZE];
    uint16_t tail;
    uint16_t head;
} uart_ring_buffer_t;

static uart_ring_buffer_t uart_dma_rx_buf = {0};
// DMA 环形输入
HAL_UART_Receive_DMA(&huart1, uart_dma_rx_buf.data, RING_BUFFER_SIZE);
```
### 2.1.4 aiot_at_init();初始化AT组件上下文数据结构体

```c
/*at_module_init*/
int res = aiot_at_init();
1. 设置命令响应时间
2. AT命令发送互斥锁
3. 网络数据发送互斥锁
4. 一行命令接收环形缓冲区
5. 初始化完成标志
```
```c
/**
 * @brief AT组件上下文数据结构体
 */
typedef struct {
    /* 是否初始化 */
    uint8_t is_init;
    /* 模组网络状态 */
    core_ip_status_t        ip_status;
    /* 模组socket连接数据 */
    link_descript_t fd[AIOT_AT_SOCKET_NUM];
    /* AT命令发送超时 */
    uint32_t tx_timeout;
    /* ip数据接收处理上下文 */
    core_at_read_t reader;
    /* 当前处理的命令 */
    const core_at_cmd_item_t *cmd_content;
    /* AT命令返回数据缓存<--接收到一行写入 */
    core_ringbuf_t rsp_rb;
    /* 缓存一行数据 */
    char rsp_buf[AIOT_AT_RSP_LEN_MAXIMUM];
    /* 缓存一行数据偏移 */
    uint32_t rsp_buf_offset;
    /* 串口发送接口，由外部设置*/
    aiot_at_uart_tx_func_t uart_tx_func;
    /* AT指令发送锁*/
    void *cmd_mutex;
    /* 网络数据发送锁*/
    void *send_mutex;
    void *user_data;
    /* AT设备 */
    at_device_t *device;
} core_at_handle_t;
```
### 2.1.5 设置发送接口  （面向串口硬件）
	aiot_at_setopt(AIOT_ATOPT_UART_TX_FUNC, at_uart_send);
### 2.1.6 设置模组   （面向不同网络模块，差异化AT控制指令）
    aiot_at_setopt(AIOT_ATOPT_DEVICE, device);
	1. 模块初始化AT指令列表  module_init_cmd
	2. 网络初始化AT指令列表  ip_init_cmd
	3. 建立scoket指令       open_cmd
	4. 发送数据指令          send_cmd
	5. core_at_recv_data_prefix *recv; 模组主动上报数据标识符，识别到数据上报，会进入数据接收流程
	6. 关闭scoket指令       close_cmd
	7. 错误标识符，控制命令返回数据中，识别到错误标识符会返回执行失败 error_prefix; 
	8. 订阅模组主动上报处理,可选 core_at_urc_item_t *urc_register;
	9. core_at_cmd_item_t *ssl_cmd;开启ssl命令列表 ，可选