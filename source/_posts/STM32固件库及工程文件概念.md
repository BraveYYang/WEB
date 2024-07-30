---
title: STM32固件库及工程文件概念
tag: 固件库
date: 2024-06-30
categories: STM32
---

# STM32固件库及工程文件概念

## STM32固件库整体结构

 	1-汇编编写的启动文件
 	startup_stm32f10x_hd.s:设置堆栈指针、设置PC指针、初始化中断向量表、配置系统时钟、调用c库函数_main最终去到c的世界
 	
 	2-时钟配置文件
 	system_stm32f10x.c:把外部时钟HSE=8M，经过PLL倍频为72M。
 	
 	3-外设相关的
 	stm32f10x.h:实现了内核之外的外设的寄存器映射xxx:GPIO、USRAT、I2c、SPI、FSMC
 	stm32f10x_xx.c:外设的驱动函数库文件
 	stm32f10x_xx.h:存放外设的初始化结构体，外设初始化结构体成员的参数列表，外设固件库函数的声明
 	
 	4-内核相关的
 	CMSIs - Cortex 微控制器软件接口标准
 	core_cm3.h:实现了内核里面外设的寄存器映射
 	core_cm3.c
 	NVIC(嵌套向量中断控制器)、sysTick(系统滴答定时器)
 	
 	ST公司裁剪后的内核文件
 	misc.h
 	misc.c
 	
 	5-头文件的配置文件
 	stm32f10x_conf.h:头文件的头文件
 	/ / stm32f10x_usart.h
 	/ / stm32f10x_i2c.h
 	/ / stm32f10x_spi.h
 	/ / stm32f10x_adc.h
 	/ / stm32f10x_fsmc.h
 	
 	6-专门存放中断服务函数的c文件
 	stm32f10x_it.c
 	stm32f10x_it.h
 	
 	中断服务函数你可以随意放在其他的地方，并不是一定要放在stm32f10x_it.c
 	
 	#include "stm32f10x.h"//相当于51单片机中的#include <reg51.h>
 	int main (void)
 	{
 		//来到这里的时候，系统的时钟已经被配置成72M
 	}

### 文件设置

#### 1.设置创建项目文件夹，并在文件夹内设置四个文件

#### ①Libraries：

```
内核相关文件（core_cm3.c、core_cm3.h）

启动文件（startup_stm32f10x_hd）

时钟配置文件（system_stm32f10x.c、system_stm32f10x.h）

外设文件（stm32f10x.h和固件库h.c文件）
```

#### ②Project

```
KEIL工程文件

输出HEX文件，用于准备烧录
```

#### ③User

```
用户程序放置

main.c文件
```

#### ④Doc

```
README.md文档
```

#### 2.打开工程，创建五个文件夹

```
STARTUP：导入启动文件

CMSIS：导入内核文件和时钟配置文件

FWLIB：导入固件库

USER：导入main.c文件和用户文件

DOC：导入README文件
```

#### 3.所有设计到头文件的文件夹配置到系统环境中

#### 4.开始编译全文件

## 其他

### 资源

keil软件芯片包：[Arm Keil | Keil STM32F1xx_DFP](https://www.keil.arm.com/packs/stm32f1xx_dfp-keil/boards/)

### 报错

Reason: Pack [schema](https://so.csdn.net/so/search?q=schema&spm=1001.2101.3001.7020) version ‘1.4.0’ is not supported. Maximum supported [version](https://so.csdn.net/so/search?q=version&spm=1001.2101.3001.7020) is’1.3’. Please update to a newer version of MDK-ARM. 使用keil5 MDK安装芯片包时出错（软件版本过低）

![image-20240531210625344.png](https://s2.loli.net/2024/07/25/HgserB9cVJfObdM.png)

#### 解决方案

根据弹窗的内容我们可知我们需要对MDK进行升级，在升级时我们直接覆盖安装。

官方下载渠道：https://www.keil.com/download/product/
填写相关信息便可进行下载。

破解keil出现出现you are not logged in as an administrator....的问题，请退出软件，右击软件图标，以管理员身份运行，重新进行安装操作！然后就安装成功了！
