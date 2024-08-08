---
title: STM32的USART接收测试
tag: USART
date: 2024-07-31
categories: STM32
index_img: https://s2.loli.net/2024/07/31/CsbJKkLaQt9OgVu.jpg
---

# STM32的USART接收测试

## printf()和getchar()的重定义

这两个函数在STM32里面并没有进行强定义，因为他的意思是输出和输入，但是输出输入的位置并没有确定，所以无法定义，因此我们要使用的时候，需要对这两个函数进行指定定义，比如定义到USART1，通过电脑或者其他上位机进行读取或者发送数据

```
//重定向c库函数printf到串口，重定向后可使用printf函数
int fputc(int ch, FILE *f)
{
		/* 发送一个字节数据到串口 */
		USART_SendData(USART1, (uint8_t) ch);
		
		/* 等待发送完毕 */
		while (USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET);		
	
		return (ch);
}

//重定向c库函数scanf到串口，重写向后可使用scanf、getchar等函数
int fgetc(FILE *f)
{
		/* 等待串口输入数据 */
		while (USART_GetFlagStatus(USART1, USART_FLAG_RXNE) == RESET);

		return (int)USART_ReceiveData(USART1);
}
```

## 中断直接接收数据

USART接收一共有两种方法，第一种是利用中断直接接收数据，这种方式的优先级可以自定义，如果有特殊需要时可以采用这种方式，还有一种方法是正常接收，按照程序流程进行接收

接收数据需要随时进行触发，在函数运行时也有可能收到数据，所以需要对USART进行中断配置，当然如果不需要使用中断接收数据，可以不进行中断配置，直接读取USART_RX串口的数据即可，但是需要把所有中断注释

中断接收数据需要对中断函数进行配置

#### 中断优先级配置

这个跟中断配置代码的一样，可以参照中断配置，这个主要为内部中断配置，而不是外部中断配置，因为外部中断是真的中断，USART有属于自己的额外的中断线，所以配置USART约等于配置了外部中断，所以只需要内部中断即可

```
	//定义结构体
	NVIC_InitTypeDef  NVIC_InitStruct;
	
	//NVIC初始化
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//配置组优先级（0，1，2，3，4，5）
	NVIC_InitStruct.NVIC_IRQChannel = USART1_IRQn;//配置内部中断线
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = 1;//配置抢占优先级
	NVIC_InitStruct.NVIC_IRQChannelSubPriority = 1;//配置子优先级
	NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;//使能中断通道
	NVIC_Init(&NVIC_InitStruct);//将结构体进行定义
```

#### 中断串口和串口使能

```
	// 使能串口接收中断
	USART_ITConfig(USART1, USART_IT_RXNE, ENABLE);	
	// 使能串口空闲中断
	USART_ITConfig(USART1, USART_IT_IDLE, ENABLE);
```

#### USART中断服务函数

这个是需要在stm32f10x_it.c文件里面进行设置，跟按键中断一致，所有的中断函数的触发都是需要在这个文件里面进行定义，中断函数的函数名需要查询中断表格，不能自定义

USART_IT_IDLE标志位会在RX总线从busy状态转换到free状态时被触发，也就是只有接收到数据后连续一个数据帧时间内不再有数据发送过来的时候触发，常用于判断单次数据接收完成。

USART_IT_IDLE标志位只能在中断中进行使用

中断标志位由硬件触发（代表USART_RX端口接收数据检测），软件清零（需要使用USART_IT_IDLE函数判断是否完成传输，是否有空闲线路，再使用USART_ReceiveData(USART1)进行软件标志位清零），清零方法时读取一遍USART_SR寄存器然后读取一遍USART_DR寄存器，也就是不能简单的用USART_ClearITPendingBit()来清除就完事。

如果想实现长数据的接收，可以定义直接在中断里面定义一个字节长度的数组，然后不断的往里面塞数据，用中断空闲标志位来判断什么时候停止接收退出循环（判断最后一位是否为“\0”，如果是的话，则中断停止回到原有程序中）

```
#include "bsp_usart.h"//必须在stm32f10x_it.c文件里面包含头文件，否则会报错

// USART1中断服务函数
void USART1_IRQHandler (void)
{
	//定义接收缓存区
	uint8_t ucTemp;
	//判断中断接收标志位，进入该函数，说明有数据进入
	if(USART_GetITStatus(USART1, USART_IT_RXNE) != RESET)
	{
		//读取数据并储存
		ucTemp = USART_ReceiveData(USART1);
		//将数据发送到USART1进行查看
    	USART_SendData(USART1,ucTemp);
    	
    	//做各种函数接收数据处理
    	/*.................*/
    	
	} 
	//判断中断是否接收完毕
	if(USART_GetITStatus(USART1, USART_IT_IDLE) != RESET)
	{	
    	USART_ReceiveData(USART1);//软件序列清除标志位流程

		//做各种函数接收数据结束标志位处理
    	/*.................*/
	}
}
```

## 不通过中断直接接收发数据

不通过中断的话就不需要定义中断，直接再函数里面调用getchar()函数进行读取数据即可

但是这种方式有比较大的缺陷，就是只能读取1字节的数据，因为缺少了判断的标志位，无法获取是否完成读取，但是很多时候一个字符就已经足够判断

```
	uint8_t str;
	str = getchar();
	printf("str:%c\n", str);
```

