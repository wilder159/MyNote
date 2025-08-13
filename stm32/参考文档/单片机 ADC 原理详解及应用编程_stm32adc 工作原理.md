> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/m0_73931287/article/details/142071392)

![[../../_resources/单片机 ADC 原理详解及应用编程_stm32adc 工作原理/b8236a5e9852524517bdcec1310eef57_MD5.gif]]

> **本篇文章主要详细讲述****单片机的 ADC 原理和编程应用，****希望我的分享对你有所帮助！**

![[../../_resources/单片机 ADC 原理详解及应用编程_stm32adc 工作原理/426b2eaee7fb52e0f8a167de828f6f29_MD5.png]]

**目录**

 [一、STM32ADC 概述](#%C2%A0%E4%B8%80%E3%80%81STM32ADC%E6%A6%82%E8%BF%B0)

[1、ADC（Analog-to-Digital Converter，模数转换器）](#t0)

[2、STM32 工作原理](#t1) 

 [二、STM32ADC 编程实战](#t2)

[（一）、ADC 开发的寄存器库函数](#t3)

[（二）、ADC 开发的 HAL 库](#t4)

[（三）、实战工程](#t5) 

[1、ADC 单通道采集](#t6)

[2、ADC 多通道采集](#t7)

[三、结语](#t8) 

一、[STM32ADC](https://so.csdn.net/so/search?q=STM32ADC&spm=1001.2101.3001.7020) 概述
---------------------------------------------------------------------------------

### 1、ADC（Analog-to-Digital Converter，模数转换器）

> STM32 的 ADC（Analog-to-Digital Converter，模拟数字转换器）是 STM32 微控制器系列中集成的一种功能强大的模块，**用于将模拟信号转换为数字信号**。STM32 微控制器广泛应用于嵌入式系统，ADC 模块在许多应用场景中都至关重要，例如**传感器读取、信号处理和控制系统。**

> 模拟量（Analog Quantity）是指在一个连续范围内可以取任意值的物理量。这种物理量的值可以是任意的实数，**通常用来表示那些变化是渐进的、连续的特征**，而不是离散的。
> 
> *   **模拟量**：可以在一个连续的范围内变化，例如温度可以是 25.1°C、25.2°C 等，具有无限个可能值。
> *   **数字量**：只能取有限的离散值，例如开关的开（1）和关（0）状态，或者数字传感器读取的值。
> 
> 在许多应用中，**模拟量需要转换为数字量以便进行处理，这通常通过模数转换器（ADC）实现**。转换后，计算机或微控制器能够以数字形式读取和处理这些信号。

![[../../_resources/单片机 ADC 原理详解及应用编程_stm32adc 工作原理/671b373ade036223b997c5e47c9274d9_MD5.png]]

> **ADC 转换模式：** 
> 
> 1.  **单次转换模式（Single Conversion Mode）**：ADC 在每次触发时只进行一次转换。适用于低速、低功耗的应用。
>     
> 2.  **连续转换模式（Continuous Conversion Mode）**：ADC 持续进行转换，适用于需要实时监测的应用，如信号处理和实时数据采集。
>     
> 3.  **扫描模式（Scan Mode）**：ADC 可以在多个通道间进行扫描，每个通道依次进行转换，适合多通道数据采集。
>     
> 4.  **触发模式（Triggered Mode）**：转换过程由外部信号触发，可以是定时器、GPIO 引脚等，适合需要同步数据采集的场景。
>     
> 5.  **差分模式（Differential Mode）**：ADC 测量两个输入信号的差值，提供更高的噪声抗性，适用于高精度测量。
>     
> 6.  **伪差分模式（Pseudo-Differential Mode）**：其中一个输入端连接到地，另一端测量信号，适合简单的差分测量。
>     

> 在 ADC（模数转换器）的应用中，通道组可以分为**规则通道组**（Regular Channel Group）和**注入通道组**（Injected Channel Group）。这两种通道组的主要区别在于它们的工作方式、优先级以及使用场景。 
> 
> **规则通道组（Regular Channel Group）**
> 
> *   **定义**：规则通道组是 ADC 的主要通道组，用于常规的信号采集。它通常用于周期性采集的传感器信号。
>     
> *   **特点**：
>     
>     *   **持续转换**：在连续转换模式下，规则通道组可以在多个通道间进行循环采样。
>     *   **优先级低**：相较于注入通道组，规则通道组的优先级较低，通常用于常规数据的采集。
>     *   **数据存储**：转换结果通常存储在一个数据寄存器中，等待主程序读取。
>     *   **触发方式**：可以通过定时器、外部事件等方式触发采样。
> *   **适用场景**：适用于需要实时采集且对响应时间要求不高的应用，如环境监测、温度传感器等。
>     
> 
> **注入通道组（Injected Channel Group）**
> 
> *   **定义**：注入通道组用于优先级更高的信号采集，通常用于突发事件或特定条件下的快速采样。
>     
> *   **特点**：
>     
>     *   **高优先级**：注入通道组具有较高的优先级，能够在任何时候中断规则通道组的采样进行数据采集。
>     *   **快速响应**：适合快速响应的应用，如检测瞬时信号变化、故障检测等。
>     *   **独立触发**：可以独立于规则通道组进行触发，支持多种触发源（如外部引脚、内部事件等）。
>     *   **多个通道**：通常可以配置多个注入通道，进行快速的信号采样。
> *   **适用场景**：适用于需要在特定条件下迅速采集信号的应用，如运动控制、脉冲信号采集等。
>     

### 2、STM32 工作原理  

> STM32 包含 1~3 个 **12 位**逐次逼近型的模拟数字转换器。每个 ADC 最多有 **18 个通道**，可测量 16 个外部信号源和 2 个内部信号源。各通道的 A/D 转换可以**单次、连续、扫描或间断模式**执行，有**规则通道组**和**注入通道组**，每次转换结束可产生中断。转换的结果可以左对齐或右对齐方式存储在 **16 位数据寄存器**中。
> 
> **1）STM32F103C8T6 有 2 个 ADC，ADC1 和 ADC2。记为 ADCx。**
> 
> **2）每个 ADC 有 18 个通道。**16 个外部信号源测量通道 ADCx_IN0~ADCx_IN15，2 个内部信号源测量通道。信号源引脚对应如下：
> 
> ![[../../_resources/单片机 ADC 原理详解及应用编程_stm32adc 工作原理/c8426f64ccd533d9a27e0dc0e9c94699_MD5.png]]

> ADC 的工作过程一般包括以下几个步骤：
> 
> 1.  **采样**：在某个时间点上对模拟信号进行测量，获取其电压值。
> 2.  **量化**：将模拟信号的电压值与 ADC 的参考电压进行比较，将其转换为相应的数字值。
> 3.  **编码**：将量化后的结果编码为二进制形式，输出给后续的数字电路或处理器。

![[../../_resources/单片机 ADC 原理详解及应用编程_stm32adc 工作原理/44cc72713269f9b9b2f5b980e9003760_MD5.png]]

> **ADC 工作原理思维导图概况如下：**

![[../../_resources/单片机 ADC 原理详解及应用编程_stm32adc 工作原理/bf991de5d0524536c7de252df6e12716_MD5.png]]

> **STM32F103ADC 时钟和采样时间**
> 
> **1、时钟源**
> 
> *   STM32F103 的 ADC 通常由 **APB2 总线时钟**提供时钟。ADC 的最大工作频率为 **14 MHz**。
> *   你需要配置 APB2 时钟（通常通过时钟配置寄存器进行配置）以确保 ADC 的工作频率在合适范围内。
> 
> **2、ADC 时钟设置**
> 
> *   ADC 时钟的配置可以通过配置系统时钟（HSE、HSI 或 PLL）来实现。通常在系统初始化时设置。
> *   在 ADC 模块中，可以通过寄存器设置 ADC 的预分频系数，以确保 ADC 时钟不超过最大工作频率。
> 
> **3、采样时间配置**
> 
> *   STM32F103 的 ADC 允许用户根据输入信号的特性选择不同的采样时间。可选的采样时间设置包括：
>     
>     *   **1.5 个 ADC 时钟周期**
>     *   **7.5 个 ADC 时钟周期**
>     *   **13.5 个 ADC 时钟周期**
>     *   **28.5 个 ADC 时钟周期**
>     *   **41.5 个 ADC 时钟周期**
>     *   **55.5 个 ADC 时钟周期**
>     *   **71.5 个 ADC 时钟周期**
>     *   **239.5 个 ADC 时钟周期**
> *   通过设置 ADC 寄存器中的采样时间字段，可以选择合适的采样时间。例如，对于快速变化的信号，可能选择较短的采样时间；而对于慢变化的信号，较长的采样时间可以提高测量的准确性。
>     
> 
> **4、采样时间与转换时间的关系**
> 
> *   采样时间加上转换时间组成了每次 ADC 转换的总时间。转换时间对于 STM32F103 的 ADC 是固定的，大约为 **1.5 个 ADC 时钟周期**。
>     
> *   因此，总的转换时间公式可以表示为：
>     
>     **总时间 = 采样时间 + 1.5xADC 时钟周期**
>     

 二、STM32ADC 编程实战
----------------

> **在编程实战之前，让我们先来了解一下 ADC 开发相关的库函数。**

### （一）、ADC 开发的寄存器库函数

> **1. ADC 初始化函数**
> 
> **`void ADC_Init(ADC_TypeDef *ADCx, ADC_InitTypeDef *ADC_InitStruct)`**
> 
> *   **功能**：初始化指定的 ADC 外设。
>     
> *   **参数**：
>     
>     *   `ADC_TypeDef *ADCx`：指向 ADC 外设的指针（如`ADC1`、`ADC2`）。
>         
>     *   `ADC_InitTypeDef *ADC_InitStruct`：指向 ADC 初始化结构的指针，包含 ADC 配置参数。
>         
> *   **用途**：设置 ADC 的基本参数，如分辨率、对齐方式、时钟分频等。
>     
> 
> **2. 配置 ADC 通道**
> 
> **`void ADC_RegularChannelConfig(ADC_TypeDef *ADCx, uint32_t Channel, uint32_t Rank, uint32_t SamplingTime)`**
> 
> *   **功能**：配置 ADC 的常规通道。
>     
> *   **参数**：
>     
>     *   `ADC_TypeDef *ADCx`：指向 ADC 外设的指针。
>         
>     *   `uint32_t Channel`：选择要配置的 ADC 通道。
>         
>     *   `uint32_t Rank`：在转换序列中的排名。
>         
>     *   `uint32_t SamplingTime`：采样时间配置。
>         
> *   **用途**：配置 ADC 通道以供后续的采样和转换。
>     
> 
> **3. 启动和停止 ADC 转换**
> 
> **`void ADC_Cmd(ADC_TypeDef *ADCx, FunctionalState NewState)`**
> 
> *   **功能**：启用或禁用指定的 ADC 外设。
>     
> *   **参数**：
>     
>     *   `ADC_TypeDef *ADCx`：指向 ADC 外设的指针。
>         
>     *   `FunctionalState NewState`：功能状态，选择`ENABLE`或`DISABLE`。
>         
> *   **用途**：控制 ADC 的开启和关闭。
>     
> 
> `void ADC_StartConversion(ADC_TypeDef *ADCx)`
> 
> *   **功能**：开始 ADC 的转换。
>     
> *   **参数**：
>     
>     *   `ADC_TypeDef *ADCx`：指向 ADC 外设的指针。
>         
> *   **用途**：启动 ADC 转换过程。
>     
> 
> **4. 读取 ADC 转换结果**
> 
> **`uint16_t ADC_GetConversionValue(ADC_TypeDef *ADCx)`**
> 
> *   **功能**：获取 ADC 的转换结果。
>     
> *   **参数**：
>     
>     *   `ADC_TypeDef *ADCx`：指向 ADC 外设的指针。
>         
> *   **返回值**：返回 ADC 转换后的数值。
>     
> *   **用途**：读取转换完成后的结果。
>     
> 
> **5. 配置 DMA 支持**
> 
> **`void ADC_DMACmd(ADC_TypeDef *ADCx, FunctionalState NewState)`**
> 
> *   **功能**：启用或禁用 ADC 的 DMA 功能。
>     
> *   **参数**：
>     
>     *   `ADC_TypeDef *ADCx`：指向 ADC 外设的指针。
>         
>     *   `FunctionalState NewState`：功能状态，选择`ENABLE`或`DISABLE`。
>         
> *   **用途**：在使用 DMA 传输 ADC 数据时配置 DMA。
>     
> 
> **6. 中断支持**
> 
> **`void ADC_ITConfig(ADC_TypeDef *ADCx, uint32_t ADC_IT, FunctionalState NewState)`**
> 
> *   **功能**：启用或禁用 ADC 中断。
>     
> *   **参数**：
>     
>     *   `ADC_TypeDef *ADCx`：指向 ADC 外设的指针。
>         
>     *   `uint32_t ADC_IT`：选择中断源。
>         
>     *   `FunctionalState NewState`：功能状态，选择`ENABLE`或`DISABLE`。
>         
> *   **用途**：控制 ADC 的中断行为。
>     
> 
> **7. 中断回调函数**
> 
> 在使用中断时，需要定义回调函数以处理 ADC 转换完成的事件。
> 
> ```
> void ADC1_2_IRQHandler(void) {
>     if (ADC_GetITStatus(ADC1, ADC_IT_EOC) != RESET) {
>         // 处理ADC转换完成
>         uint16_t adcValue = ADC_GetConversionValue(ADC1);
>         // 清除中断标志
>         ADC_ClearITPendingBit(ADC1, ADC_IT_EOC);
>     }
> }
> ```

![[../../_resources/单片机 ADC 原理详解及应用编程_stm32adc 工作原理/27d9396560b3fb0fb2d9e5d5f7a3f4b6_MD5.png]]

### （二）、ADC 开发的 HAL 库

> **1. ADC 初始化函数**
> 
> **`HAL_ADC_Init()`**
> 
> *   功能：初始化 ADC 外设。
>     
> *   参数：
>     
>     *   `ADC_HandleTypeDef *hadc`：指向 ADC 句柄的指针，结构体中包含 ADC 的配置参数。
>         
> *   返回值：HAL 库返回状态，通常为`HAL_OK`（成功）或错误代码。
>     
> *   用途：设置 ADC 的基本参数，如分辨率、对齐方式、扫描模式等。
>     
> 
> **2. ADC 通道配置函数**
> 
> **`HAL_ADC_ConfigChannel()`**
> 
> *   功能：配置指定的 ADC 通道。
>     
> *   参数：
>     
>     *   `ADC_HandleTypeDef *hadc`：指向 ADC 句柄的指针。
>         
>     *   `ADC_ChannelConfTypeDef *sConfig`：指向通道配置结构的指针，包含通道选择、采样时间等。
>         
> *   返回值：HAL 库返回状态，通常为`HAL_OK`（成功）或错误代码。
>     
> *   用途：设置通道的采样时间和输入模式等参数。
>     
> 
> **3. 启动和停止 ADC 转换**
> 
> **`HAL_ADC_Start()`**
> 
> *   功能：启动 ADC 转换。
>     
> *   参数：
>     
>     *   `ADC_HandleTypeDef *hadc`：指向 ADC 句柄的指针。
>         
> *   返回值：HAL 库返回状态。
>     
> *   用途：使 ADC 开始进行转换。
>     
> 
> `HAL_ADC_Stop()`
> 
> *   功能：停止 ADC 转换。
>     
> *   参数：
>     
>     *   `ADC_HandleTypeDef *hadc`：指向 ADC 句柄的指针。
>         
> *   返回值：HAL 库返回状态。
>     
> *   用途：结束 ADC 转换过程，释放资源。
>     
> 
> **4. 读取 ADC 转换结果**
> 
> **`HAL_ADC_PollForConversion()`**
> 
> *   功能：等待 ADC 转换完成（轮询方式）。
>     
> *   参数：
>     
>     *   `ADC_HandleTypeDef *hadc`：指向 ADC 句柄的指针。
>         
>     *   `uint32_t Timeout`：等待超时的时间（单位：毫秒）。
>         
> *   返回值：HAL 库返回状态，通常为`HAL_OK`（成功）或超时错误代码。
>     
> *   用途：在转换过程中进行轮询，直到转换完成。
>     
> 
> **`HAL_ADC_GetValue()`**
> 
> *   功能：获取 ADC 转换结果。
>     
> *   参数：
>     
>     *   `ADC_HandleTypeDef *hadc`：指向 ADC 句柄的指针。
>         
> *   返回值：ADC 的转换结果。
>     
> *   用途：在转换完成后读取结果值。
>     
> 
> **5. DMA 支持**
> 
> **`HAL_ADC_Start_DMA()`**
> 
> *   功能：启动 ADC 转换并通过 DMA 传输数据。
>     
> *   参数：
>     
>     *   `ADC_HandleTypeDef *hadc`：指向 ADC 句柄的指针。
>         
>     *   `uint32_t *pData`：指向存储结果的缓冲区指针。
>         
>     *   `uint32_t Length`：缓冲区的长度。
>         
> *   返回值：HAL 库返回状态。
>     
> *   用途：使用 DMA 提高数据传输效率。
>     
> 
> **6. 中断支持**
> 
> **`HAL_ADC_Start_IT()`**
> 
> *   功能：启动 ADC 转换并使能中断。
>     
> *   参数：
>     
>     *   `ADC_HandleTypeDef *hadc`：指向 ADC 句柄的指针。
>         
> *   返回值：HAL 库返回状态。
>     
> *   用途：在需要中断处理的应用中使用。
>     
> 
> **7. 中断回调函数**
> 
> **`HAL_ADC_ConvCpltCallback()`**
> 
> *   功能：ADC 转换完成时的回调函数。
>     
> *   参数：
>     
>     *   `ADC_HandleTypeDef *hadc`：指向 ADC 句柄的指针。
>         
> *   用途：在此函数中处理转换结果。
>     

![[../../_resources/单片机 ADC 原理详解及应用编程_stm32adc 工作原理/7c398c9374e54b274b0c7fd9601ad94a_MD5.png]]

### （三）、实战工程 

#### **1、ADC 单通道采集**

```
#include "stm32f10x.h"                  // 引入 STM32F10x 设备头文件，包含特定于设备的定义和功能
 
// 初始化 ADC (模数转换器)
void AD_Init(void)
{
    // 使能 ADC1 的时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
    // 使能 GPIOA 的时钟，以便配置 GPIO 引脚
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    // 配置 ADC 时钟为 PCLK2 的 1/6
    RCC_ADCCLKConfig(RCC_PCLK2_Div6);
    
    // 定义一个 GPIO 初始化结构体
    GPIO_InitTypeDef GPIO_InitStructure;
    // 设置引脚模式为模拟输入
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
    // 设置要配置的引脚为 PA0
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
    // 设置 GPIO 引脚的速度为 50MHz
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    // 初始化 GPIOA
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    // 配置 ADC 的常规通道，设置通道为 ADC_Channel_0，序列为 1，采样时间为 55.5 个周期
    ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_55Cycles5);
    
    // 定义一个 ADC 初始化结构体
    ADC_InitTypeDef ADC_InitStructure;
    // 设置 ADC 工作模式为独立模式
    ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;
    // 设置数据对齐方式为右对齐
    ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
    // 设置外部触发转换为无
    ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
    // 设置连续转换模式为禁用
    ADC_InitStructure.ADC_ContinuousConvMode = DISABLE;
    // 设置扫描模式为禁用
    ADC_InitStructure.ADC_ScanConvMode = DISABLE;
    // 设置转换通道数量为 1
    ADC_InitStructure.ADC_NbrOfChannel = 1;
    // 初始化 ADC1
    ADC_Init(ADC1, &ADC_InitStructure);
    
    // 使能 ADC1
    ADC_Cmd(ADC1, ENABLE);
    
    // 复位 ADC 校准寄存器
    ADC_ResetCalibration(ADC1);
    // 等待复位完成
    while (ADC_GetResetCalibrationStatus(ADC1) == SET);
    // 开始 ADC 校准
    ADC_StartCalibration(ADC1);
    // 等待校准完成
    while (ADC_GetCalibrationStatus(ADC1) == SET);
}
 
// 获取 ADC 转换值的函数
uint16_t AD_GetValue(void)
{
    // 启动软件触发的 ADC 转换
    ADC_SoftwareStartConvCmd(ADC1, ENABLE);
    // 等待转换完成标志位设置
    while (ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC) == RESET);
    // 返回 ADC 转换结果
    return ADC_GetConversionValue(ADC1);
}
```

![[../../_resources/单片机 ADC 原理详解及应用编程_stm32adc 工作原理/e2e0b97fc34c7ebce1fefd45480927e5_MD5.png]]

#### 2、ADC 多通道采集

```
#include "stm32f10x.h"                  // 引入 STM32F10x 设备头文件，包含特定于设备的定义和功能
 
// 初始化 ADC (模数转换器)
void AD_Init(void)
{
    // 使能 ADC1 的时钟，确保 ADC1 可以正常工作
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
    // 使能 GPIOA 的时钟，以便配置 GPIO 引脚用于 ADC
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    // 配置 ADC 时钟为 PCLK2 的 1/6
    RCC_ADCCLKConfig(RCC_PCLK2_Div6);
    
    // 定义一个 GPIO 初始化结构体，用于设置 GPIO 的模式和速度
    GPIO_InitTypeDef GPIO_InitStructure;
    // 设置 GPIO 模式为模拟输入 (AIN)
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
    // 设置要配置的引脚为 PA0, PA1, PA2 和 PA3
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_2 | GPIO_Pin_3;
    // 设置 GPIO 引脚的速度为 50MHz
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    // 初始化 GPIOA，应用上面的配置
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    // 定义一个 ADC 初始化结构体，用于配置 ADC 参数
    ADC_InitTypeDef ADC_InitStructure;
    // 设置 ADC 工作模式为独立模式
    ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;
    // 设置数据对齐方式为右对齐
    ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
    // 设置外部触发转换为无（软件触发）
    ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
    // 设置连续转换模式为禁用
    ADC_InitStructure.ADC_ContinuousConvMode = DISABLE;
    // 设置扫描模式为禁用
    ADC_InitStructure.ADC_ScanConvMode = DISABLE;
    // 设置转换通道数量为 1
    ADC_InitStructure.ADC_NbrOfChannel = 1;
    // 初始化 ADC1，应用上面的配置
    ADC_Init(ADC1, &ADC_InitStructure);
    
    // 使能 ADC1
    ADC_Cmd(ADC1, ENABLE);
    
    // 复位 ADC 校准寄存器
    ADC_ResetCalibration(ADC1);
    // 等待复位完成
    while (ADC_GetResetCalibrationStatus(ADC1) == SET);
    // 开始 ADC 校准
    ADC_StartCalibration(ADC1);
    // 等待校准完成
    while (ADC_GetCalibrationStatus(ADC1) == SET);
}
 
// 获取指定 ADC 通道的转换值
uint16_t AD_GetValue(uint8_t ADC_Channel)
{
    // 配置 ADC 通道，设置通道、序列和采样时间
    ADC_RegularChannelConfig(ADC1, ADC_Channel, 1, ADC_SampleTime_55Cycles5);
    // 启动软件触发的 ADC 转换
    ADC_SoftwareStartConvCmd(ADC1, ENABLE);
    // 等待转换完成标志位设置
    while (ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC) == RESET);
    // 返回 ADC 转换结果
    return ADC_GetConversionValue(ADC1);
}
```

![[../../_resources/单片机 ADC 原理详解及应用编程_stm32adc 工作原理/3ddcaa10b0d673c607389c041d401467_MD5.png]]

三、结语 
-----

> 关于 [STM32 单片机](https://so.csdn.net/so/search?q=STM32%E5%8D%95%E7%89%87%E6%9C%BA&spm=1001.2101.3001.7020)的 ADC 原理及编程实现就分享到此了，希望我的分享对你有所帮助！
> 
> **关于以上工程的源代码，大家可以私信我，收到后我会第一时间回复！**

![[../../_resources/单片机 ADC 原理详解及应用编程_stm32adc 工作原理/0570b98c5afda78efa3bc27b8b322c2a_MD5.jpg]]

![[../../_resources/单片机 ADC 原理详解及应用编程_stm32adc 工作原理/dfcac7f34c2ad66dbedb3aa2c412b97e_MD5.gif]]