---
title: STM32的ADC学习
tag: SPI
date: 2024-09-12
categories: STM32
index_img: 
---

# STM32的ADC学习

## 基本知识

ADC一共有三个ADC设备，这三个设备可以看成三个不一样的工人，他们可以同时，做各自的事情，当然如果他们三个人一起做一件事情，这件事情也会做的更快，做的更好，然后每个工人身上又有十几个通道，这十几个通道可以看出十几件不同的事情，干完这件事情才能干下一件事情，所以通道之间需要排队完成。

## ADC的配置内容

ADC的配置分为共同配置和ADC个体配置，在配置完ADC硬件后，还需要针对每个通道进行配置

### ADC_CommonInitTypeDef

共同配置主要配置所有ADC的采集模式、采集频率、传输模式和采集间隔时间，共同配置是在使用两个ADC硬件以上的才会选择进行配置，如果是单ADC的话，是不进行配置的

ADC的不同模式如下

```
ADC_Mode_Independent (0x00000000):

独立模式。每个 ADC 独立工作，互不干扰。
ADC_DualMode_RegSimult_InjecSimult (0x00000001):

双重模式，在常规和注入模式下同时进行采集。
ADC_DualMode_RegSimult_AlterTrig (0x00000002):

双重模式，常规采集同时进行，交替触发采集。
ADC_DualMode_InjecSimult (0x00000005):

双重模式，注入采集同时进行。
ADC_DualMode_RegSimult (0x00000006):

双重模式，常规采集同时进行。
ADC_DualMode_Interl (0x00000007):

双重模式，交错模式。两个 ADC 交替进行常规采集，增加数据采集速度。
ADC_DualMode_AlterTrig (0x00000009):

双重模式，交替触发模式。ADC1 和 ADC2 在不同的时间点进行采集。
ADC_TripleMode_RegSimult_InjecSimult (0x00000011):

三重模式，常规和注入采集同时进行。
ADC_TripleMode_RegSimult_AlterTrig (0x00000012):

三重模式，常规采集同时进行，交替触发采集。
ADC_TripleMode_InjecSimult (0x00000015):

三重模式，注入采集同时进行。
ADC_TripleMode_RegSimult (0x00000016):

三重模式，常规采集同时进行。
ADC_TripleMode_Interl (0x00000017):

三重模式，交错模式。三个 ADC 交替进行常规采集，增加采样速率。
ADC_TripleMode_AlterTrig (0x00000019):

三重模式，交替触发模式。ADC1、ADC2 和 ADC3 在不同时间点进行采集。
```

ADC的最大频率是36MHz，所以尽可能分频的时候将频率接近最大频率

单ADC和单通道也需要配置ADC_CommonInitTypeDef，并且在模式上选择独立模式

如果是单ADC和单通道的模式，可以不使能该模式。

```
typedef struct 
{
  uint32_t ADC_Mode;                      /*!< 配置ADC在独立模式或多ADC模式下运行。
                                               该参数的取值可以是 @ref ADC_Common_mode 定义的值。 */                                              
  uint32_t ADC_Prescaler;                 /*!< 选择ADC时钟的频率，该时钟是所有ADC通用的。
                                               该参数的取值可以是 @ref ADC_Prescaler 定义的值。 */
  uint32_t ADC_DMAAccessMode;             /*!< 配置多ADC模式下的直接存储器访问（DMA）模式。
                                               该参数的取值可以是 @ref ADC_Direct_memory_access_mode_for_multi_mode 定义的值。 */
  uint32_t ADC_TwoSamplingDelay;          /*!< 配置两次采样阶段之间的延迟。
                                               该参数的取值可以是 @ref ADC_delay_between_2_sampling_phases 定义的值。 */
  
} ADC_CommonInitTypeDef;
```

### ADC_InitTypeDef

个体配置主要配置ADC的采集精度、是否扫描、是否连续读取、是否外部触发、外部触发边沿选择、外部触发模式、采集通道数、数据对齐方式，不管是单ADC，还是多ADC，都需要配置每个ADC的个体配置

