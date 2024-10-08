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

共同配置主要配置所有ADC的采集模式、采集频率、传输模式和采集间隔时间，共同配置是在使用两个ADC硬件以上的才会选择进行配置，如果是单ADC的话，是不进行配置的

```
	/* adc共同配置 ADC_CommonInitTypeDef */
	
	// 初始化结构体
	ADC_CommonInitTypeDef ADC_CommonInitStructure;
	// 双ADC模式，常规的交替转换
	ADC_CommonInitStructure.ADC_Mode = ADC_DualMode_Interl;
	// AHB2的时钟是168MHz，四分频是42MHz，ADC最大为36MHz
	ADC_CommonInitStructure.ADC_Prescaler = ADC_Prescaler_Div4;
	// 选择DMA的传输方式，ADC_DMAAccessMode_1、2、3三种模式，1适用于单通道，2和3适用于多通道，3适用于更高速、更大的传输，比如图片、视频等
	ADC_CommonInitStructure.ADC_DMAAccessMode = ADC_DMAAccessMode_2;
	// 两个ADC采样的间隔时间
	ADC_CommonInitStructure.ADC_TwoSamplingDelay = ADC_TwoSamplingDelay_20Cycles;
```

个体配置主要配置ADC的采集精度、是否扫描、是否连续读取、是否外部触发、外部触发边沿选择、外部触发模式、采集通道数、数据对齐方式，不管是单ADC，还是多ADC，都需要配置每个ADC的个体配置

```
	/* adc额外配置 ADC_CommonInitTypeDef */
	
	// 定义结构体
	ADC_InitTypeDef ADC_InitStructure;
	// ADC读取为是12位，处理的时间也是12个ADC时钟
	ADC_InitStructure.ADC_Resolution = ADC_Resolution_12b;
	// 使能扫描模式，如果是多通道的，就需要扫描模式
	ADC_InitStructure.ADC_ScanConvMode = ENABLE;
	// 连续读取模式，可以连续不停而不需要软硬件触发就可以采样ADC
	ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;
	// 外部边沿触发
	ADC_InitStructure.ADC_ExternalTrigConvEdge = ADC_ExternalTrigConvEdge_None;
	// 外部触发的模式，定时器或者中断
	ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_T1_CC1;
	// 采样的通道数量
	ADC_InitStructure.ADC_NbrOfConversion = 1;
	// 数据右对齐，就是先写入右边数据，多通道的话就是右八位为第一个检测到的数据
	ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
	// 写入配置
	ADC_Init(ADC1, &ADC_InitStructure);
```

最后还需要选择所需通道和采样时长，并使能所有ADC

```
	// 配置通道和采样的时长
	ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_3Cycles);  
	
	// 使能ADC
	ADC_Cmd(ADC1, ENABLE);
	
	// 使能ADC-DMA
	ADC_DMACmd(ADC1, ENABLE);
```

## ADC配置顺序

ADC的使用方式有很多种，如果比较简单的话就直接进行单ADC和单通道直接使用，也可以多通道单ADC使用，或者多通道多ADC使用，这几种方法可以根据不同的配置进行实现

### 单ADC单通道

1.初始化结构体和启用ADC时钟

```
	// 初始化结构体
	ADC_CommonInitTypeDef ADC_CommonInitStructure;
	
	// 使能ADC时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);


```

2.初始化ADC个体配置，单ADC不需要分频，因为系统会自动分频到36MHz

3.选择ADC的通道和采样时间

4.使能ADC

5.启用ADC

6.如果单ADC单通道传输的话想要启用DMA，就需要使能ADC和DMA通道

7.如果软件触发ADC的话，那就需要将外部触发的不配置为使能，如果外部触发配置使能，那么软件触发将不生效

### 单ADC多通道





## DMA传输流程图

![image.png](https://s2.loli.net/2024/09/15/lQoOgFdmNfSWJ8P.png)

![image.png](https://s2.loli.net/2024/09/15/Bw6jIWZN8kOYtiD.png)



ADC_Resolution

STM32F407的ADC读取精度是12位（即 12-bit）。ADC 分辨率决定了转换结果的精度，12 位分辨率意味着 ADC 可以将模拟信号转换为 4096 个不同的数字值（2^12）。当然也就是说处理的时间是12个ADC时钟。

ADC_ScanConvMode

扫描模式的作用是可以针对多个通道进行扫描，如果不开启的话只能在同一个通道上面进行读取

ADC_ContinuousConvMode

连续转换模式。连续转换模式使得 ADC 在完成一个转换后自动开始下一个转换，而无需额外的触发信号。这样可以实现不断更新的 ADC 值。

ADC_ExternalTrigConvEdge

外部触发转换的边缘，设置外部触发信号的边缘，以控制 ADC 转换的启动时机。主要就是上升沿、下降沿、两个都用或者不启用

ADC_ExternalTrigConv

选择外部触发信号，外部触发的信号主要有定时器、中断触发，并且这些触发的话每个设备都是固定的，在头文件中宏定义会全部定义完全。内部触发意味着 ADC 转换的启动由微控制器内部的触发源控制，例如内部定时器、软件触发或其他内部事件，内部触发的话是比较快，比较简单，而且可以根据软件触发的方式进行随意检测，但是硬件检测的方式对于周期的控制比较精确，但是比较麻烦。

ADC_NbrOfConversion

设置要进行的转换通道数，如果启用扫描模式，则可以启用更多的通道

ADC_DataAlign

设置 ADC 转换结果的数据对齐方式。`ADC_DataAlign_Right` 表示转换结果右对齐，即数据的低位在数据寄存器的低位部分，高位在高位部分。另一种对齐方式是左对齐

ADC_RegularChannelConfig

配置 ADC 的常规（非扫描模式）通道。这个是配置ADC通道和设置采样事件













