---
title: Linux消息队列学习
tag: 消息队列
date: 2024-08-09
categories: Linux
index_img: https://s2.loli.net/2024/08/02/smPFoif4rdC3xeW.png
---

# Linux消息队列学习

## 感谢博主：

[消息队列（定义、结构、如何创建、消息队列的发送与接收、发送与接收实例）_vc中如何定义使用消息队列-CSDN博客](https://blog.csdn.net/scarificed/article/details/121475146)

## 消息队列概述

官方说法是，MQ（全称Message Queue）是一种进程间通信或同一进程的不同线程间的通信方式，队列就是一个消息容器

现实使用中，我们将消息队列称之为**中间件**，从它的名字就可以看出，消息队列不存储消息内容的本身，它只是消息的搬运工

消息队列是一种先进先出的队列型数据结构，实际上是系统内核中的一个内部链表。消息被顺序插入队列中，其中发送进程将消息添加到队列末尾，接受进程从队列头读取消息。

#### 消息和队列

**消息：**需要传输的数据，可以是最简单的文本字符串，也可以是自定义的复杂格式

**队列：**一种先进先出的数据结构，在这里是存放消息的容器，入队即发消息的过程，出队即收消息的过程

**过程：**生产者将消息投递到一个叫做 队列 的容器中，然后再从容器中取出消息，最后转发给消费者

#### 队列模型（又叫做点对点模型）

**点对点模式（PTP）**：一个生产者发送的每一个消息，都只有一个消费者消费，看起来就像消息从一个点被传递到另一个点，也就是**单播**

它允许多个生产者向同一个队列里发送消息，但是只有一个消费者能拿到数据，如果有多个消费者，那么多个消费者是竞争者关系，最终一条消息只能被其中一个消费者接收到，读完即会被删除

#### 发布订阅模式

一个生产者发送的每一个消息，都会发送到所有订阅了此队列的消费者，也就是**广播**

在发布订阅模型中，存放消息的容器变成了主题（主题也是一个队列），订阅者在接收消息之前需要先订阅主题，最终，每个订阅者都可以收到同一个主题的全量消息，可以看到和队列模式唯一不同的就是，发布订阅模式中的**一份消息数据可以被多次消费**

## 消息队列结构

消息队列中消息本身由消息类型和消息数据组成，通常使用如下结构：

```
struct msgbuf
{
	long 	mtype;//mtype指定了消息类型，为正整数。
	char	mtext[1];//mtext指定了消息的数据。我们可以定义任意的数据类型甚至包括结构来描述消息数据
}
```

**mtype：**引入消息类型之后，消息队列在逻辑上由一个消息链表转化为多个消息链表。发送进程仍然无条件把消息写入队列的尾部，但接收进程却可以有选择地读取某个特定类型的消息中最接近队列头的一个，即使该消息不在队列头。相应消息一旦被读取，就从队列中删除，其它消息维持不变。

#### 创建消息队列

```
int msgget(key_t key, int msgflg);
//参数key是消息队列的关键字
//参数msgflg的低9位指定队列的属主、属组和其他用户的访问权限，其它位指定消息队列的创建方式。
	//IPC_CREAT：创建，如存在则打开；
	//IPC_EXCL：与IPC_CREAT使用，单独使用无意义。创建时，如存在则失败。
	
//创建关键字为0x1234，访问权限为0666的消息队列，如队列已存在返回其标识号。
int msgid;
msgid = msgget(0x1234, 0666|IPC_CREAT);

//创建关键字为0x1234，访问权限为0666的消息队列，如队列已存在则报错。
int msgid;
msgid = msgget(0x1234, 0666|IPC_CREAT|IPC_EXCL);
```

#### 消息队列发送

```
//发送消息
int msgsnd(int msqid, void *msgp, int msgsz, int msgflg);
    //msgid：指定发送消息队列的标识号；
    //msgp：指向存储待发送消息内容的内存地址，用户可设计自己的消息结构；
    //msgsz：指定长度，仅记载数据的长度，不包括消息类型部分，且必须大于0；
    //msgflg：控制消息发送的方式，有阻塞和非阻塞（IPC_NOWAIT）两种方式。
```

#### 消息队列的发送流程

```
//第一步，定义消息结构
struct msgbuf{	
	long mtype;		
	char ctext[100];
}	

//第二步，创建消息队列
int msgid;
msgid = msgget(KEY, 0666|IPC_CREAT);
if(msgid < 0)	//打开或创建消息失败；

//第三步，组织消息
struct msgbuf buf;
buf.mtype = 100;
strcpy(buf.ctext, “HELLO UNIX!”);

//第四步，发送消息
int ret;
ret = msgsnd(msgid, &buf, strlen(buf.ctext), 0);

//第五步，判断发送是否成功
if(ret == -1)
{
	if(errno == EINTR)	//信号中断，重新发送；
	else //系统错误
}
```

#### 从消息队列中接收消息

```
//接收队列消息
int msgrcv(int msgid, void *msgp, int msgsz, long msgtyp, int msgflg);

	//msgid：消息队列标识号；
	//msgp：指向接收消息的内存缓冲区；
	//msgsz：指定该缓冲区的最大容量，不包括消息类型占用的部分；
	//msgtyp：指定读取消息的类型；
```

#### 消息队列的接收流程

```
//第一步，定义消息结构
struct msgbuf{	
	long mtype;		
	char ctext[100];
}	

//打开（创建）消息队列
int msgid;
msgid = msgget(KEY, 0666|IPC_CREAT);

//准备接收消息缓冲区
struct msgbuf buf;
memset(buf, 0, sizeof(buf));

//接收消息
int ret;
ret = msgrcv(msgid, &buf, sizeof(buf.ctext), TYPE, 0);

//接收判断
if(ret == -1)
{
	if(errno == EINTR)	 //信号中断，重新接收；
	else                 //系统错误
}
```

### 消息队列信息情况核实

```
    struct msqid_ds queue_info;
    if (msgctl(ID, IPC_STAT, &queue_info) == -1) {
        printf("Failed to get queue info for control_queue_ID: %d, errno = %d, error: %s\n", ID, errno, strerror(errno));
    } else {
        printf("Control queue status - messages: %ld\n", queue_info.msg_qnum);//检查消息的数量
        printf("Max size of message: %ld\n", queue_info.msg_qbytes);//检查消息的最大限制长度
    }
    
    //output
    Control queue status - messages: 0
```

### 终端查看消息队列

```
ipcs命令
ipcs -a ：显示全部可以显示的信息
ipcs -q：显示活动的消息队列
ipcs -m：显示活动的共享内存信息
ipcs -s：显示活动的信号量信息

ipcrm命令：
ipcrm -m id：删除共享内存标识
ipcrm -M key：删除由关键字创建的共享内存标识
ipcrm -q id ：删除消息队列标识 id和其相关的消息队列和数据结构
ipcrm -Q key：删除由关键字key创建的消息队列和其相关的消息队列和数据结构
ipcs -s id：删除信号标识符id和其相关的信号量集及数据结构
ipcs -S key:删除由关键字key创建的信号量标识及其相关的信号量集及数据结构
```

## 消息队列优势

1、采用消息队列通信比采用管道通信具有更多的灵活性，通信的进程不但没有血缘上的要求，也不需要进行同步处理。
2、消息队列是一种先进先出的队列型数据结构；
3、消息队列将输出的信息进行了打包处理，可以保证以消息为单位进行接收；
4、消息队列对信息进行分类服务，根据消息的类别进行分别处理。
5、提供消息数据自动拆分功能，同时不能接受两次发送的消息。
6、消息队列提供了不完全随机读取的服务。
7、消息队列提供了完全异步的读写服务。
