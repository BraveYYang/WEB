---
title: Linux信号signal学习
tag: 信号signal
date: 2024-08-09
categories: Linux
index_img: https://s2.loli.net/2024/08/02/5ZbTO7NnIPkCox8.png
---

# Linux信号signal学习

### 感谢博主：

[Linux 信号(Signal)-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/2363224)

我们经常会使用 `kill` 命令杀掉运行中的进程，对多次杀不死的进程进一步用 `kill -9` 干掉它。

你可能知道这是在用 `kill` 命令向进程发送信号，优雅或粗暴的让进程退出。

我们能向进程发送很多类型的信号，其中一些常见的信号 **SIGINT** 、**SIGQUIT**、 **SIGTERM** 和 **SIGKILL** 都是通知进程退出

### 什么是信号

信号（Signal）是 Linux 进程收到的一个通知。当进程收到一个信号时，该进程会中断其执行，并执行收到信号对应的处理程序。

信号机制作为 Linux 进程间通信的一种方法。Linux 进程间通信常用的方法还有管道、消息、共享内存等。

#### 信号的产生有多种来源：

硬件来源，例如 CPU 内存访问出错，当前进程会收到信号 SIGSEGV；按下 `Ctrl+C` 键，当前运行的进程会收到信号 SIGINT 而退出；

软件来源，例如用户通过命令 `kill [pid]`，直接向一个进程发送信号。进程使用系统调用 [int kill(pid_t pid, int sig)](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fman7.org%2Flinux%2Fman-pages%2Fman2%2Fkill.2.html&source=article&objectId=2363224) 显示的向另一个进程发送信号。内核在某些情况下，也会给进程发送信号，例如当子进程退出时，内核给父进程发送 SIGCHLD 信号。

#### 系统信号种类

```
$ kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX
```

### 信号和中断

信号处理是一种典型的**异步事件处理方式**：进程需要提前向内核注册信号处理函数，当某个信号到来时，内核会就执行相应的信号处理函数。

我们知道，硬件中断也是一种内核的**异步事件处理方式**。当外部设备出现一个必须由 CPU 处理的事件，如键盘敲击、数据到达网卡等，内核会收到中断通知，暂时打断当前程序的执行，跳转到该中断类型对应的中断处理程序。中断处理程序是由 BIOS 和操作系统在系统启动过程中预先注册在内核中的。

中断和信号通知都是在内核产生。中断是完全在内核里完成处理，而**信号的处理则是在用户态完成的**。也就是说，内核只是将信号保存在进程相关的数据结构里面，在执行信号处理程序之前，需要从内核态切换到用户态，执行完信号处理程序之后，又回到内核态，再恢复进程正常的运行。

可以看出，中断和信号的严重程度不一样。信号影响的是一个进程，信号处理出了问题，最多是这个进程被干掉。而中断影响的是整个系统，一旦中断处理程序出了问题，可能整个系统都会挂掉。

### 信号处理

一旦有信号产生，进程对它的处理都有下面三个选择。

1. **执行缺省操作（Default）**。Linux 为每个信号都定义了一个缺省的行为。例如，信号 SIGKILL 的缺省操作是 Term，也就是终止进程的意思。信号 SIGQUIT 的缺省操作是 Core，即终止进程后，通过 Core Dump 将当前进程的运行状态保存在文件里面。
2. **捕捉信号（Catch）**。这个是指让用户进程可以注册自己针对这个信号的处理函数。当信号发生时，就执行我们注册的信号处理函数。
3. **忽略信号（Ignore）**。当我们不希望处理某些信号的时候，就可以忽略该信号，不做任何处理。

有两个信号例外，对于 **SIGKILL** 和 **SIGSTOP** 这个两个信号，进程是无法捕捉和忽略，它们用于在任何时候中断或结束某一进程。**SIGKILL** 和 **SIGSTOP** 为内核和超级用户提供了删除任意进程的特权。
