---
title: STM32的SPI学习
tag: SPI
date: 2024-08-30
categories: STM32
index_img: https://s2.loli.net/2024/08/15/b89FBs1i47y5VDI.jpg
---

# STM32的SPI学习

## SPI基本知识

SPI协议是由摩托罗拉公司提出的通讯协议(Serial Peripheral Interface)，即串行外围设备接口， 是一种高速全双工的通信总线。它被广泛地使用在ADC、LCD等设备与MCU间，要求通讯速率较高的场合。

SPI通讯使用3条总线及片选线，3条总线分别为SCK、MOSI、MISO，片选线为SS

SCK：时钟线，用于同步消息和检测消息，传输速率也跟时钟线有关

MOSI：Master Output， Slave Input，以主机为主，所以是主机是输出端，传感器的输入端

MISO：Master Input,，Slave Output，以主机为主，所以是主机是输入端，传感器的输出端

SS：片选线，也称为NSS、CS，主要用于识别设备的标志，可以用自带的片选线，也可以随便找一个GPIO口作为片选线

## 通信流程图

![image.png](https://s2.loli.net/2024/08/26/MhngUwW54dCKuEy.png)

通讯的时候需要将片选线拉低，标志SPI通信开始，这个也是主机告诉某个选定的从机开始通信，如果信号线被拉高，标志停止通信， MOSI与MISO的信号只在NSS为低电平的时候才有效，在SCK的每个时钟周期MOSI和MISO传输一位数据。

SPI使用MOSI及MISO信号线来传输数据，使用SCK信号线进行数据同步。MOSI及MISO数据线在SCK的每个时钟周期传输一位数据， 且数据输入输出是同时进行的。

## SPI初始化结构体

SPI的使用主要有6个步骤，定义GPIO和SPI的结构体，初始化GPIO和SPI的时钟，设置GPIO配置，设置SPI配置，使能SPI，拉高片选线

这几个步骤跟其他的也很类似，所以就不需要过多描述，其中有个点，就是GPIO引脚的设置，可以查阅STM32官方手册中文版111页，有推荐的GPIO口的配置，例如复用推挽、浮空输入等等模式

![image.png](https://s2.loli.net/2024/09/01/CTVcqPOJjdlEIAr.png)

根据这个表格我们就知道NSS、SCK、MOSI、MISO的GPIO模式设置应该如何设置

```
void SPI_config(void)
{
	GPIO_InitTypeDef SPI_GPIO_InitStructure;  //初始化GPIO结构体
	SPI_InitTypeDef  SPI_InitStructure;  //初始化I2C结构体
	
	//打开GPIOB的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	//打开SPI的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_SPI1, ENABLE);
	
	//配置I2C NSS引脚
	SPI_GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	SPI_GPIO_InitStructure.GPIO_Pin = GPIO_Pin_4;
	SPI_GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &SPI_GPIO_InitStructure);
	
	//配置I2C SCK引脚
	SPI_GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
	SPI_GPIO_InitStructure.GPIO_Pin = GPIO_Pin_5;
	SPI_GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &SPI_GPIO_InitStructure);
	
	//配置I2C MOSI引脚
	SPI_GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
	SPI_GPIO_InitStructure.GPIO_Pin = GPIO_Pin_7;
	SPI_GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &SPI_GPIO_InitStructure);
	
	//配置I2C MISO引脚
	SPI_GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;
	SPI_GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
	SPI_GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &SPI_GPIO_InitStructure);
	
	//配置SPI 结构体设置
	SPI_InitStructure.SPI_BaudRatePrescaler = SPI_BaudRatePrescaler_2; //设置波特率为72/4=36MHz
	SPI_InitStructure.SPI_CPHA = SPI_CPHA_2Edge; //设置为偶数边沿采集
	SPI_InitStructure.SPI_CPOL = SPI_CPOL_High; //设置为上升沿采集数据
	SPI_InitStructure.SPI_CRCPolynomial = 0; //不进行校验，所以可以随便设置
	SPI_InitStructure.SPI_DataSize = SPI_DataSize_8b; //配置数据传输大小为8字节
	SPI_InitStructure.SPI_Direction = SPI_Direction_2Lines_FullDuplex; //SPI模式为双线全双工模式
	SPI_InitStructure.SPI_FirstBit = SPI_FirstBit_MSB; //配置为高位先行
	SPI_InitStructure.SPI_Mode = SPI_Mode_Master; //配置主机主发送
	SPI_InitStructure.SPI_NSS = SPI_NSS_Soft; //片选线配置为软件设置
	SPI_Init(SPI1, &SPI_InitStructure); //初始化SPI1的结构体
	
	//使能SPI
	SPI_Cmd(SPI1, ENABLE);
	
	//直接拉高片选线，等到需要的时候进行拉低
	SPI_CS_GPIO_HIGH;
}
```

在代码的SPI结构体部分，一共有10个步骤

1.设置波特率，最大波特率是40MHz，SPI1是挂载在APB2上面，所以最大是72MHz，选择2分频

2.设置CPHA，选择位偶数边沿采集，还有一个是奇数边沿采集，主要为时钟的奇偶段进行数据的检测校验，两种选项都可以，没太大区别

3.设置CPOL，选择是上升沿检测还是下降沿检测，主要就是引脚低变高或者高变低的时候去检测数据

4.设置校验位，不进行设置，这边可以写校验逻辑，对数据进行按特定方法进行计算校验

5.设置数据的传输字节，是要一次性8位或者16位

6.设置SPI的模式，主要有单线双线，还有全双工或者只发送，只接收等模式

7.设置SPI数据的高八位或者低八位先进行发送，这个得依照读取的设备要求进行配置，有些存储设备只能先读高八位或者低八位

8.设置以什么设备为主，是主机或者从机，默认以主机单片机为主

9.设置片选线的方式，有硬件或者软件方式，硬件方式是单片机写好的，无法更改，软件方式可以随便挑一个GPIO口进行设置

10.初始化结构体，将配置写入初始化中

## SPI传输

SPI传输的话单次是一个字节进行发送，共发送八位数据，采用固件库定义的函数进行发送，传输的过程需要依照时序图进行判断

![image.png](https://s2.loli.net/2024/09/01/2oCk65Rb8BeZJx1.png)

首先要拉低片选线NSS，标志SPI传输开始，之后进行TXE标志位的校验，校验是否将数据放入移位寄存器进行准备发送

再利用函数，将八位数据通过SPI进行发送

发送后，需要校验是否完成发送，但是这个不能使用TXE进行校验，因为数据的在移到移位寄存器的时候TXE会发生变化，会导致标志位识别不对，所以我们直接使用RXNE标志位进行校验，因为必须是发送成功了，才能读取到数据

```
uint8_t SPI_Send_Byte_Data(uint8_t data)
{
	// 校验TXE标志位是否复位，也就是数据发送寄存器是否将数据放置移位寄存器
	while(SPI_I2S_GetFlagStatus(SPI1, SPI_I2S_FLAG_TXE) == RESET)
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(0);
  }
	
	//如果清空，则发送8位数据
	SPI_I2S_SendData(SPI1, data);
	
	// 校验RXNE标志位是否复位，校验该位置可以准确知晓是否发送成功，因为只有发送成功，才会接收到数据
	while(SPI_I2S_GetFlagStatus(SPI1, SPI_I2S_FLAG_RXNE) == RESET)
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(1);
  }
	
	// 返回接收到的数据
	return SPI_I2S_ReceiveData(SPI1);
}
```

而且，因为SPI是全双工模式，我们每发送一个数据，就可以同时接收一个数据，因此我们将读写程序放到一个函数中

当然，如果你不想放到一起也是不行的，因为SPI传输也是依靠时钟的频率进行识别，而时钟的频率只有在主机发送的时候才会产生，所以单独发送可以实现，但是单独读取是不行的

因此我们可以定义一个空值，用于接收时的发送，这个空值没有任何的作用，只是进行启动时钟用于读取

下面是进行FLASH的读取，获取他的ID值

```
#define DUMMY 0x00

uint32_t SPI_Receive_Flash_ID(void)
{
	// 定义将要接收的缓存区
	uint32_t flash_id;
	
	// 将片选信号拉低，标志开始启用SPI传输
	SPI_CS_GPIO_LOW;
	
	// 发送flash标志数据，以此来获取flash的地址信息	
	SPI_Send_Byte_Data(0x9f);
	
	// 接收第一个第一个8位地址数据，flash_id = 0x00 00 00 ef
	flash_id = SPI_Send_Byte_Data(DUMMY);
	
	// 将地址左移八位，空置后两位用于再次接收，flash_id = 0x00 00 ef 00
	flash_id <<= 8;
	
	// 接收第二个第一个8位地址数据，flash_id = 0x00 00 ef 40
	flash_id |= SPI_Send_Byte_Data(DUMMY);
	
	// 将地址左移八位，空置后两位用于再次接收，flash_id = 0x00 ef 40 00
	flash_id <<= 8;
	
	// 接收第三个第一个8位地址数据，flash_id = 0x00 ef 40 17
	flash_id |= SPI_Send_Byte_Data(DUMMY);
	
	// 将片选信号拉高，标志SPI传输结束
	SPI_CS_GPIO_LOW;
	
	// 返回ID值
	return flash_id;
}
```

## 总代码如下

```
bsp_spi.h

#ifndef _BSP_SPI_H
#define _BSP_SPI_H

#include "STM32F10x.h"
#include "bsp_usart.h"

#define DUMMY 0x00

#define  SPI_CS_GPIO_HIGH         GPIO_SetBits( GPIOA, GPIO_Pin_4 )
#define  SPI_CS_GPIO_LOW          GPIO_ResetBits( GPIOA, GPIO_Pin_4 )

static __IO uint32_t  I2CTimeout = 10*(0x1000); 

void SPI_config(void);
uint8_t SPI_Send_Byte_Data(uint8_t data);
uint8_t SPI_Receive_Byte_Data(void);
uint32_t SPI_Receive_Flash_ID(void);

#endif /* _BSP_SPI_H */

```

```
bsp_spi.c

#include "bsp_spi.h"

static uint32_t I2C_TIMEOUT_UserCallback(uint8_t errorCode)
{
  /* Block communication and all processes */
  printf("I2C 等待超时!errorCode = %d",errorCode);
  
  return 0;
}

void SPI_config(void)
{
	GPIO_InitTypeDef SPI_GPIO_InitStructure;  //初始化GPIO结构体
	SPI_InitTypeDef  SPI_InitStructure;  //初始化I2C结构体
	
	//打开GPIOB的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	//打开SPI的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_SPI1, ENABLE);
	
	//配置I2C NSS引脚
	SPI_GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	SPI_GPIO_InitStructure.GPIO_Pin = GPIO_Pin_4;
	SPI_GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &SPI_GPIO_InitStructure);
	
	//配置I2C SCK引脚
	SPI_GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
	SPI_GPIO_InitStructure.GPIO_Pin = GPIO_Pin_5;
	SPI_GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &SPI_GPIO_InitStructure);
	
	//配置I2C MOSI引脚
	SPI_GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
	SPI_GPIO_InitStructure.GPIO_Pin = GPIO_Pin_7;
	SPI_GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &SPI_GPIO_InitStructure);
	
	//配置I2C MISO引脚
	SPI_GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;
	SPI_GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
	SPI_GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &SPI_GPIO_InitStructure);
	
	//配置SPI 结构体设置
	SPI_InitStructure.SPI_BaudRatePrescaler = SPI_BaudRatePrescaler_4; //设置波特率为72/4=18MHz
	SPI_InitStructure.SPI_CPHA = SPI_CPHA_2Edge; //设置为偶数边沿采集
	SPI_InitStructure.SPI_CPOL = SPI_CPOL_High; //设置为上升沿采集数据
	SPI_InitStructure.SPI_CRCPolynomial = 0; //不进行校验，所以可以随便设置
	SPI_InitStructure.SPI_DataSize = SPI_DataSize_8b; //配置数据传输大小为8字节
	SPI_InitStructure.SPI_Direction = SPI_Direction_2Lines_FullDuplex; //SPI模式为双线全双工模式
	SPI_InitStructure.SPI_FirstBit = SPI_FirstBit_MSB; //配置为高位先行
	SPI_InitStructure.SPI_Mode = SPI_Mode_Master; //配置主机主发送
	SPI_InitStructure.SPI_NSS = SPI_NSS_Soft; //片选线配置为软件设置
	SPI_Init(SPI1, &SPI_InitStructure); //初始化SPI1的结构体
	
	//使能SPI
	SPI_Cmd(SPI1, ENABLE);
	
	//直接拉高片选线，等到需要的时候进行拉低
	SPI_CS_GPIO_HIGH;
}

uint8_t SPI_Send_Byte_Data(uint8_t data)
{
	// 校验TXE标志位是否复位，也就是数据发送寄存器是否将数据放置移位寄存器
	while(SPI_I2S_GetFlagStatus(SPI1, SPI_I2S_FLAG_TXE) == RESET)
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(0);
  }
	
	//如果清空，则发送8位数据
	SPI_I2S_SendData(SPI1, data);
	
	// 校验RXNE标志位是否复位，校验该位置可以准确知晓是否发送成功，因为只有发送成功，才会接收到数据
	while(SPI_I2S_GetFlagStatus(SPI1, SPI_I2S_FLAG_RXNE) == RESET)
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(1);
  }
	
	// 返回接收到的数据
	return SPI_I2S_ReceiveData(SPI1);
}

// 读取数据和发送数据一致
// 因为时钟是由主机产生，如果主机没有发送，就不会产生时钟信号，也就无法读取，所以直接在发送的过程进行读取数据
uint8_t SPI_Receive_Byte_Data(void)
{
	// 直接返回读取值
	return SPI_Send_Byte_Data(DUMMY);
}

uint32_t SPI_Receive_Flash_ID(void)
{
	// 定义将要接收的缓存区
	uint32_t flash_id;
	
	// 将片选信号拉低，标志开始启用SPI传输
	SPI_CS_GPIO_LOW;
	
	// 发送flash标志数据，以此来获取flash的地址信息	
	SPI_Send_Byte_Data(0x9f);
	
	// 接收第一个第一个8位地址数据，flash_id = 0x00 00 00 ef
	flash_id = SPI_Send_Byte_Data(DUMMY);
	
	// 将地址左移八位，空置后两位用于再次接收，flash_id = 0x00 00 ef 00
	flash_id <<= 8;
	
	// 接收第二个第一个8位地址数据，flash_id = 0x00 00 ef 40
	flash_id |= SPI_Send_Byte_Data(DUMMY);
	
	// 将地址左移八位，空置后两位用于再次接收，flash_id = 0x00 ef 40 00
	flash_id <<= 8;
	
	// 接收第三个第一个8位地址数据，flash_id = 0x00 ef 40 17
	flash_id |= SPI_Send_Byte_Data(DUMMY);
	
	// 将片选信号拉高，标志SPI传输结束
	SPI_CS_GPIO_LOW;
	
	// 返回ID值
	return flash_id;
}
```

