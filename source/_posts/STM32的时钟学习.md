---
title: STM32的时钟学习
tag: Systick
date: 2024-07-25
categories: STM32
index_img: https://s2.loli.net/2024/07/31/kWFf3yjEqbCtX8M.jpg
---

# STM32的时钟学习

## 时钟SysTick

时钟一共有五个部分，重装载寄存器、递减计数器、时钟源、时钟中断控制器、校准数值寄存器（用不到）

![SysTick.png](https://s2.loli.net/2024/07/27/mbKvAFMpZEVXnd4.png)

#### 重装载寄存器 LOAD

作用就是当递减计数器递减到零时，重新装载计数器，再次进行递减计数

#### 递减计数器 VAL

读取时会返回该递减值，如果在该位写入1，则会清理，同时也会清除时钟中断控制器里面的COUNTFLAG

#### 时钟源

提供递减计数器工作的动力源，有外部晶振提供

#### 时钟中断控制器

```
16-COUNTFLAG：如果上次读取该寄存器后，再递减到1，则该位置为1

2-CLKSOURCE：时钟源选择，0=AHB/8=72/8=9M，1=AHB=72M

1-TICKINT：递减计数器到零时发出异常请求

0-ENABLE：控制器的使能位
```

#### 代码Demo

```
#include "bsp_systick.h"

#if 0

static __INLINE uint32_t SysTick_Config(uint32_t ticks)
{ 
	//判断ticks传入的值是否大于 2^24，如果大于则返回错误值
  if (ticks > SysTick_LOAD_RELOAD_Msk)  return (1);            /* Reload value impossible */
  
	//初始化寄存器reload的值                                  
  SysTick->LOAD  = (ticks & SysTick_LOAD_RELOAD_Msk) - 1;      /* set reload register */
	
	//配置时钟中断优先级，1左移四个位，再减1，配置的是15，最低等级
  NVIC_SetPriority (SysTick_IRQn, (1<<__NVIC_PRIO_BITS) - 1);  /* set Priority for Cortex-M0 System Interrupts */
  
	//初始化count的值为0
	SysTick->VAL   = 0;                                          /* Load the SysTick Counter Value */
	
	//时钟配置成72M，使能中断，使能systick
  SysTick->CTRL  = SysTick_CTRL_CLKSOURCE_Msk | 
                   SysTick_CTRL_TICKINT_Msk   | 
                   SysTick_CTRL_ENABLE_Msk;                    /* Enable SysTick IRQ and SysTick Timer */
  return (0);                                                  /* Function successful */
}

#endif

void SysTick_Delay_us(uint32_t us)
{
	uint32_t i;
	
	//T=tick/CLKAHB  CLKAHB是配置为 72M，所以中断的时间就是 72/72M=1us 微秒
	SysTick_Config(72);
	
	for(i=0;i<us;i++)//进行输入值的循环，循环结束则退出
	{
		while (!((SysTick -> CTRL) & (1<<16)));//使用标志位判断是否完成一次循环，如果该位变成1，代表完成循环
	}
	SysTick->CTRL &= ~SysTick_CTRL_ENABLE_Msk;//将使能位关闭，停止计时，退出循环
}

void SysTick_Delay_ms(uint32_t ms)
{
	uint32_t j;
	
	//T=tick/CLKAHB  CLKAHB是配置为 72M，所以中断的时间就是 72/72M=1us 微秒
	SysTick_Config(72000);

	
	for(j=0;j<ms;j++)//进行输入值的循环，循环结束则退出
	{
		while (!((SysTick -> CTRL) & (1<<16)));//使用标志位判断是否完成一次循环，如果该位变成1，代表完成循环
	}
	SysTick->CTRL &= ~SysTick_CTRL_ENABLE_Msk;//将使能位关闭，停止计时，退出循环
}

```
