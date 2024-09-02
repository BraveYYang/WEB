---
title: STM32的I2C通信学习
tag: I2C
date: 2024-08-18
categories: STM32
index_img: https://s2.loli.net/2024/08/15/LZ7VJDqv4sKMilB.jpg
---

# STM32的I2C通信学习

## 基本知识

IIC总线上最好接一个4.7k的上拉电阻，可以防止互相干扰，SDL、SCL都接，上拉电阻接3.3v电源

I2C的使用主要有两根总线，一根是SDL，一根是SDA，SDL上面主要连接的是时钟，SDA主要用来发送数据

所有的设备都是连接在这两个总线上面，单次只能实现一个数据通信，因为通信会占用总线

设备的对接基于I2C的地址分配，每个传感器在出厂的时候基本上是固定好I2C的地址，所以我们只需要输入发送这个地址，就能通过I2C找到这个设备，并把数据发给他

在两个设备通信的过程中，其他的设备都处于高阻态状态，也就是等于断路，如果需要发送数据时，将会连接退出高阻态模式

I2C在通信的过程中由于SDA线他发送数据时存在的干扰性比较强，所以其他设备无法知晓其是否发送成功，并且也无法表示什么时候开始发送，什么时候结束

这个时候就需要SCL总线来协助，SCL是时钟线，当时钟线拉高的时候，SDA线也发送标志位，才表示数据开始发送，并且每个发送的位数据都必须在SCL总线有反应的情况下才算是数据，其他情况皆为干扰信号，不进行读取

STM32的I2C外设可用作通讯的主机及从机，支持100Kbit/s和400Kbit/s的速率，支持7位、10位设备地址， 支持DMA数据传输，并具有数据校验功能。

## I2C系统配置

因为I2C的引脚还是GPIO，所以我们在使用的时候，需要配置GPIO口，如之前配置所示

```
	GPIO_InitTypeDef I2C_GPIO_InitStructure;  //初始化GPIO结构体

	//打开GPIOB的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	
	//配置I2C SCL引脚
	I2C_GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_OD;
	I2C_GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
	I2C_GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &I2C_GPIO_InitStructure);
	
	//配置I2C SDL引脚
	I2C_GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_OD;
	I2C_GPIO_InitStructure.GPIO_Pin = GPIO_Pin_7;
	I2C_GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &I2C_GPIO_InitStructure);
```

在选择上需要注意的是，引脚需要所有都改成复用浮空输入，复用代表着这个原先是GPIO的引脚被复用到了IIC所以，需要改成复用浮空输入

在使用I2C引脚，就需要打开I2C的时钟

```
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
```

之后就需要配置I2C引脚的初始化

I2C的配置主要为ACK使能，这个配置会在每次发送单字节数据结束后，发送一个方波，用来给下位机判别字节的结束，这个类似于换行符

```
	I2C_InitStructure.I2C_Ack = I2C_Ack_Enable;//配置i2c的应答使能
```

I2C的地址通常配置为7位，因为还有一位决定的是读和写，总共加起来8位数据

```
	I2C_InitStructure.I2C_AcknowledgedAddress = I2C_AcknowledgedAddress_7bit;//配置接收字节为7，还有一位是读写位
```

时钟的配置上可以直接选择400k，其中的计算过程如下

```
	I2C_InitStructure.I2C_ClockSpeed = 400000;//配置配置SCL时钟频率为400k
```

```
    标准模式：
    Thigh=CCR*TPCKL1 Tlow = CCR*TPCLK1
    快速模式中 Tlow/Thigh=2 时：
    Thigh = CCR*TPCKL1 Tlow = 2*CCR*TPCKL1
    快速模式中 Tlow/Thigh=16/9 时：
    Thigh = 9*CCR*TPCKL1 Tlow = 16*CCR*TPCKL1

    例如，我们的PCLK1=36MHz，想要配置400Kbit/s的速率，计算方式如下：

    PCLK时钟周期： TPCLK1 = 1/36000000
    目标SCL时钟周期： TSCL = 1/400000
    SCL时钟周期内的高电平时间： THIGH = TSCL/3
    SCL时钟周期内的低电平时间： TLOW = 2*TSCL/3
    计算CCR的值： CCR = THIGH/TPCLK1 = 30
```

再就是占空比设置，这个影响不大，可以直接默认即可

```
	I2C_InitStructure.I2C_DutyCycle = I2C_DutyCycle_2;//配置占空比位2:1
```