STM32F407的ADC读取精度是12位（即 12-bit）。ADC 分辨率决定了转换结果的精度，12 位分辨率意味着 ADC 可以将模拟信号转换为 4096 个不同的数字值（2^12）。当然也就是说处理的时间是12个ADC时钟。

扫描模式的作用是可以针对多个通道进行扫描，如果不开启的话只能在同一个通道上面进行读取

连续转换模式。连续转换模式使得 ADC 在完成一个转换后自动开始下一个转换，而无需额外的触发信号。这样可以实现不断更新的 ADC 值。

外部触发转换的边缘，设置外部触发信号的边缘，以控制 ADC 转换的启动时机。主要就是上升沿、下降沿、两个都用或者不启用

选择外部触发信号，外部触发的信号主要有定时器、中断触发，并且这些触发的话每个设备都是固定的，在头文件中宏定义会全部定义完全。内部触发意味着 ADC 转换的启动由微控制器内部的触发源控制，例如内部定时器、软件触发或其他内部事件，内部触发的话是比较快，比较简单，而且可以根据软件触发的方式进行随意检测，但是硬件检测的方式对于周期的控制比较精确，但是比较麻烦。

设置 ADC 转换结果的数据对齐方式。`ADC_DataAlign_Right` 表示转换结果右对齐，即数据的低位在数据寄存器的低位部分，高位在高位部分。另一种对齐方式是左对齐

```
typedef struct
{
  uint32_t ADC_Resolution;                /*!< 配置ADC的分辨率（dual mode）。
                                               该参数的取值可以是 @ref ADC_resolution 定义的值。 */                                   
  FunctionalState ADC_ScanConvMode;       /*!< 指定转换是在扫描模式（多个通道）还是单通道模式下进行。
                                               该参数可以设置为 ENABLE 或 DISABLE。 */ 
  FunctionalState ADC_ContinuousConvMode; /*!< 指定转换是在连续模式还是单次模式下进行。
                                               该参数可以设置为 ENABLE 或 DISABLE。 */
  uint32_t ADC_ExternalTrigConvEdge;      /*!< 选择外部触发的边沿，并启用常规组的触发。
                                               该参数的取值可以是 @ref ADC_external_trigger_edge_for_regular_channels_conversion 定义的值。 */
  uint32_t ADC_ExternalTrigConv;          /*!< 选择用于触发常规组转换的外部事件。
                                               该参数的取值可以是 @ref ADC_extrenal_trigger_sources_for_regular_channels_conversion 定义的值。 */
  uint32_t ADC_DataAlign;                 /*!< 指定ADC数据对齐方式（左对齐或右对齐）。
                                               该参数的取值可以是 @ref ADC_data_align 定义的值。 */
  uint8_t  ADC_NbrOfConversion;           /*!< 指定使用序列器进行常规通道组转换的次数。
                                               该参数的取值范围为 1 到 16。 */
} ADC_InitTypeDef;
```

最后还需要选择所需通道和采样时长，并使能所有ADC

```
// 选择ADC的通道和采样优先级以及采样时间
ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_15Cycles );	

// 使能DMA请求，这个是单通道的使能
ADC_DMARequestAfterLastTransferCmd(ADC1, ENABLE);

// 使能DMA
ADC_DMACmd(ADC1, ENABLE);
	
// 使能ADC
ADC_Cmd(ADC1, ENABLE);

// 软件启动ADC 
ADC_SoftwareStartConv(ADC1);

// 对是否启动进行校验
while(!ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC ));
```

## DMA配置内容

DMA_PeripheralBaseAddr是ADC的数据寄存器地址，如果不是交替模式，地址是(uint32_t)&(ADC1->DR)，如果是交替模式，地址是(uint32_t)&(ADC1->CDR)

DMA_Memory0BaseAddr的定义上一定要定义成下面这种形式，volatile关键字代表数据是每次输入时都要进行重新设定，如果全局变量的话，需要加上extern关键字，以此来实现全局使用

