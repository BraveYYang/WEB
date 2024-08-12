---
title: STM32的DMA收发配置
tag: DMA数据传输
date: 2024-08-08
categories: STM32
index_img: https://s2.loli.net/2024/08/02/Jw2m6LlYHhnTg5q.jpg
---

# STM32的DMA收发配置

DMA(Direct Memory Access)—直接存储器存取，是单片机的一个外设，它的主要功能是用来搬数据，但是不需要占用 CPU，即在传输数据的时候， CPU 可以干其他的事情，好像是多线程一样

DMA主要组成有控制器请求、通道、仲裁器

### 控制器

请求的话主要由想要使用方发起，如果外设要想通过 DMA 来传输数据，必须先给 DMA 控制器发送 DMA 请求， DMA 收到请求信号之后，控制器会给外设一个应答信号，当外设应答后且 DMA 控制器收到应答信号之后，就会启动 DMA 的传输，直到传输完毕。

### 通道

DMA 控制器包含了 DMA1 和 DMA2，其中 DMA1 有 7 个通道， DMA2 有 5 个通道，通道的地址也是DMA存储器的地址

每个通道对应不同的外设的 DMA 请求。**虽然每个通道可以接收多个外设的请求，但是同一时间只能接收一个，不能同时接收多个。**

### 仲裁器

当发生多个 DMA 通道请求时，就意味着有先后响应处理的顺序问题，这个就由仲裁器也管理。

仲裁器管理 DMA 通道请求分为两个阶段。

第一阶段属于软件阶段，可以在 **DMA_CCRx 寄存器中设置，有 4 个等级：非常高、高、中和低四个优先级。**

第二阶段属于硬件阶段，如果两个或以上的 DMA 通道请求设置的优先级一样，则他们优先级取决于通道编号，编号越低优先权越高，比如通道 0 高于通道1。

### 数据传输

DMA 传输数据的方向有三个：从外设到存储器，从存储器到外设，从存储器到存储器

DMA_CCR 位 4 DIR 配置： 0 表示从外设到存储器， 1 表示从存储器到外设。储存器到储存器有单独设置。

当我们使用从外设到存储器传输时，以 ADC 采集为例。 **DMA 外设寄存器的地址对应的就是 ADC数据寄存器的地址， DMA 存储器的地址就是我们自定义的变量**（用来接收存储 AD 采集的数据）的地址。

当我们使用从存储器到外设传输时，以串口向电脑端发送数据为例。 **DMA 外设寄存器的地址对应的就是串口数据寄存器的地址， DMA 存储器的地址就是我们自定义的变量**（相当于一个缓冲区，用来存储通过串口发送到电脑的数据）的地址。  

当我们使用从存储器到存储器传输时，以内部 FLASH 向内部 SRAM 复制数据为例。 DMA 外设寄存器的地址对应的就是内部 FLASH（我们这里把内部 FALSH 当作一个外设来看）的地址， DMA存储器的地址就是我们自定义的变量  

一个 32 位的寄存器，一次最多只能传输 65535 个数据。  

要想数据传输正确，源和目标地址存储的数据宽度还必须一致，串口数据寄存器是 8 位的，所以我们定义的要发送的数据也必须是 8 位。  

### 代码解释

DMA的配置上也比较清晰，跟之前的类似，首先要初始化结构体，并且打开DMA1的时钟，所有的外设都要有这个操作

```
	DMA_InitTypeDef DMA_InitStruct;//初始化结构体
	
	RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);//打开DMA1的时钟
```

第一部分是定义数据的去向和大小，分别是要去的地址(外设)，从哪里来的地址(DMA1地址)，传输方向和传输数目

```
	DMA_InitStruct.DMA_PeripheralBaseAddr = (uint32_t)(USART1_BASE+0x04);//外设地址，定义数据去向地址
	
	DMA_InitStruct.DMA_MemoryBaseAddr = (uint32_t)DMA1_Channel4;//存储器地址
	
	DMA_InitStruct.DMA_DIR = DMA_DIR_PeripheralDST;//传输方向选择，可选外设到存储器、存储器到外设。
	
	DMA_InitStruct.DMA_BufferSize = 256;//设定待传输数据数目
```

第二部分是设定数据的传输模式和宽度

首先是外设数据的模式要设为不自动递增，因为寄存器就只有一个，无法多次递增，在设置外设数据的宽度，外设数据的宽度要和DMA1存储器的宽度一致，如果不一致的话，会导致数据丢失

