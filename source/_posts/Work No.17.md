---
title: Linux系统进程线程学习
tag: 进程和线程
date: 2024-08-09
categories: Linux
index_img: https://s2.loli.net/2024/08/02/1PqnXLKIcmsuZzO.png
---

# Linux系统进程线程学习

## 感谢博主

[Linux之进程与线程详解(一文足矣)_linux下进程和线程-CSDN博客](https://blog.csdn.net/weixin_53447537/article/details/129372775)

[进程和线程的区别(超详细)-CSDN博客](https://blog.csdn.net/ThinkWon/article/details/102021274)

[进程、子进程、线程-CSDN博客](https://blog.csdn.net/msg1122/article/details/82911644)

## 进程与线程的基本知识

一个在内存中运行的应用程序。每个进程都有自己独立的一块内存空间，一个进程可以有多个线程，比如在Windows系统中，一个运行的xx.exe就是一个进程。

进程中的一个执行任务（控制单元），负责当前进程中程序的执行。一个进程至少有一个线程，一个进程可以运行多个线程，多个线程可共享数据。

同类的多个线程共享进程的**堆**和**方法区**资源，但每个线程有自己的**程序计数器**、**虚拟机栈**和**本地方法栈**，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。

进程是操作系统资源分配的基本单位，而线程是处理器任务调度和执行的基本单位

Linux 进程之间的通信和同步需要使用进程间通信机制（如**管道、信号、共享内存**等），因为它们之间是相互独立的。而线程之间可以直接共享进程的地址空间，因此它们之间的通信和同步要比进程更加高效。

**在调试时，进程比线程更容易处理**。由于进程是相互独立的，因此可以分别调试每个进程，而线程共享进程的地址空间和资源，因此在调试线程时，需要特别注意线程之间的交互和影响，这使得调试线程变得更加困难。

## 进程与线程的区别总结

线程具有许多传统进程所具有的特征，故又称为轻型进程(Light—Weight Process)或进程元；而把传统的进程称为重型进程(Heavy—Weight Process)，它相当于只有一个线程的任务。在引入了线程的操作系统中，通常一个进程都有若干个线程，至少包含一个线程。

**根本区别：**进程是操作系统资源分配的基本单位，而线程是处理器任务调度和执行的基本单位

**资源开销：**每个进程都有独立的代码和数据空间（程序上下文），程序之间的切换会有较大的开销；线程可以看做轻量级的进程，同一类线程共享代码和数据空间，每个线程都有自己独立的运行栈和程序计数器（PC），线程之间切换的开销小。

**包含关系：**如果一个进程内有多个线程，则执行过程不是一条线的，而是多条线（线程）共同完成的；线程是进程的一部分，所以线程也被称为轻权进程或者轻量级进程。

**内存分配：**同一进程的线程共享本进程的地址空间和资源，而进程之间的地址空间和资源是相互独立的

**影响关系：**一个进程崩溃后，在保护模式下不会对其他进程产生影响，但是一个线程崩溃整个进程都死掉。所以多进程要比多线程健壮。

**执行过程：**每个独立的进程有程序运行的入口、顺序执行序列和程序出口。但是线程不能独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制，两者均可并发执行

### 父进程、子进程、线程的区别

在Windows下开一个IE浏览器，这个IE浏览器是一个进程。你用浏览器去打开一个pdf，IE就去调用Acrobat去打开，这时Acrobat是一个独立的进程，就是IE的子进程。而IE自己本身同时用同一个进程开了2个网页，并且同时在跑两个网页上的脚本，这两个网页的执行就是IE自己通过两个线程实现的。

一个独立程序的运行称为一个进程， 在进程里并发执行的不同部分称为线程。 由这个进程引发的另外的独立程序运行为这个进程的子进程

父进程还是可以通过和子进程通信来获得一些信息的。 拿上面的例子来说，IE可以通过一些进程间通信的接口来知道Acrobat是否顺利的把pdf打开了之类的信息。但有一点我觉得你的理解基本正确，就是父进程和子进程是独立的。假如IE开了一个病毒子进程，子进程不听话，父进程也没什么特别的办法，除了向系统申请去关闭它之外。

## 进程创建和使用

### 进程的创建函数

```
#include <sys/types.h>
#include <unistd.h>

//创建一个进程
pid_t fork(void);
//在父进程中，fork返回新创建子进程的进程ID；
//在子进程中，fork返回0；
//如果出现错误，fork返回一个负值;

//得到一个进程的父进程的PID;
getppid();
//得到当前进程的PID;
getpid();
```

fork函数执行完毕后，如果创建新进程成功，则出现两个进程，一个是子进程，一个是父进程。在子进程中，fork函数返回0，在父进程中，fork返回新创建子进程的进程ID。我们可以通过fork返回的值来判断当前进程是子进程还是父进程。

### 进程退出

在main函数中使用 `return关键字` ，使用 `return` 后系统会调用 `exit()`函数来终止进程。或者调用 `_exit()` 来终止进程。

_exit会立刻结束进程将缓存释放掉，而exit先查看当前进程有没有文件缓存区，如果有，会先处理缓存区的数据，然后释放内存

```
void _exit(int status);

void exit(int status);
//exit(0):表示运行正常结束进程；exit(1):表示异常退出，返回值1是给操作系统的
```

return 是关键字，是语言级别，表示调用堆栈的返回，并将控制权移交给控制的前一级；exit是函数，表示系统级别，表示进程的结束。

### 子进程等待和销毁处理

父进程一旦调用了wait就立即阻塞自己，由wait自动分析是否当前进程的某个子进程已经退出，如果让它找到了这样一个已经变成僵尸的子进程，wait就会收集这个子进程的信息，并把它彻底销毁后返回；如果没有找到这样一个子进程，wait就会一直阻塞在这里，直到有一个出现为止。

```
#include <sys/types.h>
#include <sys/wait.h>

pid_t wait(int *wstatus);
//正常情况下，wait的返回值为子进程的PID，如果调用进程没有子进程，调用就会失败，此时wait返回-1，同时errno被置为ECHILD
```

当父进程忘了用wait()函数等待已终止的子进程时,子进程就会进入一种无父进程的状态,此时子进程就是僵尸进程.

如果先终止父进程,子进程将继续正常进行，只是它将由init进程(PID 1)继承,当子进程终止时,init进程捕获这个状态.

参数status用来保存被收集进程退出时的一些状态，它是一个指向int类型的指针。但如果我们对这个子进程是如何死掉毫不在意，只想把这个僵尸进程消灭掉，（事实上绝大多数情况下，我们都会这样想），我们就可以设定这个参数为NULL，

```
pid = wait(NULL);
//如果成功，wait会返回被收集的子进程的进程ID，如果调用进程没有子进程，调用就会失败，此时wait返回-1，同时errno被置为ECHILD。
```

如果参数status的值不是NULL，wait就会把子进程退出时的状态取出并存入其中， 这是一个整数值（int），指出了子进程是正常退出还是被非正常结束的，以及正常结束时的返回值，或被哪一个信号结束的等信息。由于这些信息 被存放在一个整数的不同二进制位中，所以用常规的方法读取会非常麻烦，人们就设计了一套专门的宏（macro）来完成这项工作

WIFEXITED(status) 这个宏用来指出子进程是否为正常退出的，如果是，它会返回一个非零值。这里的参数status并不同于wait唯一的参数–指向整数的指针status，而是那个指针所指向的整数

WEXITSTATUS(status) 当WIFEXITED返回非零值时，我们可以用这个宏来提取子进程的返回值，如果子进程调用exit(5)退出，WEXITSTATUS(status) 就会返回5；如果子进程调用exit(7)，WEXITSTATUS(status)就会返回7。如果进程不是正常退出的，也就是说， WIFEXITED返回0，这个值就毫无意义。

```
/*
下面的程序没有调用wait函数，那么子进程会成为无父进程状态，也就是僵尸进程，可以根据程序运行后
所打印的PID，然后命令行输 ps -aux 查看进程状态，
发现这个子进程依然存在，所以要调用wait来进行所谓的收尸
*/

int main(){
	pid_t pid;//定义fork的返回值,存储在相同类型的变量pid中
//如果创建成功，在父进程中fork返回新创建的子进程的进程号，在子进程中返回0
	printf("hello world\n");
	pid=fork();//fork创建子进程，将返回值给pid
	if(pid<0)//如果出错，fork返回一个负值并设置错误码
		perror("fork error");
	else if(pid==0)//判断pid==0就是子进程
		printf("I am child,my pid:%d\n",getpid());//getpid得到当前进程的进程号,getppid()是得到父进程的进程ID
	else if(pid>0)//如果pid>0是父进程
		printf("I am father,my pid:%d\n",pid);//
 
	puts("hi ");//这里是父子进程都执行的部分,运行结果可以看到两个hi,
	exit(0);
}
```

```
/*
下面的程序调用了wait来为孩子收尸，这样再ps -aux会发现进程结束，没有尸体残留
*/

int main(){
	pid_t pid;//定义fork的返回值,存储在相同类型的变量pid中
	printf("hello world\n");
	pid=fork();//fork创建子进程，将返回值给pid
	if(pid<0)//如果出错，fork返回一个负值并设置错误码
		perror("fork error");
	else if(pid==0){//如果创建成功,在子进程中，返回的pid==0  是子进程
		printf("I am child,my pid:%d\n",getpid());//getpid得到当前进程的进程号,getppid()得到父进程的进程号
		printf("my father's pid:%d\n",getppid());
	}
	else if(pid>0){//如果pid>0是父进程，返回的pid 为创建的子进程的ID
		printf("I am father,my son's pid:%d\n",pid);
		printf("my pid:%d\n",getpid());
		int status;
		pid_t sonpid=wait(NULL);
		if(sonpid<0)puts("调用进程没有子进程");
		else printf("等到了孩子的尸体，孩子进程号为:%d\n",sonpid);
 
	}
	puts("hi ");//这里是父子进程都执行的部分,运行结果可以看到两个hi,
	exit(0);
}

//output
hello world
I am father,my son's pid:12345
my pid:6789
hi 
I am child,my pid:12345
my father's pid:6789
hi 
等到了孩子的尸体，孩子进程号为:12345
```

这个程序在子进程创建的时候，等于开了另一个main函数，用于执行这个函数指令，但是子进程不会重新创建子进程，只会执行子进程创建后的命令，比如进行函数判断和输出puts，而父进程也是会执行该操作，因此会输出两个hi

### exec函数族

exec函数族提供一个在进程中启动另一个程序执行的方法。它可以根据指定的文件名或目录名找到可执行文件，并用它来取代原调用进程的数据段、代码段和堆栈段，在执行完之后，原调用进程的内容除了进程号外，其他全部被新程序的内容替换了。另外，这里的可执行文件既可以是二进制文件，也可以是Linux下任何可执行脚本文件。
`fork()` 创建一个子进程后，父进程和子进程共享同一个代码段（程序），但在有些情况下，我们希望子进程运行一个完全不同的程序，而不是继续执行父进程的代码。**子进程执行完新的程序后，它依然需要父进程wait收尸!!!**

exec函数族的函数执行成功后不会返回，调用失败时，会设置errno并返回-1，然后从原程序的调用点接着往下执行。

```
exec函数族：execl 、execlp、execvp
exec族函数参数极难记忆和分辨，函数名中的字符会给我们一些帮助： 
l : 使用参数列表 
p：使用文件名，并从PATH环境进行寻找可执行文件 ,自动搜索环境变量PATH
v：应先构造一个指向各参数的指针数组，然后将该数组的地址作为这些函数的参数。 
e：多了envp[]数组，使用新的环境变量代替调用进程的环境变量,表示自己维护环境变量
```

### 示例

```
//exec_test.c  中的代码，这个程序会在主函数中进程的子进程里面进行替换执行
#include<stdio.h>
#include <unistd.h>
int main(){
	int i=0;
	while(i<6){
		printf("正在执行exec_test.out,此时i:%d\n",i++);
		sleep(1);
	}
	return 0;
}
```

```
//exec.c  文件
#include<stdio.h>
#include<stdlib.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/wait.h>
 #include <unistd.h>
int main(){
	printf("这是刚开始的父进程,进程号:%d\n",getpid());
	pid_t pid=fork();//创建子进程
	if(pid<0){//判断是否成功
		perror("fork error");
	}
	else if(pid==0){
		puts("接下来执行子进程");
		//执行该语句的话，后续父进程的puts就不再执行
		int ret=execl("./exec_test.out","exec_test.out",NULL,NULL);//将子进程即将执行的程序进行替换
		if(ret<0)perror("exec error");//判断是否失败，成功的话都无法来到这个指令
		puts("子进程执行结束");//如果执行失败，则会输出该log
	}
	else if(pid>0){
		printf("这是父进程,父进程pid:%d\n",getpid());
		printf("我创建的子进程的pid:%d\n",pid);
		puts("接下来调用wait");
		  pid_t sonpid=wait(NULL); 
        if(sonpid<0)puts("调用进程没有子进程"); 
        else printf("等到了孩子的尸体，孩子进程号为:%d\n",sonpid);
	}
puts("准备结束父进程");
return 0;
}

//output
这是刚开始的父进程,进程号:4588
这是父进程,父进程pid:4588
创建的子进程的pid:4589
接下来调用wait
接下来执行子进程

正在执行exec_test.out,此时i
...
...
...

等到了孩子的尸体，孩子进程号为:
准备结束父进程
```

**有了exec函数族我们就可以用它来调用任何程序，也就是说，你还可以在C/C++，可以调用其他任何语言，比如python，Java等程序；**

## 线程的创建和使用

当我们使用进程的时候，我们不自主的使用if else嵌套来判断pid，使得程序结构繁琐，但是当我们使用线程的时候，基本上可以甩掉它，当然程序内部执行功能单元需要使用的时候还是要使用，所以线程对程序结构的改善有很大帮助。

### 线程创建函数

```
//创建一个线程
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,void *(*start_routine)(void *), void *arg);
//函数返回值：成功返回0；失败返回错误码

参数解释：
第一个参数：pthread_t *thread
    pthread_t是类型，在linux下被定义为“unsigned long int”，是一种用于表示线程的数据类型；
    phtread_t *thread:传递一个pthread_t类型的指针变量，也可以传递这个类型变量的地址
第二个参数： const pthread_attr_t *attr
    const pthread_attr_t：该类型以结构体的形式定义在<pthread.h>头文件中，专门表示线程的属性
    const pthread_attr_t *attr：用于手动设置新建线程的属性，例如线程的调用策略、线程所能使用的栈内存的大小等。大部分场景中，我们都不需要手动修改线程的属性，将 attr 参数赋值为 NULL，pthread_create() 函数会采用系统默认的属性值创建线程。
 
第三个参数：void *(*start_routine)(void *)
    以函数指针的方式指明新建线程需要执行的函数，该函数的参数最多有 1 个（可以省略不写），形参和返回值的类型都必须为 void 类型。void 类型又称空指针类型，表明指针所指数据的类型是未知的。使用此类型指针时，我们通常需要先对其进行强制类型转换，然后才能正常访问指针指向的数据。
    如果该函数有返回值，则线程执行完函数后，函数的返回值可以由 pthread_join() 函数接收
第四个参数：void *arg
    void *arg：指定传递给 start_routine 函数的实参，当不需要传递任何数据时，将 arg 赋值为 NULL 即可。
```

### 线程退出函数

```
//线程退出函数
void pthread_exit(void *retval);
 
功能说明：主动结束当前线程（终止调用它的线程并返回一个指向某个对象的指针）
 
参数：
 void *retval：函数的返回指针，只要pthread_join中的第二个参数retval不是NULL，这个值将被传递给retval
```

### 线程等待函数

```
//线程等待也叫阻塞
int pthread_join(pthread_t thread, void **retval);

功能说明：
    这个函数是一个线程阻塞的函数，调用它的函数将一直等待到被等待的线程结束为止，当函数返回时，被等待线程的资源被收回，可以理解为进程当中的wait收尸
 
参数：
    pthread_t thread  被等待的线程标识符
    void **retval     一个用户定义的指针，它可以用来存储被等待线程的返回值
返回值：
On success, pthread_join() returns 0; on error,  it  returns  an  error number.
```

### 示例

```
//以下是这三个函数的程序实例：
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<pthread.h>
void *thread1fun(){
	puts("进入线程1");
	puts("执行线程1");
	puts("hello i am thread1");
	char *p="线程1已经退出，返回指针char *p";
	pthread_exit(p);
}
void *thread2fun(){
	puts("进入线程2");
	puts("执行线程2");
	puts("hello i am thread2");
	char *p="线程2已经退出，返回指针char *p";
	pthread_exit(p);
}
int main(){
	pthread_t thread1;
	pthread_t thread2;//定义两个线程类型变量
	//创建线程，开始执行其他线程函数thread1fun
	if(pthread_create(&thread1,NULL,thread1fun,NULL)!=0){//判断是否错误
		perror("thread1_create error");
		exit(1);
	}
	else {
		puts("线程1创建成功");
		printf("线程1线程号：%lu\n",thread1);
	}
 	//创建线程，开始执行其他线程函数thread2fun
	if(pthread_create(&thread2,NULL,thread2fun,NULL)!=0){//判断是否错误
		perror("thread1_create error");
       exit(1);
	}
	else {
		puts("线程2创建成功");
		printf("线程2线程号：%lu\n",thread2);
	}
	
    puts("阻塞等待回收线程");
    if(pthread_join(thread1,(void**)&thread1)==0){
        puts("等到了，打印返回指针");
    	puts((char*) *(&thread1));
    }
    if(pthread_join(thread2,(void **)&thread2)==0){
        puts("等到了，打印返回指针");
    	puts((char*) *(&thread2));
    }
    return 0;
}

//output
线程1创建成功
线程1线程号：<thread1的线程ID>
线程2创建成功
线程2线程号：<thread2的线程ID>
进入线程1
执行线程1
hello i am thread1
进入线程2
执行线程2
hello i am thread2
阻塞等待回收线程
等到了，打印返回指针
线程1已经退出，返回指针char *p
等到了，打印返回指针
线程2已经退出，返回指针char *p
```

在执行线程1的时候，主函数和线程2仍然在执行。

### 线程取消函数

```
//取消某个线程的执行
int pthread_cancel(pthread_t thread);

//返回值：成功返回0，否则返回perror
//参数：要取消线程的标识符ID
```

### 获取线程当前ID

```
//获取当前调用线程的ID
pthread_t pthread_self(void);

//返回值：当前线程的线程ID标识
```

## 线程互斥锁

互斥锁的初始化和销毁：在使用互斥锁之前，需要先对其进行初始化，使用完毕后需要将其销毁。初始化可以使用 `pthread_mutex_init`() 函数，销毁可以使用 `pthread_mutex_destroy`() 函数。

加锁和解锁的配对：在使用 `pthread_mutex_lock`() 和 `pthread_mutex_unlock`() 函数时，需要保证加锁和解锁的配对性，即每次加锁后都必须要对应的解锁。

避免死锁：死锁是指两个或多个线程互相等待对方释放锁，导致程序无法继续执行。为了避免死锁，需要确保**加锁的顺序一致**，并且**不要在加锁的状态下等待其他锁。**

不要重复加锁：重复加锁会导致线程阻塞，无法进行下去，需要注意避免。

对共享资源的访问必须要在加锁状态下进行：为了确保多个线程访问共享资源的互斥性，需要在对共享资源进行访问之前，先获取对应的互斥锁，以避免多个线程同时访问共享资源。

```
pthread_mutex_init()：用于初始化一个互斥锁。可以使用默认属性或者自定义属性进行初始化。
pthread_mutex_destroy()：用于销毁一个互斥锁，释放相关资源。
pthread_mutex_lock()：用于获取一个互斥锁，如果该锁已经被其他线程获取，则当前线程将阻塞，直到该锁被释放为止。
pthread_mutex_unlock()：用于释放一个互斥锁，如果该锁当前没有被任何线程获取，则此函数将返回一个错误。
```

## 线程锁（读写锁）

上面是互斥锁，但是线程并不只有互斥锁，在实际的多线程编程中，我们应该选择合适的线程锁。

互斥锁（Mutex）：互斥锁是一种二元锁，只有两个状态：锁定和未锁定。当一个线程持有互斥锁时，其他线程必须等待该线程释放锁之后才能继续执行。如果多个线程同时竞争锁，只有一个线程可以获得锁，其他线程将被阻塞，直到获得锁的线程释放锁。互斥锁提供了一种简单而有效的方法，用于确保多个线程之间对共享资源的互斥访问。

读写锁（Reader-Writer Lock）：读写锁是一种更高效的锁，适用于读多写少的场景。读写锁允许多个线程同时读取共享资源，但是只有一个线程可以写入共享资源。当一个线程持有读锁时，其他线程也可以持有读锁，但是当一个线程持有写锁时，其他线程必须等待该线程释放写锁之后才能继续执行。读写锁的优点是在读取共享资源时可以允许多个线程同时进行，从而提高了程序的并发性和性能。

自旋锁（Spin Lock）：自旋锁是一种非阻塞锁，线程在尝试获取锁时不会被挂起，而是一直循环尝试获取锁。当锁被其他线程持有时，线程会一直尝试获取锁，直到锁被释放。自旋锁适用于锁被持有时间很短的场景，因为在等待锁的过程中，线程会消耗大量的CPU资源。

条件变量（Condition Variable）：条件变量用于在某个条件满足时通知等待线程。线程可以在条件变量上等待，直到满足条件后才被唤醒。条件变量通常与互斥锁一起使用，以确保在修改共享资源时不会出现竞态条件。

信号量（Semaphore）：信号量是一种计数器，用于控制多个线程对共享资源的访问。当计数器大于零时，线程可以访问资源，并将计数器减一；当计数器等于零时，线程就必须等待。信号量可以用来实现读写锁和生产者-消费者模型等并发编程模型。

```
//此函数用于初始化读写锁对象。读写锁对象在使用前必须被初始化，可以使用静态初始化或者动态初始化。
pthread_rwlock_init()
int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock, const pthread_rwlockattr_t *restrict attr);

    rwlock：读写锁对象的指针。
    attr：读写锁属性对象的指针，可以为NULL。
    返回值：成功返回0，失败返回错误码。


//获取读取锁，如果写入锁已经被占用，则函数会阻塞等待，直到写入锁被释放。多个线程可以同时获取读取锁。
pthread_rwlock_rdlock()
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);

    rwlock：读写锁对象的指针。
    返回值：成功返回0，失败返回错误码。

//此函数用于获取写入锁，如果读取锁或者写入锁已经被占用，则函数会阻塞等待，直到锁被释放。写入锁是独占的，只有一个线程可以获取写入锁。
pthread_rwlock_wrlock()
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);

    rwlock：读写锁对象的指针。
    返回值：成功返回0，失败返回错误码。

//此函数用于释放读取锁或者写入锁。如果锁没有被获取，释放锁会导致未定义的行为。
pthread_rwlock_unlock()
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);

    rwlock：读写锁对象的指针。
    返回值：成功返回0，失败返回错误码。

//此函数尝试获取读取锁，如果读取锁已经被占用，则函数会立即返回，不会阻塞等待。如果获取成功，则进行读取操作。
pthread_rwlock_tryrdlock()
int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);

    rwlock：读写锁对象的指针。
    返回值：成功返回0，失败返回错误码。

//此函数尝试获取写入锁，如果读取锁或者写入锁已经被占用，则函数会立即返回，不会阻塞等待。如果获取成功，则进行写入操作。
pthread_rwlock_trywrlock()
int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);

    rwlock：读写锁对象的指针。
    返回值：成功返回0，失败返回错误码。

//此函数用于销毁读写锁
pthread_rwlock_destroy()
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);

    rwlock：读写锁对象的指针。
    返回值：成功返回0，失败返回错误码。
```

## 进程通信方式

**管道（Pipe）：管道是一种半双工的通信方式，它只能用于父子进程或者兄弟进程之间的通信**。管道有两种类型：无名管道和命名管道。无名管道只存在于相关进程的内存中，而命名管道则存在于文件系统中，允许不相关的进程进行通信。

**命名管道（Named Pipe）：命名管道允许不相关的进程进行通信。**它在文件系统中创建一个特殊文件，任何知道文件名的进程都可以通过文件访问方式进行通信。

**信号（Signal）：信号是一种异步通信机制，用于通知进程发生了某些事件。**一个进程可以向另一个进程发送信号，另一个进程可以选择接受或忽略信号。

**消息队列（Message Queue）：消息队列是一种存放在内核中的消息链表，用于不相关的进程间的通信。**进程可以向队列中发送消息，其他进程可以从队列中接收消息。

**共享内存（Shared Memory）：共享内存允许多个进程访问同一块物理内存。**进程可以将数据写入共享内存区域，其他进程可以直接从该内存区域读取数据。

**套接字（Socket）：套接字是一种通用的进程间通信方式，可用于不同主机之间的通信。**套接字提供了一组接口，进程可以使用这些接口来建立网络连接并进行通信。