```
volatile uint16_t ConvertedValueTab;
```

DMA_PeripheralDataSize的字节都是半个字也就是16字节，虽然我们是12位精度，但是为了数据的内存对齐，所以我们选择16字节，如果是交替模式的话，超过16字节就需要选择32字节进行接收

DMA_MemoryInc是储存区递增模式，如果是多个通道一起接收，就需要打开自动增长，也就是数组的数据挨个遍历存储，如果是单ADC单通道就没必要了

DMA_Mode_Circular是循环读取数据，如果你想要一次软件启动ADC读一次的话，这个不进行使能，如果只想在配置里面只使能一次的话，这个就需要使能

DMA_Priority是优先级排列，这个通常是多个外设的时候进行设置的

DMA_PeripheralBurst突发模式，也就是告诉外设每次凑齐多少个字节，然后再给我发送

DMA_FIFOMode模式就是DMA内部的一个数据缓存区，这个缓存区最大16字节，可以选择达到缓存区多少的时候再进行发送，也可以不使用，直连定义缓存区

DMA_Init(DMA2_Stream0,&DMA_InitStruct)这个初始化一定要注意选择DMA2_Stream0和DMA_Channel_0，这个一定要匹配上，否则会出现无法读取的情况

```
typedef struct
{
  uint32_t DMA_Channel;            /*!< 指定用于 Stream 的 DMA 通道。
                                        该参数的取值可以是 @ref DMA_channel 的定义。*/
 
  uint32_t DMA_PeripheralBaseAddr; /*!< 指定 DMAy Streamx 的外设基地址。*/

  uint32_t DMA_Memory0BaseAddr;    /*!< 指定 DMAy Streamx 的内存0基地址。
                                        当双缓冲模式未启用时，此内存为默认使用的内存。*/

  uint32_t DMA_DIR;                /*!< 指定数据传输方向（从内存到外设、从内存到内存、或从外设到内存）。
                                        该参数的取值可以是 @ref DMA_data_transfer_direction 的定义。*/

  uint32_t DMA_BufferSize;         /*!< 指定指定 Stream 的缓冲区大小（以数据单元为单位）。
                                        数据单元等于根据传输方向在 DMA_PeripheralDataSize 或 DMA_MemoryDataSize 中设置的配置。*/

  uint32_t DMA_PeripheralInc;      /*!< 指定是否递增外设地址寄存器。
                                        该参数的取值可以是 @ref DMA_peripheral_incremented_mode 的定义。*/

  uint32_t DMA_MemoryInc;          /*!< 指定是否递增内存地址寄存器。
                                        该参数的取值可以是 @ref DMA_memory_incremented_mode 的定义。*/

  uint32_t DMA_PeripheralDataSize; /*!< 指定外设数据宽度。
                                        该参数的取值可以是 @ref DMA_peripheral_data_size 的定义。*/

  uint32_t DMA_MemoryDataSize;     /*!< 指定内存数据宽度。
                                        该参数的取值可以是 @ref DMA_memory_data_size 的定义。*/

  uint32_t DMA_Mode;               /*!< 指定 DMAy Streamx 的操作模式（循环模式或普通模式）。
                                        该参数的取值可以是 @ref DMA_circular_normal_mode 的定义。
                                        @note 如果配置为内存到内存的传输，则无法使用循环缓冲模式。*/

  uint32_t DMA_Priority;           /*!< 指定 DMAy Streamx 的软件优先级。
                                        该参数的取值可以是 @ref DMA_priority_level 的定义。*/

  uint32_t DMA_FIFOMode;           /*!< 指定是否为 Stream 使用 FIFO 模式或直接模式。
                                        该参数的取值可以是 @ref DMA_fifo_direct_mode 的定义。
                                        @note 如果配置为内存到内存传输，则无法使用直接模式（FIFO 模式禁用）。*/

  uint32_t DMA_FIFOThreshold;      /*!< 指定 FIFO 阈值级别。
                                        该参数的取值可以是 @ref DMA_fifo_threshold_level 的定义。*/

  uint32_t DMA_MemoryBurst;        /*!< 指定内存传输的突发传输配置。
                                        它指定在单个不可中断事务中传输的数据量。
                                        该参数的取值可以是 @ref DMA_memory_burst 的定义。
                                        @note 仅当启用了地址递增模式时，才可能使用突发模式。*/

  uint32_t DMA_PeripheralBurst;    /*!< 指定外设传输的突发传输配置。
                                        它指定在单个不可中断事务中传输的数据量。
                                        该参数的取值可以是 @ref DMA_peripheral_burst 的定义。
                                        @note 仅当启用了地址递增模式时，才可能使用突发模式。*/

} DMA_InitTypeDef;
```

