---
title: C语言链表学习
tag: C/C++
date: 2024-08-06
categories: C/C++
index_img: https://s2.loli.net/2024/08/02/jlE19dhkmfFrO2U.jpg
---

# C语言链表学习

## 感谢博主：

[数据结构--链表入门超详细解析(简单易懂纯原篇)_链表教学-CSDN博客](https://blog.csdn.net/qq_35664104/article/details/120769681)

## 链表基础知识

链表可以动态的进行存储分配，也就是说，链表是一个功能极为强大的数组，他可以在节点中定义多种数据类型，还可以根据需要随意增添，删除，插入节点

链表都有一个头指针，一般以head来表示，存放的是一个地址。

链表中的节点分为两类，头结点和一般节点，头结点是没有数据域的。

链表中每个节点都分为两部分，一个数据域，一个是指针域。

head指向第一个元素：第一个元素又指向第二个元素；……，直到最后一个元素，该元素不再指向其它元素，它称为“表尾”，它的地址部分放一个“NULL”（表示“空地址”），链表到此结束。

## 链表初始化

#### 创建链表

```
typedef struct lint{ //每一个节点其实就是一个结构体，将每个结构体用节点地址关联的方式，进行链接
	int score; //数据域，用于存放数据的位置
	struct lint* next; //指针域，定义指向下一个节点地址的指针
} LinkList;
```

采用typedef的方式，能够不用每次使用struct student  a，直接LinkList a的方式对a进行定义

链表中有一个特殊的结点–头结点。头结点的数据域一般不用来存放和其他节点同类的数据，它的诞生是为了在实际应用过程中存放一些数据+

***p代表这个地址上面的值，p代表这个地址，a[]代表这个地址上面的值，a代表这个数组的地址**

#### 无头结点的初始化

直接从数据节点开始，直接定义第一个数据节点的值和下一个节点的地址

```
Lint * initLint()
{
    // 创建头指针
    Lint* p = NULL;
    
  	// 创建一个临时指针初始化首元结点，并且用于移动和调整后续结点
	Lint* temp = (Lint*)malloc(sizeof(Lint));//创建一个结构体，并分配内存空间
  	temp->score = 90;//将数据填入数据域当中
  	temp->next = NULL;//下一节点地址指向NULL
  	
  	// 头指针需要指向首元结点，这样才能定位这串链表的位置
  	p = temp;
  	
  	// 手动创建10个元素
    for(int i = 0;i < 10;i++)
    {
        // 从第二个结点开始创建
        Lint * a = (Lint*) malloc(sizeof(Lint));
        a->score = i + 91;
        a->next = NULL;
        
        // 将当前temp指向的结点的next指向下一个结点
        temp->next = a; //temp->next这个的意思是在temp中有一个next地址，将他指向a的地址
        
        // temp移到下一个结点。这样在下次循环运行到上一行(line17)时能持续
        temp = temp->next;  // 或 temp = a;
    }
    return p;
}
```

#### 有头节点的初始化

首先定义一个头结点，头结点没有数据域，只有地址域，直接赋值地址域

```
Lint * initLint()
{
    // 创建头指针，并用其创建头结点
    Lint * p = (Lint*)malloc(sizeof (Lint));
    // 及时指定指针的指向以确保指针安全
    p->next = NULL;
    // 创建一个临时的指针指向头结点以进行后续的操作
    Lint * temp = p;
    for(int i = 0;i < 10;i++){
        // 从首元结点开始创建
        Lint * a = (Lint*) malloc(sizeof(Lint));
        a->score = i + 90;
        a->next = NULL;
        // 将当前temp指向的next指向下一个结点
        temp->next = a;
        // temp移到下一个结点
        temp = temp->next;  // 或 temp = a;
    }
    return p;
}
```

## 链表数据改变

#### 输出链表

输出链表其实就是在遍历结构体中的地址域，到达该地址域之后，将其打印出来，然后再跳转指向下一个地址域

```
void showLint(Lint* p){	
  	// 一开始p指针在头结点，这个数据域是空的，因此需要检测其指针域中是否有下一个节点的地址。如果为空，则退出循环
    while (p->next){
      	// p从头结点移出，到首元结点开始输出
        p = p->next;
        printf("%d\t", p->score);
    }
    printf("\n");
}
```

#### 增加数据

增加结点的逻辑是：将新增结点的指针域指向后一个结点，然后将原链表中的前一个结点的指针域指向新增的结点。(**注意顺序不能颠倒，否则会导致插入位置后面的结点全部丢失**)

```
Lint* insertLint(Lint* p,int n, int num){		// 将num插入到第n个位置
    Lint * temp = p;
    // 通过遍历将temp指针移到指向插入位置的直接前驱结点的位置
    for (int i = 1;i < n;i++){
        // 防止超过链表现有长度
        if (temp->next == NULL){
            printf("位置错误!\n");
            return p;
        }
        temp = temp->next;
    }
    // 创建插入的新结点
    Lint * a = (Lint*) malloc(sizeof (Lint));
    a->score = num;
  	// 新节点数据域指向原位置后一个结点
    a->next = temp->next;
  	// temp后移一个结点
    temp->next = a;
    return p;
}

//main函数输出
    int n, num;
    printf("请输入插入的位置和数字，用空格隔开:");
    scanf("%d%d", &n, &num);
    insertLint(p, n, num);
    showLint(p);
```

#### 删除数据

删除链表数据的意思其实也是先找到想要删除的地址域，从节点中一个一个往下遍历，比如想删除第五个节点，那就遍历到第四个节点，然后在第四个节点的地址域上直接指向到第五节点的地址域，这样，第五个节点的地址就在第四个节点上丢失，从而第四个节点地址域获取的就是第六个节点的地址

```
Lint* delLint(Lint* p,int n){
    Lint * temp = p;
    // 用于存储临时删除的结点
    Lint * back = NULL;
    // temp移到删除结点的前一个节点
    for (int i = 1;i < n;i++){
        // 防止越过链表
        if (temp->next == NULL){
            printf("位置错误!\n");
            return p;
        }
        //逐个跳转地址域
        temp = temp->next;
    }
    // 临时存储被删除结点,防止丢失
    back = temp->next;
  	// 指向下下个结点
    temp->next = temp->next->next;
    // 手动释放内存防止泄露
    free(back);
    return p;
}

//main函数输出
    printf("请输入删除第几个元素:");
    scanf("%d", &n);
    delLint(p, n);
    showLint(p);
```

#### 修改数据

修改数据其实跟上面的一样，指到这个数据域之后，直接对其数据进行修改即可

```
Lint* changeLint(Lint* p,int n, int num){  	
	// temp可以一开始就指向首元结点    
	Lint * temp = p->next;    
	for (int i = 1;i < n;i++){        
	// 防止越过链表        
		if (temp->next == NULL){            
			printf("位置错误!\n");            
			return p;        
		}        
		temp = temp->next;    
	}    
	// 修改    
	temp->score = num;    
	return p;
}

//main函数输出
    printf("请输入修改第几个元素为什么数,空格隔开:");
    scanf("%d%d", &n, &num);
    changeLint(p, n, num);
    showLint(p);
```

#### 查找数据

查的基本思想是指针进行遍历链表，对每个节点的数据域进行对比，如果相同就可以直接退出返回结果了。如果一直到链表结束还没有匹配成功就说明没有找到。

```
int searchLint(Lint* p, int num){
    Lint * temp = p;
    int i;  // 用于记数
    for(i = 1;temp->next != NULL;i++){
      	// 因为是从头结点开始的，因此要先移到首元结点
        temp = temp->next;
        if (temp->score == num)return i;	// 返回i表示找到的位置
    }
    return 0;	// 返回0表示未找到
}

//main函数输出
    printf("请输入需要查询的分数:");
    scanf("%d", &num);
    if (!searchLint(p, num))
        printf("未查询到该分数");
    else
        printf("在第%d个元素", searchLint(p, num));
```

如果你的头结点直接在数据域上面标记是1，那可以直接利用数据域的数据进行查找数据

```
int searchLint(Lint* p, int num){
    Lint * temp = p;
    for(p->score = 1;temp->next != NULL;p->score++){
        temp = temp->next;
        if (temp->score == num)return p->score;
    }
    return 0;
}
```

