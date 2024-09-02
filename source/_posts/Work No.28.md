---
title: Protobuf-c使用
tag: C/C++
date: 2024-08-28
categories: protobuf通信
index_img: https://s2.loli.net/2024/08/15/fDkGW65qImzX3xi.jpg
---

# Protobuf-c使用

## Protobuf基本了解

Protobuf是Google提供一个具有高效的协议数据交换格式工具库，类似于JSON，Protobuf和XML、JSON序列化的方式不同，采用了二进制字节的序列化方式，用字段索引和字段类型通过算法计算得到字段之前的关系映射，从而达到更高的时间效率和空间效率，特别适合对数据大小和传输速率比较敏感的场合使用。

protobuf的字符是二进制，无法直接打印出来，但是解包和传输速率更快，传输大小和速度只有JSON的一半，解包只需要一次性解包即可，但是JSON数据的解包需要层层解包，但是数据可以直接打印出来进行调试查看

## Protobuf使用

protobuf在使用的时候不需要太多操作，配置完protobuf-c的环境后，在所需放置的文件夹下面新建一个message.proto的文件

```
//message.proto

syntax = "proto2";  // proto3 必须加此注解
message student
{
    required string id = 1;
    required string name = 2;
    required string gender = 3;
    required int32  age = 4;
    required string object = 5;
    required string home_address = 6;
    required string phone = 7;
}

//实体结构（message）: 代表了实体结构，由多个消息字段组成。
//消息字段（field）: 包括数据类型、字段名、字段规则、字段唯一标识、默认值
//数据类型：常见的原子类型都支持(在FieldDescriptor::kTypeToName中有定义)
//字段规则：(在FieldDescriptor::kLabelToName中定义)
//required：必须初始化字段，如果没有赋值，在数据序列化时会抛出异常
//optional：可选字段，可以不必初始化。
//repeated：数据可以重复(相当于java 中的Array或List)
//字段唯一标识：序列化和反序列化将会使用到。
//默认值：在定义消息字段时可以给出默认值。
```

之后进入该位置终端，输入

```
protoc-c --c_cout=.  message.proto
```

系统会自动生成两个序列文件

```
message.pb-c.c
message.pb-c.h
```

这两个文件并不需要去认真关注，因为他们只是生成后的文件，如果需要修改数据格式，数据类型等等，要在message.proto文件里面定义好，然后进行生成即可

## protobuf在其他函数中使用

初始化protobuf

```
// 初始化Protocol Buffer消息
ProtoMsgCmd protobuff = PROTO_MSG_CMD__INIT;
```

复制消息

```
// 复制命令字符串到Protocol Buffer消息中
protobuff.buffcmd = strdup(cmdStr);
```

定义存放编码缓存区

```
uint8_t Bitbuff[BITBUFF_MAX_LEN] = {0}; // 存放编码后的消息数据
```

进行protobuf序列化

```
// 将Protocol Buffer消息打包到Bitbuff中
size_t Bitbufflen = proto_msg_cmd__pack(&protobuff, Bitbuff);
```

进行protobuf反序列化

```
// 初始化反序列化消息结构
ProtoMsgCmd *unpackedMsg;

// 获取消息中序列化的Protocol Buffer数据
uint8_t *Bitbuff = buffStr; // 假设buffStr是保存序列化数据的字段
size_t Bitbufflen = buffLen; // 假设buffLen是保存序列化数据长度的字段
```

进行解包

```
// 解包
protobuff  = proto_msg_cmd__unpack(NULL, Bitbufflen, Bitbuff);
if (protobuff  == NULL) {
	printf("Error unpacking protocol buffer message %d: %s\n", errno, strerror(errno));
}
```

输出数据

```
// 处理解包后的数据，例如输出解包后的命令字符串
if (protobuff->buffcmd != NULL) {
	printf("Received Command: %s\n", protobuff->buffcmd);
} else {
	printf("No command string found!\n");
}
```

释放内存，防止出现内存泄漏

```
// 完成后释放消息结构内存
proto_msg_cmd__free_unpacked(protobuff , NULL);
```



