### DMA传输流程图

![image.png](https://s2.loli.net/2024/09/15/lQoOgFdmNfSWJ8P.png)

![image.png](https://s2.loli.net/2024/09/15/Bw6jIWZN8kOYtiD.png)

## GPIO配置

```
	GPIO_InitTypeDef GPIO_InitStructure;
	
	RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOA, ENABLE);
	
	// 同时配置四个ADC引脚口
	// GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_2 | GPIO_Pin_3;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	
	// 要配置成模拟输入，而不是简单的输入模式
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AN;
	
	// GPIO_Speed 参数用于控制引脚在输出时的驱动能力和切换速度（引脚状态的上升和下降时间），也不进行配置
	// GPIO_InitStructure.GPIO_Speed = GPIO_Speed_100MHz;
	
	// 只适用于输出模式，所以不配置
	// GPIO_InitStructure.GPIO_OType = GPIO_OType_PP; 
	
	// 输入模式不进行上下拉，不然数据不准
	GPIO_InitStructure.GPIO_PuPd = GPIO_PuPd_NOPULL;
	// 写入配置
	GPIO_Init(GPIOA, &GPIO_InitStructure);
```

## ADC配置顺序

ADC的使用方式有很多种，如果比较简单的话就直接进行单ADC和单通道直接使用，也可以多通道单ADC使用，或者多通道多ADC使用，这几种方法可以根据不同的配置进行实现

### 单ADC单通道

```
// 单通道接收缓存区
volatile uint16_t ADC_1_1_data;

void ADC_1_Channel_1_Config(void)
{
	// 定义结构体
	ADC_InitTypeDef ADC_InitStructure;
	ADC_CommonInitTypeDef ADC_CommonInitStructure;
	DMA_InitTypeDef DMA_InitStruct;
	GPIO_InitTypeDef GPIO_InitStructure;
	
	// 打开时钟
	RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOA, ENABLE);
	// 配置引脚
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	// 要配置成模拟输入，而不是简单的输入模式
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AN;
	// GPIO_Speed 参数用于控制引脚在输出时的驱动能力和切换速度（引脚状态的上升和下降时间），也不进行配置
	// GPIO_InitStructure.GPIO_Speed = GPIO_Speed_100MHz;
	// 只适用于输出模式，所以不配置
	// GPIO_InitStructure.GPIO_OType = GPIO_OType_PP; 
	// 输入模式不进行上下拉，不然数据不准
	GPIO_InitStructure.GPIO_PuPd = GPIO_PuPd_NOPULL;
	// 写入配置
	GPIO_Init(GPIOA, &GPIO_InitStructure);

	// 打开DMA2时钟
	RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_DMA2, ENABLE);
	// 选择通道0
	DMA_InitStruct.DMA_Channel = DMA_Channel_0;
	// 定义数据的去向和大小
	DMA_InitStruct.DMA_PeripheralBaseAddr =(uint32_t)&(ADC1->DR);// 单通道采集使用CSR通用寄存器，单通道交替采集使用CDR数据寄存器
	// 定义接受缓存区
	DMA_InitStruct.DMA_Memory0BaseAddr = (uint32_t)&ADC_ONE_data;
	// 选择DMA方向
	DMA_InitStruct.DMA_DIR = DMA_DIR_PeripheralToMemory;
	// 选择传输数据量，与ADC传输通道一致
	DMA_InitStruct.DMA_BufferSize = 1;
	// 设定外设的循环模式，不使能，外设只有一个地址
	DMA_InitStruct.DMA_PeripheralInc = DMA_PeripheralInc_Disable;
	// 定义外设传输的字节大小
	DMA_InitStruct.DMA_PeripheralDataSize = DMA_PeripheralDataSize_HalfWord;
	// 设定存储的循环模式，不使能，单通道只有一个数据
	DMA_InitStruct.DMA_MemoryInc = DMA_MemoryInc_Disable;
	// 定义存储的字节大小
	DMA_InitStruct.DMA_MemoryDataSize = DMA_MemoryDataSize_HalfWord;
	// 要循环读取数据
	DMA_InitStruct.DMA_Mode = DMA_Mode_Circular;
	// 数据的软优先级设置
	DMA_InitStruct.DMA_Priority = DMA_Priority_High;
	// 不使用突发模式，直接改为单字节
	DMA_InitStruct.DMA_PeripheralBurst = DMA_PeripheralBurst_Single;
	// 不使用突发模式，直接改为单字节
	DMA_InitStruct.DMA_MemoryBurst = DMA_MemoryBurst_Single;
	// 不适用FIFO模式
	DMA_InitStruct.DMA_FIFOMode = DMA_FIFOMode_Disable; // FIFO模式是DMA自定义缓存区，达到标准后，一次性发送给CPU，最大为4*4=16字节
	// DMA结构体初始化
	DMA_Init(DMA2_Stream0,&DMA_InitStruct);
	// DMA使能
	DMA_Cmd(DMA2_Stream0,ENABLE);
	// 使能ADC时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
	
	// ADC复位
	ADC_DeInit();
	// 单ADC选择1，也可以不使能
	ADC_CommonInitStructure.ADC_DMAAccessMode=ADC_DMAAccessMode_1;
	// 单ADC单通道选择独立模式
	ADC_CommonInitStructure.ADC_Mode=ADC_Mode_Independent;
	// 四分频，168/4 = 42MHz
	ADC_CommonInitStructure.ADC_Prescaler=ADC_Prescaler_Div4;
	// 两个ADC采样间隔时间选择20个ADC频率
	ADC_CommonInitStructure.ADC_TwoSamplingDelay=ADC_TwoSamplingDelay_20Cycles;
	// 添加配置
	ADC_CommonInit(&ADC_CommonInitStructure);
	
	// 12位精度
	ADC_InitStructure.ADC_Resolution = ADC_Resolution_12b;
	// 单通道不开启连续扫描模式
	ADC_InitStructure.ADC_ScanConvMode = DISABLE;
	// 使能连续读取模式
	ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;
	// 不进行外部边沿触发
	ADC_InitStructure.ADC_ExternalTrigConvEdge = ADC_ExternalTrigConvEdge_None;
	// 数据右对齐
	ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
	// 开启通道数为1
	ADC_InitStructure.ADC_NbrOfConversion = 1;
	// 写入配置
	ADC_Init(ADC1, &ADC_InitStructure);
	
	// 选择ADC、通道、优先级和采样时间
	ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_15Cycles );	
	// 使能DMA请求
  ADC_DMARequestAfterLastTransferCmd(ADC1, ENABLE);
	// 使能DMA
	ADC_DMACmd(ADC1, ENABLE);
	// 使能ADC
	ADC_Cmd(ADC1, ENABLE);
	// 进行使能标志位检测
	ADC_SoftwareStartConv(ADC1);
	while(!ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC ));
}
```

### 单ADC多通道

```
// 多通道接收缓存区
volatile uint16_t ADC_1_4_data[4];

void ADC_1_Channel_4_Config(void)
{
	// 定义结构体
	ADC_InitTypeDef ADC_InitStructure;
	ADC_CommonInitTypeDef ADC_CommonInitStructure;
	DMA_InitTypeDef DMA_InitStruct;
	GPIO_InitTypeDef GPIO_InitStructure;
	
	// 打开时钟
	RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOA, ENABLE);
	// 配置引脚
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_2 | GPIO_Pin_3;
	// 要配置成模拟输入，而不是简单的输入模式
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AN;
	// GPIO_Speed 参数用于控制引脚在输出时的驱动能力和切换速度（引脚状态的上升和下降时间），也不进行配置
	// GPIO_InitStructure.GPIO_Speed = GPIO_Speed_100MHz;
	// 只适用于输出模式，所以不配置
	// GPIO_InitStructure.GPIO_OType = GPIO_OType_PP; 
	// 输入模式不进行上下拉，不然数据不准
	GPIO_InitStructure.GPIO_PuPd = GPIO_PuPd_NOPULL;
	// 写入配置
	GPIO_Init(GPIOA, &GPIO_InitStructure);

	// 打开DMA2时钟
	RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_DMA2, ENABLE);
	// 选择通道0
	DMA_InitStruct.DMA_Channel = DMA_Channel_0;
	// 定义数据的去向和大小
	DMA_InitStruct.DMA_PeripheralBaseAddr =(uint32_t)&(ADC1->DR);// 单通道采集使用CSR通用寄存器，单通道交替采集使用CDR数据寄存器
	// 定义接受缓存区
	DMA_InitStruct.DMA_Memory0BaseAddr = (uint32_t)&ADC_1_4_data;
	// 选择DMA方向
	DMA_InitStruct.DMA_DIR = DMA_DIR_PeripheralToMemory;
	// 选择传输数据量，与ADC传输通道一致
	DMA_InitStruct.DMA_BufferSize = 4;
	// 设定外设的循环模式，不使能，外设只有一个地址
	DMA_InitStruct.DMA_PeripheralInc = DMA_PeripheralInc_Disable;
	// 定义外设传输的字节大小
	DMA_InitStruct.DMA_PeripheralDataSize = DMA_PeripheralDataSize_HalfWord;
	// 设定存储的循环模式
	DMA_InitStruct.DMA_MemoryInc = DMA_MemoryInc_Enable;
	// 定义存储的字节大小
	DMA_InitStruct.DMA_MemoryDataSize = DMA_MemoryDataSize_HalfWord;
	// 要循环读取数据
	DMA_InitStruct.DMA_Mode = DMA_Mode_Circular;
	// 数据的软优先级设置
	DMA_InitStruct.DMA_Priority = DMA_Priority_High;
	// 不使用突发模式，直接改为单字节
	DMA_InitStruct.DMA_PeripheralBurst = DMA_PeripheralBurst_Single;
	// 不使用突发模式，直接改为单字节
	DMA_InitStruct.DMA_MemoryBurst = DMA_MemoryBurst_Single;
	// 不适用FIFO模式
	DMA_InitStruct.DMA_FIFOMode = DMA_FIFOMode_Disable; // FIFO模式是DMA自定义缓存区，达到标准后，一次性发送给CPU，最大为4*4=16字节
	// DMA结构体初始化
	DMA_Init(DMA2_Stream0,&DMA_InitStruct);
	// DMA使能
	DMA_Cmd(DMA2_Stream0,ENABLE);
	// 使能ADC时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
	
	// ADC复位
	ADC_DeInit();
	// 单ADC选择1，也可以不使能
	ADC_CommonInitStructure.ADC_DMAAccessMode=ADC_DMAAccessMode_1;
	// 单ADC单通道选择独立模式
	ADC_CommonInitStructure.ADC_Mode=ADC_Mode_Independent;
	// 四分频，168/4 = 42MHz
	ADC_CommonInitStructure.ADC_Prescaler=ADC_Prescaler_Div4;
	// 两个ADC采样间隔时间选择20个ADC频率
	ADC_CommonInitStructure.ADC_TwoSamplingDelay=ADC_TwoSamplingDelay_20Cycles;
	// 添加配置
	ADC_CommonInit(&ADC_CommonInitStructure);
	
	// 12位精度
	ADC_InitStructure.ADC_Resolution = ADC_Resolution_12b;
	// 单通道不开启连续扫描模式
	ADC_InitStructure.ADC_ScanConvMode = ENABLE;
	// 使能连续读取模式
	ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;
	// 不进行外部边沿触发
	ADC_InitStructure.ADC_ExternalTrigConvEdge = ADC_ExternalTrigConvEdge_None;
	// 数据右对齐
	ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
	// 开启通道数为1
	ADC_InitStructure.ADC_NbrOfConversion = 4;
	// 写入配置
	ADC_Init(ADC1, &ADC_InitStructure);
	
	// 选择ADC、通道、优先级和采样时间
	ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_15Cycles );	
	ADC_RegularChannelConfig(ADC1, ADC_Channel_1, 2, ADC_SampleTime_15Cycles );	
	ADC_RegularChannelConfig(ADC1, ADC_Channel_2, 3, ADC_SampleTime_15Cycles );	
	ADC_RegularChannelConfig(ADC1, ADC_Channel_3, 4, ADC_SampleTime_15Cycles );	
	// 使能DMA请求
  ADC_DMARequestAfterLastTransferCmd(ADC1, ENABLE);
	// 使能DMA
	ADC_DMACmd(ADC1, ENABLE);
	// 使能ADC
	ADC_Cmd(ADC1, ENABLE);
	// 进行使能标志位检测
	ADC_SoftwareStartConv(ADC1);
//	while(!ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC ));
}
```

### 多ADC多通道

```
// 多通道接收缓存区
volatile uint16_t ADC_2_4_data[4];

void ADC_2_Channel_4_Config(void)
{
	// 定义结构体
	ADC_InitTypeDef ADC_InitStructure;
	ADC_CommonInitTypeDef ADC_CommonInitStructure;
	DMA_InitTypeDef DMA_InitStruct;
	GPIO_InitTypeDef GPIO_InitStructure;
	
	// 打开时钟
	RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOA, ENABLE);
	// 配置引脚
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_2 | GPIO_Pin_3;
	// 要配置成模拟输入，而不是简单的输入模式
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AN;
	// GPIO_Speed 参数用于控制引脚在输出时的驱动能力和切换速度（引脚状态的上升和下降时间），也不进行配置
	// GPIO_InitStructure.GPIO_Speed = GPIO_Speed_100MHz;
	// 只适用于输出模式，所以不配置
	// GPIO_InitStructure.GPIO_OType = GPIO_OType_PP; 
	// 输入模式不进行上下拉，不然数据不准
	GPIO_InitStructure.GPIO_PuPd = GPIO_PuPd_NOPULL;
	// 写入配置
	GPIO_Init(GPIOA, &GPIO_InitStructure);

	// 打开DMA2时钟
	RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_DMA2, ENABLE);
	// 选择通道0
	DMA_InitStruct.DMA_Channel = DMA_Channel_0;
	// 定义数据的去向和大小
	DMA_InitStruct.DMA_PeripheralBaseAddr = (uint32_t)&ADC->CDR;// 单通道采集使用CSR通用寄存器，单通道交替采集使用CDR数据寄存器
	// 定义接受缓存区
	DMA_InitStruct.DMA_Memory0BaseAddr = (uint32_t)&ADC_2_4_data;
	// 选择DMA方向
	DMA_InitStruct.DMA_DIR = DMA_DIR_PeripheralToMemory;
	// 选择传输数据量，与ADC传输通道一致
	DMA_InitStruct.DMA_BufferSize = 4;
	// 设定外设的循环模式，不使能，外设只有一个地址
	DMA_InitStruct.DMA_PeripheralInc = DMA_PeripheralInc_Disable;
	// 定义外设传输的字节大小
	DMA_InitStruct.DMA_PeripheralDataSize = DMA_PeripheralDataSize_Word;
	// 设定存储的循环模式
	DMA_InitStruct.DMA_MemoryInc = DMA_MemoryInc_Enable;
	// 定义存储的字节大小
	DMA_InitStruct.DMA_MemoryDataSize = DMA_MemoryDataSize_Word;
	// 要循环读取数据
	DMA_InitStruct.DMA_Mode = DMA_Mode_Circular;
	// 数据的软优先级设置
	DMA_InitStruct.DMA_Priority = DMA_Priority_High;
	// 不使用突发模式，直接改为单字节
	DMA_InitStruct.DMA_PeripheralBurst = DMA_PeripheralBurst_Single;
	// 不使用突发模式，直接改为单字节
	DMA_InitStruct.DMA_MemoryBurst = DMA_MemoryBurst_Single;
	// 不适用FIFO模式
	DMA_InitStruct.DMA_FIFOMode = DMA_FIFOMode_Disable; // FIFO模式是DMA自定义缓存区，达到标准后，一次性发送给CPU，最大为4*4=16字节
	// DMA结构体初始化
	DMA_Init(DMA2_Stream0,&DMA_InitStruct);
	// 使能stream0
	DMA_Cmd(DMA2_Stream0,ENABLE);
	
	// 使能ADC时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
	// 使能ADC时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC2, ENABLE);

	// 不是交替模式选择1就行
	ADC_CommonInitStructure.ADC_DMAAccessMode=ADC_DMAAccessMode_2;
	// 双ADC交替传输
	ADC_CommonInitStructure.ADC_Mode=ADC_DualMode_Interl;
	// 四分频，168/4 = 42MHz
	ADC_CommonInitStructure.ADC_Prescaler=ADC_Prescaler_Div4;
	// 两个ADC采样间隔时间选择20个ADC频率
	ADC_CommonInitStructure.ADC_TwoSamplingDelay=ADC_TwoSamplingDelay_20Cycles;
	// 添加配置
	ADC_CommonInit(&ADC_CommonInitStructure);
	
	// 12位精度
	ADC_InitStructure.ADC_Resolution = ADC_Resolution_12b;
	// 开启连续扫描模式
	ADC_InitStructure.ADC_ScanConvMode = ENABLE;
	// 使能连续读取模式
	ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;
	// 不进行外部边沿触发
	ADC_InitStructure.ADC_ExternalTrigConvEdge = ADC_ExternalTrigConvEdge_None;
	// 数据右对齐
	ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
	// 开启通道数为1
	ADC_InitStructure.ADC_NbrOfConversion = 2;
	// 写入配置
	ADC_Init(ADC1, &ADC_InitStructure);
	ADC_Init(ADC2, &ADC_InitStructure);
	
	
	// 选择ADC、通道、优先级和采样时间
  ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_3Cycles);  // ADC1 通道0 (PA0)
  ADC_RegularChannelConfig(ADC1, ADC_Channel_1, 2, ADC_SampleTime_3Cycles);  // ADC1 通道2 (PA2)

  ADC_RegularChannelConfig(ADC2, ADC_Channel_2, 1, ADC_SampleTime_3Cycles);  // ADC2 通道1 (PA1)
  ADC_RegularChannelConfig(ADC2, ADC_Channel_3, 2, ADC_SampleTime_3Cycles);  // ADC2 通道3 (PA3)

  // 使能 DMA
  ADC_DMACmd(ADC1, ENABLE);
	
	// 使能ADC
	ADC_Cmd(ADC1, ENABLE);
  ADC_Cmd(ADC2, ENABLE);

	// 使能多模式 DMA 传输
	ADC_MultiModeDMARequestAfterLastTransferCmd(ENABLE);
	
	// 如果是独立模式需要不同的进行使能，如果是交替模式，使能一个就行
	ADC_SoftwareStartConv(ADC1);
	// 修正采集完成标志检查，确保ADC1和ADC2都完成采集
	while(!ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC));
	while(!ADC_GetFlagStatus(ADC2, ADC_FLAG_EOC));
}
```