```
	//一般外设都是只有一个数据寄存器，所以一般不会使能该位
	DMA_InitStruct.DMA_PeripheralInc = DMA_PeripheralInc_Disable;//使能外设地址自动递增功能
	
	DMA_InitStruct.DMA_PeripheralDataSize = DMA_PeripheralDataSize_Byte;//外设数据宽度
```

在设置存储器的数据模式和宽度，数据模式选择递增的方式，因为数据寄存器要接收多个数据，接受完一个要继续接收，可以理解为外设接收完数据直接清零，所以不需要递增，存储器无法清零，所以要递增，宽度一致

```
	//我们自定义的存储区一般都是存放多个数据的，所以要使能存储器地址自动递增功能。
	DMA_InitStruct.DMA_MemoryInc = DMA_MemoryInc_Enable;//使能存储器地址自动递增功能
	
	DMA_InitStruct.DMA_MemoryDataSize = DMA_PeripheralDataSize_Byte;//存储器数据宽度
```

在设置数据的传输模式，是循环还是单次传输

```
	DMA_InitStruct.DMA_Mode = DMA_Mode_Normal;//DMA 传输模式选择，可选一次传输或者循环传输
```

第三部分是数据的软优先级设置，以及数据初始化、数据使能和标志位清零

```
	DMA_InitStruct.DMA_Priority = DMA_Priority_High;//软件设置通道的优先级，有 4 个可选优先级分别为非常高、高、中和低
	
	DMA_InitStruct.DMA_M2M = DMA_M2M_Disable;//存储器到存储器模式，使用存储器到存储器时用到
	
	DMA_Init(DMA1_Channel4, &DMA_InitStruct);//对配置进行初始化
	
	//用于清除 DMA 标志位，代码用到传输完成标志位，使用之前先清除传输完成标志位以免产生不必要干扰
	DMA_ClearFlag(DMA1_FLAG_TC4);
	
	DMA_Cmd(DMA1_Channel4, ENABLE);//启动或者停止 DMA 数据传输
```

### 配置总代码

```
void UtM_DMA_Config(void)
{
	DMA_InitTypeDef DMA_InitStruct;//初始化结构体
	
	RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);//打开DMA1的时钟
	
	DMA_InitStruct.DMA_PeripheralBaseAddr = (uint32_t)(USART1_BASE+0x04);//外设地址，定义数据去向地址
	
	DMA_InitStruct.DMA_MemoryBaseAddr = (uint32_t)DMA1_Channel4;//存储器地址
	
	DMA_InitStruct.DMA_DIR = DMA_DIR_PeripheralDST;//传输方向选择，可选外设到存储器、存储器到外设。
	
	DMA_InitStruct.DMA_BufferSize = 256;//设定待传输数据数目
	
	//一般外设都是只有一个数据寄存器，所以一般不会使能该位
	DMA_InitStruct.DMA_PeripheralInc = DMA_PeripheralInc_Disable;//使能外设地址自动递增功能
	
	DMA_InitStruct.DMA_PeripheralDataSize = DMA_PeripheralDataSize_Byte;//外设数据宽度
	
	//我们自定义的存储区一般都是存放多个数据的，所以要使能存储器地址自动递增功能。
	DMA_InitStruct.DMA_MemoryInc = DMA_MemoryInc_Enable;//使能存储器地址自动递增功能
	
	DMA_InitStruct.DMA_MemoryDataSize = DMA_PeripheralDataSize_Byte;//存储器数据宽度
	
	DMA_InitStruct.DMA_Mode = DMA_Mode_Normal;//DMA 传输模式选择，可选一次传输或者循环传输
	
	DMA_InitStruct.DMA_Priority = DMA_Priority_High;//软件设置通道的优先级，有 4 个可选优先级分别为非常高、高、中和低
	
	DMA_InitStruct.DMA_M2M = DMA_M2M_Disable;//存储器到存储器模式，使用存储器到存储器时用到
	
	DMA_Init(DMA1_Channel4, &DMA_InitStruct);//对配置进行初始化
	
	//用于清除 DMA 标志位，代码用到传输完成标志位，使用之前先清除传输完成标志位以免产生不必要干扰
	DMA_ClearFlag(DMA1_FLAG_TC4);
	
	DMA_Cmd(DMA1_Channel4, ENABLE);//启动或者停止 DMA 数据传输
}
```

