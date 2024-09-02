---
title: Socket通信流程
tag: C/C++
date: 2024-08-15
categories: Socket通信
index_img: https://s2.loli.net/2024/08/15/qPDXg6Z5IRdjzYO.jpg
---

# Socket通信流程

## 服务端

#### 1.创建套接字

```
int sockfd; //创建套接字接收区
sockfd = socket(AF_INET, SOCK_STREAM, 0); //创建套接字
//判断是否创建成功，成功返回0
if (sockfd < 0) {
}
```

#### 2.配置服务器地址

```
//配置地址结构体，将端口、地址输入到结构体当中
struct sockaddr_in serv_addr; 
//清除结构体内部数据，防止干扰
memset(&serv_addr, 0, sizeof(serv_addr));
//结构体内部成员赋值
serv_addr.sin_family = AF_INET;
serv_addr.sin_port = htons(PORT); // 替换PORT为具体的端口号
serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1"); // 替换为实际的IP地址
```

#### 3.绑定套接字（仅用于服务器端）

```
//将套接字和地址结构体进行绑定
int socket_bind = bind(sockfd, (struct sockaddr *)&serv_addr, sizeof(serv_addr)
//判断是否绑定成功，成功返回0
if (socket_bind < 0) {
}
```

#### 4.启动监听（仅限服务端）

```
//启动对socket的监听，5代表同时监听设备最大数为5个，超出部分直接屏蔽
int socket_listen = listen(sockfd, 5);.
//判断是否监听成功，成功返回0
if (socket_listen < 0) {
}
```

#### 5.接受设备（仅限服务端）

```
//定义一个接受成功返回值储存区
int new_socket;
//定义一个连接设备的地址结构体
struct sockaddr_in client_addr;
//计算连接设备结构体的长度
socklen_t addr_len = sizeof(client_addr);
//启动对设备的接受操作
new_socket = accept(sockfd, (struct sockaddr *)&client_addr, &addr_len);
//判断是否接受成功，成功返回0
if (new_socket < 0) {
}
```

#### 6.接收数据

```
//定义存储数据区
char buffer[1024] = {0};
//启动读取函数，读取并存储
int valread = read(sockfd, buffer, sizeof(buffer));
```

#### 7.发送数据

```
//定义发送字符串
char *message = "Hello, Server";
//将字符串发送到下位机
send(sockfd, message, strlen(message), 0);
```

#### 8.关闭设备

```
//关闭socket通信
close(sockfd);
```

## 客户端

#### 1.创建套接字

```
int sockfd; //创建套接字接收区
sockfd = socket(AF_INET, SOCK_STREAM, 0); //创建套接字
//判断是否创建成功，成功返回0
if (sockfd < 0) {
}
```

#### 2.配置服务器地址

```
//配置地址结构体，将端口、地址输入到结构体当中
struct sockaddr_in serv_addr; 
//清除结构体内部数据，防止干扰
memset(&serv_addr, 0, sizeof(serv_addr));
//结构体内部成员赋值
serv_addr.sin_family = AF_INET;
serv_addr.sin_port = htons(PORT); // 替换PORT为具体的端口号
serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1"); // 替换为实际的IP地址
```

#### 3.连接客户端（仅限客户端）

```
//用于连接客户端的设备
int socket_connect = connect(sockfd, (struct sockaddr *)&serv_addr, sizeof(serv_addr))
////判断是否连接成功，成功返回0
```

#### 4.接收数据

```
//定义存储数据区
char buffer[1024] = {0};
//启动读取函数，读取并存储
int valread = read(sockfd, buffer, sizeof(buffer));
```

#### 5.发送数据

```
//定义发送字符串
char *message = "Hello, Server";
//将字符串发送到下位机
send(sockfd, message, strlen(message), 0);
```

#### 6.关闭设备

```
//关闭socket通信
close(sockfd);
```
