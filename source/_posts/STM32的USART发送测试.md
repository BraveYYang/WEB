---
title: STM32的USART发送测试
tag: USART
date: 2024-07-30
categories: STM32
index_img: https://s2.loli.net/2024/07/31/CsbJKkLaQt9OgVu.jpg
---

# STM32的USART发送测试

### 通信的基本知识

串行通信：串行是一次性发一个数据过去，所以一般就两根线，一个收一个发，传输距离远，抗干扰能力强，例如UART、IIC、SPI…

并行通信：并行是一次性发多个数据，多根线同时发，传输速度快，但是抗干扰能力弱，传输距离短，例如SDIO(4根线)、FSMC(8根线)…

全双工：可以同时收发数据

半双工：不可以同时收发数据，需要收完再发或者发完再收

单工：只能进行收或者发数据，无法同时进行

同步传输：同步是有时钟信号的，只有时钟使能的时候，信号才有用，但是对时钟要求高，时钟的方波不能有尖峰等

异步传输：有通讯起始位、通讯内容主题、数据校验位、通信停止位

比特率：每秒钟传输二进制位数，bit/s

波特率：每秒钟传输码元个数

码元：要传输的信息

RS232：+15V传输0，-15V传输1，需要DB9接口，使用电平转换芯片，用于工业设备的直接通信，能够利用高电平防止静电干扰

### USART串口初始化配置

USART_TX：发送数据输出引脚 

USART_RX：接收数据输入引脚

USART通常只能收发8位以内数据，最大为9位，第 9 位数据是否有效要取决于 USART控制寄存器 1(USART_CR1) 的 M 位设置，当 M 位为 0 时表示 8 位数据字长，当 M 位为 1 表示 9位数据字长

接下来是代码的编写，步骤为打开GPIO、USART1的时钟、GPIO口初始化、GPIO_USART初始化、串口中断优先级配置（接收数据或者发送数据都需要随时进行触发，在函数运行时也有可能收到或者发送数据，所以需要对USART进行中断配置）、中断串口使能、串口使能

#### 初始化GPIO和GPIO_USART

这里面基本逻辑跟GPIO的设置几乎一样，要注意的是TX串口的模式要改成复用推挽输出，因为他需要做推挽输出，还要配置USART，所以不能简单的设置为推挽输出

RX串口要设置成浮空输入模式，因为不能上升沿或者下降沿，因为会产生错误信号，导致数据错乱

```
	GPIO_InitTypeDef GPIO_InitStructure;  //初始化GPIO结构体
	USART_InitTypeDef USART_InitStructure;  //初始化USART结构体
	
	//打开GPIOA的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	//打开USART的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
	
	//初始化TX的GPIO口
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;  //定义TX引脚
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;  //设置输出模式为复用推挽输出
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;  //设置速度为50MHz
	GPIO_Init(GPIOA,&GPIO_InitStructure);  //将结构体进行定义
	
	//初始化RX的GPIO口
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10;  //定义RX引脚
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;  //设置输入模式位浮空输入
	GPIO_Init(GPIOA,&GPIO_InitStructure);  //将结构体进行定义
```

#### 配置USART

波特率是码元的个数，而不是位数，其他配置可以随便调，但是要注意串口调试助手需要相对应配置

校验位用于对数据正确进行校验，通常是需要的，但是数据较少就无所谓了

```
	USART_InitStructure.USART_BaudRate = 115200;  //配置串口的波特率
	USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;  //配置硬件流控制
	USART_InitStructure.USART_Mode = USART_Mode_Rx | USART_Mode_Tx;  //配置为全双工模式，可同时收发
	USART_InitStructure.USART_StopBits = USART_StopBits_1;  //设置停止位为1位
	USART_InitStructure.USART_Parity = USART_Parity_No;  //设置校验位不打开
	USART_InitStructure.USART_WordLength =USART_WordLength_8b;  //设置数据长度为8位
	USART_Init(USART1, &USART_InitStructure);  //将结构体进行定义
```

#### 中断优先级配置

这个跟上节中断的一样，可以参照中断配置，这个主要为内部中断配置，而不是外部中断配置，因为外部中断是真的中断，USART有属于自己的额外的中断线，所以配置USART约等于配置了外部中断，所以只需要内部中断即可

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
	
	// 使能串口
	USART_Cmd(USART1, ENABLE);	
```

### USART串口发送数据

#### 发送8位数据

直接调用固件库函数，然后使用TXE标志位进行判断是否完成发送

```
void Usart_SendByte(USART_TypeDef* pUSARTx, uint8_t data)
{
	//使用发送函数对字节进行发送
	USART_SendData(pUSARTx, data);
	
	//定义循环函数，不断检测TXE位是否为SET，如果为SET则退出循环，完成单次传输
	while( USART_GetFlagStatus(pUSARTx, USART_FLAG_TXE) == RESET );
}
```

#### 发送16位数据

由于USART只能发送8位数据，所以需要把数据拆成两个8位进行发送，然后发送两次，在接收端可以写函数进行拼接，依旧是使用TXE标志位进行判断是否完成发送，TXE标志位只能判断单次发送情况，所以需要发送一次判断一次

```
//发送16位数据
void Usart_SendDoubleByte(USART_TypeDef* pUSARTx, uint16_t data)
{
	//采用高八位第八位分开发送的方式
	uint8_t temp_h,temp_l;
	
	//使用发送函数对字节进行发送
	temp_h = (data&0xff00) >> 8;//先对数据取值高八位，然后进行偏移
	temp_l = data&(0xff);//直接发送低八位
	
	//使用发送函数对字节进行发送，并利用TXE标志位进行判断
	USART_SendData(pUSARTx, temp_h);
	while( USART_GetFlagStatus(pUSARTx, USART_FLAG_TXE) == RESET );
	USART_SendData(pUSARTx, temp_l);
	while( USART_GetFlagStatus(pUSARTx, USART_FLAG_TXE) == RESET );
}
```

#### 发送字符串

需要有数组和指针的概念，直接用指针依次指向每一字节数据，每一字节数据代表8位，依次发送每一字节的数据

这里需要注意的是要采用TC标志位，因为TXE需要每发送一次判断一次，而TC针对大量数据发送结束后，单次判断即可

```
//发送字符串
void Usart_SendStr(USART_TypeDef* pUSARTx, uint8_t *str)
{
	//定义初始值
	uint8_t i=0;
	//对数组指针进行循环判断
	do
  {
		//单次发送数组的1字节
		Usart_SendByte(pUSARTx, *(str+i));
		//发送完对指向下一字节
		i++;
	}while(*(str+i) != '\0');//判断如果是空行符则退出循环，代表数组的最后一位结束
	
	//连续发送多个数据的需要采用TC标志位进行判断
	while( USART_GetFlagStatus(pUSARTx, USART_FLAG_TC) == RESET );
}
```

