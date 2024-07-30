---
title: STM32程序逻辑及GPIO使用
tag: GPIO
date: 2024-07-10
categories: STM32
---

# STM32程序逻辑及GPIO使用

## 程序逻辑

#### 1.创建在USER文件夹中创建想要的文件夹，例如bsp_led，主要用来存放C文件和h文件

#### bsp_led.c

主要用来构建所需程序的函数体（比如配置、驱动等等）

#include "bsp_led.h"用来调用各种东西

#### bsp_led.h

主要用来给bsp_led.c文件提供库函数支持

以及bsp_led.c所需的宏定义，都可以放在该文件夹中

将bsp_led.c中的函数放入

采用#ifndef和#endif的方式，实现头文件的判断编译，防止多次编译报错

```
#ifndef __BSP_LED_H
#define __BSP_LED_H

#include "stm32f10x.h"

#define GPIO_G_PIN         GPIO_Pin_0
#define GPIO_G_PORT        GPIOB
#define GPIO_G_CLK         RCC_APB2Periph_GPIOB

void delay(uint32_t count);
void LED_GPIO_Config(void);

#endif /* __BSP_LED_H */
```

#### 2.在主函数中调用#include "bsp_led.h"即可使用bsp_led.c中编写的各类函数

## GPIO设置

```
//定义关键字
GPIO_InitTypeDef  GPIO_InitStruct;

//选择引脚，将硬件部分定义成宏，方便移植
#define GPIO_G_PIN         GPIO_Pin_0
GPIO_InitStruct.GPIO_Pin = GPIO_G_PIN;

//选择输出方式
GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_PP;

//选择速度
GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;

//配置引脚
GPIO_Init(GPIO_G_PORT,&GPIO_InitStruct);

//启动时钟
RCC_APB2PeriphClockCmd(GPIO_G_CLK, ENABLE);
```