模式采用I2C模式，还有一种SMBA通信，几乎不使用，所以直接配置I2C模式

```
	I2C_InitStructure.I2C_Mode = I2C_Mode_I2C;//配置为I2C模式，其他模式不考虑
```

我们还需要给MCU分配一个地址，这个是主机地址，可以用来和主机通信的地址

```
	I2C_InitStructure.I2C_OwnAddress1 = 0x0A;//给主机一个地址，用于识别，随便定义七位的就行
```

完成所有结构体值的赋值，就需要对结构配置

```
	I2C_Init(I2C1, &I2C_InitStructure);//结构体初始化
```

配置的最后一步是对I2C进行使能，以供使用

```
	// 使能串口
	I2C_Cmd (I2C1, ENABLE);	
```

## I2C单字节和多字节发送

I2C的发送主要遵循两个图，单字节的和多字节的去别其实在于数据的多次发送

![image.png](https://s2.loli.net/2024/08/18/oAU1rEYX9MjDPp4.png)

我们实验里面采用的是EEPROM的读写，所以还需要发送EEPROM的读写位置，但是这个读写位置并不是他的地址，而是类似数据格式进行发送，由EEPROM自动判断其位置

因此从这个图可以看出来，我们的发送步骤是

1.发送起始位信号

```
	//发送起始信号
	I2C_GenerateSTART(I2C1,ENABLE);
```

2.使用EV5校验SB位是否置1，以此确定起始信号是否发送成功

```
	//EV5是校验起始信号发送情况
	while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_MODE_SELECT))
  	{
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(8);//超时等待函数，在最下面有介绍
  	}
```

3.发送设备地址(地址最后一位是是写方向)

```
	//开始发送七位地址，发送的函数内容包括总线、地址和方向，这个是写入，所以选择I2C_Direction_Transmitter
	I2C_Send7bitAddress(I2C1,0xA0,I2C_Direction_Receiver);
```

4.使用EV6校验ADDR是否置1，以此确定地址数据是否发送成功(这里面我们不对EV8进行校验，他是校验数据是否发送成功的，地址起始也是一个数据，但是我们有专门的数据位进行校验，因此就校验一次即可)

```
	//使用EV6校验ADDR是否置1，以此确定地址数据是否发送成功
	while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_RECEIVER_MODE_SELECTED))//EV6是检测地址是否发送成功
  	{
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(9);
  	}
```

5.发送数据

```
	//这边发送的也是地址（但是我们使用的还是用数据发送），但是是EEPROM的要写的具体位置地址，而不是EEPROM的整体地址
	I2C_SendData(I2C1,addr);
```

6.（仅限单数据）发送数据，并使用EV8校验TxE是否置1，以此确定数据是否发送成功(在这边可能会出现一个bug，就是EV8和EV8_2的标识符太像了，导致你忘记用EV8_2，就会出现数据无法读取，所以我们有一个妙招，所有的数据位不用EV8校验，全部用EV8_2校验即可)

```
	//这边发送的也是地址（但是我们使用的还是用数据发送），但是是EEPROM的要写的具体位置地址，而不是EEPROM的整体地址
	I2C_SendData(I2C1,addr);
	
	//EV8_2最后位数据校验是否发送成功
	while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTED))
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(4);
  }
```

6.（仅限多数据）使用循环判断，直到数据完全发送成功

```
	//循环执行发送函数，直到发送完毕
	while(NumByteToRead--)  
  {
	//发送数据
    I2C_SendData(I2C1, *data);
    //指针右移一位准备接收下一个数据
    data++; 
		//EV8是数据是否发送成功(EEPROM的地址以数据形式进行发送)
		while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTING))
		{
			if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(14);
		}
  }
```

7.发送停止位信号

```
	//发送结束信号
	I2C_GenerateSTOP(I2C1,ENABLE);
```

## I2C读取数据

读取数据的话，也是遵循一个读取的图表

![image.png](https://s2.loli.net/2024/08/18/ZlPnMOIbRqB52iS.png)

接收的步骤是

1.发送起始位信号

```
	//发送起始信号
	I2C_GenerateSTART(I2C1,ENABLE);
```

2.使用EV5校验SB位是否置1，以此确定起始信号是否发送成功

```
	while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_MODE_SELECT))
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(8);
  }
```

3.发送设备地址(地址最后一位是是读方向)

```
	//开始发送七位地址，发送的函数内容包括总线、地址和方向，这个是写入，所以选择I2C_Direction_Receiver
	I2C_Send7bitAddress(I2C1,0xA0,I2C_Direction_Receiver);
	
	//使用EV6校验ADDR是否置1，以此确定地址数据是否发送成功
	while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_RECEIVER_MODE_SELECTED))//EV6是检测地址是否发送成功
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(9);
  }
```

4.使用EV6校验ADDR是否置1，以此确定地址数据是否发送成功(这里面我们不对EV8进行校验，他是校验数据是否发送成功的，地址起始也是一个数据，但是我们有专门的数据位进行校验，因此就校验一次即可)

```
	//使用EV6校验ADDR是否置1，以此确定地址数据是否发送成功
	while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_RECEIVER_MODE_SELECTED))//EV6是检测地址是否发送成功
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(9);
  }
```

5.读取数据，使用循环判断，直到数据接收完成，在最后一位的时候，关闭ACK信号，并且发送停止位

```
	while(NumByteToRead)  
  	{
		//如果只有一位数据，则直接发送结束信号和停止信号
        if(NumByteToRead == 1)
        {
          //读取数据有个地方比较特殊，就是读到最后一位的时候，会发送NA信号，所以需要将ACK关闭
          I2C_AcknowledgeConfig(I2C1, DISABLE);

          //发送停止位
          I2C_GenerateSTOP(I2C1, ENABLE);
        }
    
		//EV7校验数据是否接收成功
		while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_RECEIVED))//EV6是检测地址是否发送成功
		{
			if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(10);
		}
        //接收数据到指针
        *data = I2C_ReceiveData(I2C1);

        //指针右移一位准备接收下一个数据
        data++; 

        //数据长度减少，等到没有数据时，退出函数
        NumByteToRead--;  
  	}
```

6.重启应答信号，方便下次使用

```
	//重启应答信号，防止下次接收失败
  I2C_AcknowledgeConfig(I2C1, ENABLE);
```

## I2C发送接收校验

这个主要用于防止数据发送过程中出现死循环情况，出错会返回错误码

```
	static __IO uint32_t  I2CTimeout = 10*(0x1000); 
	static uint32_t I2C_TIMEOUT_UserCallback(uint8_t errorCode)
```

## I2C发送超时等待函数

这个函数主要用在发送或者接收的使用，由于MCU的执行速度很快，可能还没有完全写完或者读完，就执行下一程序，导致数据错误，所以需要编写一个等待函数，使用标志位进行读取，读取到代表已经全部写入或者读取，既可以继续执行其他程序

```
void EEPROM_WaitForWriteEnd(void)
{
	do
	{
		//产生起始信号
		I2C_GenerateSTART(I2C1,ENABLE);
		
		//判断SB位是否被重置
		while(I2C_GetFlagStatus (I2C1,I2C_FLAG_SB) == RESET);
		
		//EV5事件被检测到，发送设备地址
		I2C_Send7bitAddress(I2C1,0xA0,I2C_Direction_Transmitter);
	}  
	while(I2C_GetFlagStatus (I2C1,I2C_FLAG_ADDR) == RESET );

	//EEPROM内部时序完成传输完成
	I2C_GenerateSTOP(I2C1,ENABLE);	
}
```

## 总程序代码

### I2C.c

```
#include "bsp_i2c.h"
#include "bsp_usart.h"

static __IO uint32_t  I2CTimeout = 10*(0x1000); 

static uint32_t I2C_TIMEOUT_UserCallback(uint8_t errorCode)
{
  /* Block communication and all processes */
  printf("I2C 等待超时!errorCode = %d",errorCode);
  
  return 0;
}

void I2C_config(void)
{
	GPIO_InitTypeDef I2C_GPIO_InitStructure;  //初始化GPIO结构体
	I2C_InitTypeDef  I2C_InitStructure;  //初始化I2C结构体
	
	//打开GPIOB的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	
	//打开I2C的时钟
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_I2C1, ENABLE);
	
	//配置I2C SCL引脚
	I2C_GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_OD;
	I2C_GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
	I2C_GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &I2C_GPIO_InitStructure);
	
	//配置I2C SDL引脚
	I2C_GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_OD;
	I2C_GPIO_InitStructure.GPIO_Pin = GPIO_Pin_7;
	I2C_GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &I2C_GPIO_InitStructure);
	
	//初始化I2C结构体
	I2C_InitStructure.I2C_Ack = I2C_Ack_Enable;//配置i2c的应答使能
	I2C_InitStructure.I2C_AcknowledgedAddress = I2C_AcknowledgedAddress_7bit;//配置接收字节为7，还有一位是读写位
	I2C_InitStructure.I2C_ClockSpeed = 400000;//配置配置SCL时钟频率为400k
	I2C_InitStructure.I2C_DutyCycle = I2C_DutyCycle_2;//配置占空比位2:1
	I2C_InitStructure.I2C_Mode = I2C_Mode_I2C;//配置为I2C模式，其他模式不考虑
	I2C_InitStructure.I2C_OwnAddress1 = 0x0A;//给主机一个地址，用于识别，随便定义七位的就行
	I2C_Init(I2C1, &I2C_InitStructure);//结构体初始化
	
	// 使能串口
	I2C_Cmd (I2C1, ENABLE);	
}

uint32_t I2C_ByteWrite(uint8_t addr, uint8_t data)
{
	/************************************************************/
	
	//发送起始信号
	I2C_GenerateSTART(I2C1,ENABLE);

	//EV5是校验起始信号发送情况
	while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_MODE_SELECT))
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(1);
  }
	
	/************************************************************/

	//开始发送七位地址，发送的函数内容包括总线、地址和方向，这个是写入，所以选择I2C_Direction_Transmitter
	I2C_Send7bitAddress(I2C1,0xA0,I2C_Direction_Transmitter);

	//使用EV6校验ADDR是否置1，以此确定地址数据是否发送成功
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED ))//EV6是检测地址是否发送成功
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(2);
  }

	/************************************************************/

	//这边发送的也是地址（但是我们使用的还是用数据发送），但是是EEPROM的要写的具体位置地址，而不是EEPROM的整体地址
	I2C_SendData(I2C1,addr);
	
	//EV8是数据是否发送成功(EEPROM的地址以数据形式进行发送)
	while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_BYTE_TRANSMITTED ))
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(3);
  }
	
	/************************************************************/

	//发送数据
	I2C_SendData(I2C1,data);
	
	//EV8_2最后位数据校验是否发送成功
	while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTED))
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(4);
  }
	
	/************************************************************/
	
	//发送结束信号
	I2C_GenerateSTOP(I2C1,ENABLE);
	
	return 0;
}

uint32_t I2C_ByteRead(uint8_t addr,uint8_t *data,uint8_t NumByteToRead)
{
	
	/************************************************************/
	
	//发送起始信号
	I2C_GenerateSTART(I2C1,ENABLE);
	
	//EV5是校验起始信号发送情况
	while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_MODE_SELECT))
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(8);
  }
	
	/************************************************************/
	
	//开始发送七位地址，发送的函数内容包括总线、地址和方向，这个是写入，所以选择I2C_Direction_Transmitter
	I2C_Send7bitAddress(I2C1,0xA0,I2C_Direction_Receiver);
	
	//使用EV6校验ADDR是否置1，以此确定地址数据是否发送成功
	while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_RECEIVER_MODE_SELECTED))//EV6是检测地址是否发送成功
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(9);
  }
	
	/************************************************************/
	
	while(NumByteToRead)  
  {
		//如果只有一位数据，则直接发送结束信号和停止信号
    if(NumByteToRead == 1)
    {
      //读取数据有个地方比较特殊，就是读到最后一位的时候，会发送NA信号，所以需要将ACK关闭
      I2C_AcknowledgeConfig(I2C1, DISABLE);
      
      //发送停止位
      I2C_GenerateSTOP(I2C1, ENABLE);
    }
    
		//EV7校验数据是否接收成功
		while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_RECEIVED))//EV6是检测地址是否发送成功
		{
			if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(10);
		}
    //接收数据到指针
    *data = I2C_ReceiveData(I2C1);

    //指针右移一位准备接收下一个数据
    data++; 
      
    //数据长度减少，等到没有数据时，退出函数
    NumByteToRead--;  
  }
	
	/************************************************************/
	
	//重启应答信号，防止下次接收失败
  I2C_AcknowledgeConfig(I2C1, ENABLE);
  
  return 0;
}

uint32_t I2C_PageWrite(uint8_t addr,uint8_t *data,uint8_t NumByteToRead)
{
	/************************************************************/
	
	//发送起始信号
	I2C_GenerateSTART(I2C1,ENABLE);
	
	//EV5是校验起始信号发送情况
	while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_MODE_SELECT))
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(11);
  }
	
	/************************************************************/
	
	//开始发送七位地址，发送的函数内容包括总线、地址和方向，这个是写入，所以选择I2C_Direction_Transmitter
	I2C_Send7bitAddress(I2C1,0xA0,I2C_Direction_Transmitter);
	
	//这边跳过ACK位，因为他自动产生脉冲，我们使用EV8对他进行检测，这边选择忽略，不检测
	while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED))//EV6是检测地址是否发送成功
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(12);
  }
	
	/************************************************************/
	
	//这边发送的也是地址（但是我们使用的还是用数据发送），但是是EEPROM的要写的具体位置地址，而不是EEPROM的整体地址
	I2C_SendData(I2C1,addr);
	
	//EV8是数据是否发送成功(EEPROM的地址以数据形式进行发送)
	while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTING))
  {
    if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(13);
  }
	
	/************************************************************/
	
	//循环执行发送函数，直到发送完毕
	while(NumByteToRead--)  
  {
		//发送数据
    I2C_SendData(I2C1, *data);
    //指针右移一位准备接收下一个数据
    data++; 
		//EV8是数据是否发送成功(EEPROM的地址以数据形式进行发送)
		while(!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTING))
		{
			if((I2CTimeout--) == 0) return I2C_TIMEOUT_UserCallback(14);
		}
  }
	
	/************************************************************/
	
	//发送结束信号
	I2C_GenerateSTOP(I2C1,ENABLE);
	
	return 0;
}

void EEPROM_WaitForWriteEnd(void)
{
	do
	{
		//产生起始信号
		I2C_GenerateSTART(I2C1,ENABLE);
		
		while(I2C_GetFlagStatus (I2C1,I2C_FLAG_SB) == RESET);
		
		//EV5事件被检测到，发送设备地址
		I2C_Send7bitAddress(I2C1,0xA0,I2C_Direction_Transmitter);
	}  
	while(I2C_GetFlagStatus (I2C1,I2C_FLAG_ADDR) == RESET );

	//EEPROM内部时序完成传输完成
	I2C_GenerateSTOP(I2C1,ENABLE);	
}
```

### I2C.h

```
#ifndef __BSP_I2C_H
#define __BSP_I2C_H

#include "stm32f10x.h"

void I2C_config(void);
uint32_t I2C_ByteWrite(uint8_t addr, uint8_t data);
uint32_t I2C_ByteRead(uint8_t addr,uint8_t *data,uint8_t NumByteToRead);
uint32_t I2C_PageWrite(uint8_t addr,uint8_t *data,uint8_t NumByteToRead);
void EEPROM_WaitForWriteEnd(void);

#endif  /* __BSP_I2C_H */
```

### main.c

```
#include "stm32f10x.h"//相当于51单片机中的#include <reg51.h>
#include "bsp_led.h"
#include "bsp_rccclkconfig.h"
#include "bsp_exti.h"
#include "bsp_systick.h"
#include "bsp_usart.h"
#include <stdio.h>
#include "bsp_dma.h"
#include "bsp_usart.h"
#include "bsp_i2c.h"

uint8_t readData[8]={0};
uint8_t writeData[8]={4,5,6,7,8,9,10,11};

int main (void)
{	
	uint8_t i=0;
  /*初始化USART 配置模式为 115200 8-N-1，中断接收*/
  USART_Config();
	
	/* 发送一个字符串 */
	printf("这是一个IIC通讯实验\n");
	
	//初始化IIC
	I2C_config();

	//写入一个字节
	I2C_ByteWrite(11,55);
	
	//等待写入操作完成
	EEPROM_WaitForWriteEnd();
	
	//写入一个字节
	I2C_ByteWrite(12,52);
	
	//等待写入操作完成
	EEPROM_WaitForWriteEnd();
	
	//addr%8 == 0 ,即为地址对齐
	I2C_PageWrite(16,writeData,8);
	
	//等待写入操作完成
	EEPROM_WaitForWriteEnd();
	
	//读取数据
	I2C_ByteRead(16,readData,8);
	
	
	for(i=0;i<8;i++)
	{	
		printf("%d ",readData[i]);	
	}
	printf("111111111111111111");
	
  while(1)
	{	
		
	}	
}
```

