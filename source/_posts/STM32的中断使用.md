---
title: STM32的中断使用
tag: 中断函数
date: 2024-07-23
categories: STM32
---

# STM32的中断使用

### 内部中断NVIC

配置过程中，首先初始化 NVIC_InitTypeDef 结构体，配置内部中断线、配置中断优先级分组，设置抢占优先级和子优先级，使能中断请求，最后写入初始化结构体。

在启动文件 startup_stm32f10x_hd.s 中我们预先为每个中断都写了一个中断服务函数，只是这些中断函数都是为空 实际的中断服务函数都需要我们重新编写，为了方便管理我们把中断服务函数统一写在 stm32f10x_it.c 这个库文件中。

NVIC_IROChannel：用来设置中断源，不同的中断中断源不一样

系统就在中断向量表中找不到中断服务函数的入口，直接跳转到启动文件里面预先写好的空函数，并且在里面无限循环，实现不了中断。

NVIC_PriorityGroupConfig 是整个程序中只需要设置一次，多次设置无效，他只是一种配置方式，具体的分类有主优先级和子优先级有关，他只是提供不同的组合方式，需要根据项目需求进行设计

从代码布局逻辑来说,NVIC_PriorityGroupConfig 适合放在 main() 函数中。

```
	//定义结构体
	NVIC_InitTypeDef  NVIC_InitStruct;
	
	//NVIC初始化
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup);//配置组优先级（0，1，2，3，4，5）
	NVIC_InitStruct.NVIC_IRQChannel = KEY_INT_EXTI_IRQ;//配置内部中断线
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = PreemptionPriority;//配置抢占优先级
	NVIC_InitStruct.NVIC_IRQChannelSubPriority = SubPriority;//配置子优先级
	NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;//使能中断通道
	NVIC_Init(&NVIC_InitStruct);//将结构体进行定义
```

### 外部中断EXTI

EXTI（External interrupt/event controller）—外部中断/事件控制器，管理了控制器的 20 个中断/事件线。每个中断/事件线都对应有一个边沿检测器，可以实现输入信号的上升沿检测和下降沿的检测。 EXTI 可以实现对每个中断/事件线进行单独配置，可以单独配置为中断或者事件，以及触发事件的属性。

所以EXTI的配置，首先需要中断结构体初始化、打开AFIO中断线管理器的时钟、配置中断/事件线（20条）、选择中断屏蔽器输入位，并使能、选择中断/事件触发类型、选择上升沿或下降沿检测，最后写入初始化结构体

```
//定义结构体
	GPIO_InitTypeDef  GPIO_InitStruct;
	EXTI_InitTypeDef  EXTI_InitStruct;
	
	//配置中断优先级
	EXTI_NVIC_Config();
	
	//按键引脚初始化
	RCC_APB2PeriphClockCmd(KEY_INT_GPIO_CLK, ENABLE);//打开按键时钟
	GPIO_InitStruct.GPIO_Pin = KEY_INT_GPIO_PIN;//按键引脚定义
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IN_FLOATING;//选择输入输出模式（浮空输入）
	GPIO_Init(KEY_INT_GPIO_PORT,&GPIO_InitStruct);//将结构体进行定义
	
	//中断配置初始化
	RCC_APB2PeriphClockCmd(KEY_INT_EXTI_CLK, ENABLE);//打开AFIO时钟
	GPIO_EXTILineConfig(KEY_INT_EXTI_PORTSOURCE, KEY_INT_EXTI_PINSOURCE);//产生中断的事件线（Px0属于Line0，Px1属于Line1...）
	
	EXTI_InitStruct.EXTI_Line = KEY_INT_EXTI_LINE;//中断屏蔽器输入位的选择，以供EXTI_LineCmd精准使能
	EXTI_InitStruct.EXTI_LineCmd = ENABLE;//中断屏蔽器使能（控制中断接受IMR/EMR）
	EXTI_InitStruct.EXTI_Mode = EXTI_Mode_Interrupt;//选择触发类型（中断触发和事件触发）
	EXTI_InitStruct.EXTI_Trigger = EXTI_Trigger_Rising;//选择模式（上升沿和下降沿）
	EXTI_Init(&EXTI_InitStruct);//将结构体进行定义
```

前面 4 个线路有单独的中断函数，后面 5 至 9 和 10 至 15 线路使用 复用的思想思考，区分出什么是可以唯一标识的，什么是复用的，EXTI_Lines 在寄存器中都是一一对应状态标位, 中断函数复用，**因此在 EXTI9_5_IRQHandler 和 EXTI15_10_IRQHandler 的中断函数里面使用多次EXTI_GetITStatus 函数来判断出线路。**  

```
	if(EXTI_GetITStatus(EXTI_Line10) != RESET)
	{
		LED_TOGGLE;
	}
	if(EXTI_GetITStatus(EXTI_Line14) != RESET)
	{
		LED_TOGGLE;
	}
```
