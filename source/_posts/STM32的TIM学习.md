---
title: STM32的TIM学习
tag: TIM
date: 2024-09-12
categories: STM32
index_img: https://s2.loli.net/2024/08/15/McDF8GUdLIQNSpy.jpg
---

# STM32的TIM学习

## 基本知识

高级控制定时器(TIM1和TIM8)和通用定时器在基本定时器的基础上引入了外部引脚，可以输入捕获和输出比较功能。

高级控制定时器比通用定时器增加了可编程死区互补输出、 重复计数器、带刹车(断路)功能，这些功能都是针对工业电机控制方面。

高级控制定时器时基单元包含一个16位自动重载计数器ARR，一个16位的计数器CNT，可向上/下计数，一个16位可编程预分频器PSC，预分频器时钟源有多种可选， 有内部的时钟、外部时钟。还有一个8位的重复计数器RCR，这样最高可实现40位的可编程定时。

高级控制定时器有四个时钟源可选：

- 内部时钟源CK_INT，内部时钟CK_INT即来自于芯片内部，等于168M

**一个高级定时器的多个输出通道**（如 `TIM1_CH1`、`TIM1_CH2`、`TIM1_CH3`）并**不等同于多个定时器**。每个通道可以产生不同的PWM占空比（但频率相同，因为它们共享计数器）。

**多个通道之间可以实现同步输出**，因为它们依赖同一个计数器。

在输入捕获时，定时器时钟频率越高，捕获的时间精度越高。

定时器的计数周期（`Period`）应足够大，以便记录下信号的变化。如果信号周期非常长，而定时器周期设置得太短，可能会导致计数器溢出。

### TIM_OCMode

```
TIM_OCMode_Timing (0x0000)
作用：这是一个时间模式。在此模式下，定时器仅比较计数器和捕获比较寄存器（CCR）中的值，不产生任何外部信号变化。
应用场景：用于定时、计数功能，不影响输出管脚。

TIM_OCMode_Active (0x0010)
作用：当计数器值与CCR中的值匹配时，输出变为高电平。
应用场景：用于激活外设或者作为标志信号触发。

TIM_OCMode_Inactive (0x0020)
作用：当计数器值与CCR中的值匹配时，输出变为低电平。
应用场景：用于关闭外设或者表示事件结束。

TIM_OCMode_Toggle (0x0030)
作用：当计数器值与CCR中的值匹配时，输出电平翻转（从高变低或从低变高）。
应用场景：用于产生方波或周期性信号输出。

TIM_OCMode_PWM1 (0x0060)
作用：这是标准的PWM（脉宽调制）模式1。当计数器值小于CCR中的值时，输出为高电平；当计数器值大于或等于CCR中的值时，输出为低电平。
应用场景：用于产生PWM信号，用于电机控制、LED调光等场景。

TIM_OCMode_PWM2 (0x0070)
作用：这是PWM模式2，与模式1相反。当计数器值小于CCR中的值时，输出为低电平；当计数器值大于或等于CCR中的值时，输出为高电平。
应用场景：用于产生相反极性的PWM信号。

TIM_ForcedAction_Active
作用：强制输出高电平，忽略计数器的比较操作。
应用场景：用于强制将输出设为高电平，通常用于紧急停止或强制触发信号。

TIM_ForcedAction_InActive
作用：强制输出低电平，忽略计数器的比较操作。
应用场景：用于强制将输出设为低电平，例如复位某些信号或设备。
```

## 代码实现PWM互补输出

```
void TIM_1_Config(void)
{
	// 结构体初始化
	GPIO_InitTypeDef GPIO_InitStruct;
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	TIM_OCInitTypeDef TIM_OCInitStruct;
	TIM_BDTRInitTypeDef TIM_BDTRInitStruct;
	
	// 打开时钟
	RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOA, ENABLE);
	RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOB, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_TIM1, ENABLE);
	
	// 引脚的复用，必须在配置之前
	GPIO_PinAFConfig(GPIOA, GPIO_PinSource8, GPIO_AF_TIM1);
	GPIO_PinAFConfig(GPIOB, GPIO_PinSource13, GPIO_AF_TIM1);
	GPIO_PinAFConfig(GPIOB, GPIO_PinSource12, GPIO_AF_TIM1);
	
	// PWM正向输出引脚
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_8;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF;
	GPIO_InitStruct.GPIO_OType = GPIO_OType_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_100MHz;
	GPIO_InitStruct.GPIO_PuPd = GPIO_PuPd_NOPULL;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// PWM互补输出引脚
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_13;
	GPIO_Init(GPIOB, &GPIO_InitStruct);
	
	// 断路保护输出引脚
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_12;
	GPIO_Init(GPIOB, &GPIO_InitStruct);
	
	// 重装载计数器，主要用于实现定时器的计数总数
	TIM_TimeBaseInitStruct.TIM_Period = 280-1; //600KHz = 1.67us
	// 设定定时器频率为=TIMxCLK(168000000Hz)/(TIM_Prescaler+1)=100000Hz，单次计数的时间
	TIM_TimeBaseInitStruct.TIM_Prescaler = 1 - 1;
	// 定时器的计数方向
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	// 重复计数，等效于上面TIM_Period * n ,方便用于控制产生多少个pwm波
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	// 时钟分频，在计算死区时间的时候会用到
	TIM_TimeBaseInitStruct.TIM_ClockDivision = TIM_CKD_DIV1;
	// 配置结构体
	TIM_TimeBaseInit(TIM1, &TIM_TimeBaseInitStruct);
	
	// 输出比较模式选择PWM1，
	TIM_OCInitStruct.TIM_OCMode = TIM_OCMode_PWM1;
	// 主输出引脚使能
	TIM_OCInitStruct.TIM_OutputState = TIM_OutputState_Enable;
	// 互补输出使能
	TIM_OCInitStruct.TIM_OutputNState = TIM_OutputNState_Enable;
	// 脉冲调制宽度，也就是占空比 = 重装载计数器/脉冲调制宽度
	TIM_OCInitStruct.TIM_Pulse = 140;
	// 主输出的高电平有效，也就是高电平作为输出信号，反之则为低电平
	TIM_OCInitStruct.TIM_OCPolarity = TIM_OCPolarity_High;
	// 互补输出的高电平有效，也就是高电平作为输出信号，反之则为低电平
	TIM_OCInitStruct.TIM_OCNPolarity = TIM_OCNPolarity_High;
	// 当停止计数器时，比较器稳定为高电平，保证输出到引脚的信号是一致稳定的
	TIM_OCInitStruct.TIM_OCIdleState = TIM_OCIdleState_Set;
	// 当停止计数器时，比较器稳定为高电平，保证输出到引脚的信号是一致稳定的
	TIM_OCInitStruct.TIM_OCNIdleState = TIM_OCNIdleState_Reset;
	// 配置通道1的输出比较
	TIM_OC1Init(TIM1, &TIM_OCInitStruct);
	// 配置预装载，在改变脉冲的时候能够让数据更加平缓和准确
	TIM_OC1PreloadConfig(TIM1, TIM_OCPreload_Enable);
	
	// 自动输出使能，在停止时或者特定状态时，保证引脚的输出是想要的状态，确保状态一致
	TIM_BDTRInitStruct.TIM_OSSRState = TIM_OSSRState_Enable;
	// 自动输出使能，在空闲状态时，保证引脚的输出是想要的状态，确保状态一致
	TIM_BDTRInitStruct.TIM_OSSIState = TIM_OSSIState_Enable;
	// 锁定定时器的部分配置，防止被篡改
	TIM_BDTRInitStruct.TIM_LOCKLevel = TIM_LOCKLevel_1;
	// 死区时间的配置
	TIM_BDTRInitStruct.TIM_DeadTime = 11;
	// 断路保护功能，将输出引脚复位至TIM_OCIdleState或TIM_OCNIdleState设定的状态
	TIM_BDTRInitStruct.TIM_Break = TIM_Break_Enable;
	// 触发断路保护时引脚的电平信号,引脚的初始化为高电平，如果设置为高电平触发的话，无法生成PWM
	TIM_BDTRInitStruct.TIM_BreakPolarity = TIM_BreakPolarity_Low;
	// 发生短路时，开启自动断路保护
	TIM_BDTRInitStruct.TIM_AutomaticOutput = TIM_AutomaticOutput_Enable;
	// 配置死区定时器
	TIM_BDTRConfig(TIM1, &TIM_BDTRInitStruct);
	
	// 使能定时器
  	TIM_Cmd(TIM1, ENABLE);

    // 用于开启高级定时器的PWM输出功能，可以初始化时进行配置，也可以在后续使用时配置
    TIM_CtrlPWMOutputs(TIM1, ENABLE);
}
```

