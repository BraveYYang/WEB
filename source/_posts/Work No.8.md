---
title: C语言学习
tag: C/C++
date: 2024-07-31
categories: C/C++
index_img: https://s2.loli.net/2024/07/31/jBIEv4rcQJnSp1K.jpg
---

# C语言学习

### void *memset(void *str,int c, size_t n)用于将一段内存区域设置为指定的值

```
void *memset(void *str, int c, size_t n);
*str  //要指向填充的内存区域指针
c  //代表要填充的值
n  //要被设置的字节数
```

```
    char str[10];
    memset(str, 'a', 9); //将str字符前九个填入字符a，要注意是''号，不能是""，""代表的是字符串
    printf("output: %s\n",str);
    memset(str, '\0' ,10); //将str字符前九个填入字符'\0'或者0，可以将字符串进行清零操作
    printf("output: %s\n",str);
    return 0;
    
//output: aaaaaaaaa
//output:
```

### return 0

**没有返回值的函数为空类型，用`void`表示**。一旦函数的返回值类型被定义为 void，就不能再接收它的值，即若函数没有返回值

返回值的类型必须与函数定义类型一致。例如：在返回类型是**char的函数**中，return后应该是char类型的值。

```
return 表达式; return (表达式); 为了简明，（）一般不写。例如：return 0

return 0；表示程序正常退出，即当return语句提供了一个值时，这个值就成为函数的返回值。
return 1；表示程序异常退出，返回主调函数来处理，继续往下执行。
```

### void *memcpy(void *str1, const void *str2, size_t n)从存储区 **str2** 复制 **n** 个字节到存储区 str1

```
str1  //代表用于存储复制内容的目标数组
str2  //代表被复制内容的目标数组
n  //代表需要从str复制n个字节到str1
```

```
    char str_1[256] = "123456sdadasdadasdasda7454589";
    char str_2[256];
 
    memcpy(&str_2[0] ,str_1 ,strlen(str_1));//strlen(str_1)获取str_1字符串的字节，然后将str_1的字符串复制strlen(str_1)个字节到str_2
    printf("output: %s\n",str_2);
    
//output: aaaaaaaaa
//output: aa
```

### 储存区

```
内存分配可分为三种：静态存储区、栈区、堆区。

1、静态存储区：该内存在程序编译的时候就已经分配好，这块内存在程序的整个运行期间都存在，它主要存放静态数据、全局数据和常量。

2、栈区：它的用途是完成函数的调用。在执行函数时，函数内局部变量及函数参数的存储单元在栈上创建，函数调用结束时这些存储单元自动被释放。

3、堆区：程序在运行时使用库函数为变量申请内存，在变量使用结束后再调用库函数释放内存。动态内存的生存期是由我们决定的，如果我们不释放内存，就会导致内存泄漏。

static char line[LINE_SIZE] = {0};
//静态变量的生命周期贯穿整个程序运行期间，适合需要长期保存数据的场景

chat *line = "0"
//多个指针可以共享相同的字符串字面量，节省内存，字符串字面量是不可变的，试图修改会导致未定义行为。

char str[LINE_SIZE] = "0"
//字符数组在声明时可以初始化为一个特定的字符串，数组的其余部分为未初始化（通常是随机值）。对于局部变量，栈空间有限，大数组可能导致栈溢出
```

### char *strchr(const char *str, int c)用于查找字符串中的一个字符，并返回该字符在字符串中第一次出现的位置

```
str  //所要查询的字符
c  //代表所要查找的字符，单个字符需要用''符号，否则警告
```

```
    char str_1[256] = "asdfghjklzxcvbnm";
    char str_2[256];

    char *find = strchr(str_1, 'z');  //查找str_1里面的z字符，并将指针指向这个位置
    if(find){  //判断是否找到，如果找到，则将其替换成换行符，等于舍弃后面的内容
        *find = '\0';
    }
    memcpy(str_2 ,str_1 ,strlen(str_1));  //将str_1的字符全部复制给str_2，由于上述指针指的位置替换成了换行符，所以到此字符进行截断
    printf("output: %s\n",str_2);  //所以输出的是z前面的字符
    
//output
output: asdfghjkl
```

#### char \*strcpy(char \*dest, const char \*src)把 **src** 所指向的字符串复制到 dest

```
char *strcpy(char *dest, const char *src)

//dest -- 指向用于存储复制内容的目标数组。
//src -- 要复制的字符串。

	char str_1[256] = "123456sdadasdadasdasda7454589";
    char str_3[256];
    printf("output: %s\n",str_1);
    
    strcpy(str_3,str_1);
    printf("output: %s\n",str_3);
    
//output
output: 123456sdadasdadasdasda7454589
output: 123456sdadasdadasdasda7454589
```

strcpy只能复制字符串，而memcpy可以复制任意内容，例如字符数组、整型、结构体、类等

strcpy不需要指定长度，它遇到被复制字符的串结束符"\0"才结束，所以容易溢出。memcpy则是根据其第3个参数决定复制的长度

### strcat()

```
//把 src 所指向的字符串追加到 dest 所指向的字符串的结尾
char *strcat(char *dest, const char *src) 
	//dest -- 指向目标数组，该数组包含了一个 C 字符串，且足够容纳追加后的字符串。
	//src -- 指向要追加的字符串，该字符串不会覆盖目标字符串。
```

### strcmp()

```
//把 str1 所指向的字符串和 str2 所指向的字符串进行比较
int strcmp(const char *str1, const char *str2) 
	//str1 -- 要进行比较的第一个字符串。
	//str2 -- 要进行比较的第二个字符串。
	
	//返回值
	如果返回值小于 0，则表示 str1 小于 str2。
	如果返回值大于 0，则表示 str1 大于 str2。
	如果返回值等于 0，则表示 str1 等于 str2。
```

### 判断式if

根据上面的内容进行假设，由于strchr如果找到，则返回该值所在的位置，如果没有找到，则返回NULL空值

如果我想要他返回真值时定义操作，我的函数可以写为

```
if(find){
    *find = '\0';
}
或者
if(find != NULL){
    *find = '\0';
}
```

如果我想要发返回空值时定义操作，函数可以写为

```
if(find = NULL){
    *find = '\0';
}
或者
if(!find){
    *find = '\0';
}
```

### 正则表达式

正则表达式的使用流程

```
编译正则表达式 regcomp()
匹配正则表达式 regexec()
释放正则表达式 regfree()
```

#### 1.regcomp()

```
//把指定的正则表达式pattern编译成一种特定的数据格式compiled，这样可以使匹配更有效
int regcomp (regex_t *compiled, const char *pattern, int cflags)

//regex_t 是一个结构体数据类型，用来存放编译后的正则表达式，它的成员re_nsub 用来存储正则表达式中的子正则表达式的个数，子正则表达式就是用圆括号包起来的部分表达式。
//pattern 是指向我们写好的正则表达式的指针。
//cflags 有如下4个值或者是它们或运算(|)后的值：
	//REG_EXTENDED 以功能更加强大的扩展正则表达式的方式进行匹配。
	//REG_ICASE 匹配字母时忽略大小写。
	//REG_NOSUB 不用存储匹配后的结果。
	//REG_NEWLINE 识别换行符，这样’$’就可以从行尾开始匹配，’^’就可以从行的开头开始匹配。
```

#### 2.regexec()

```
//在编译正则表达式的时候没有指定cflags的参数REG_NEWLINE，则默认情况下是忽略换行符的，也就是把整个文本串当作一个字符串处理。执行成功返回０。
int regexec (regex_t *compiled, char *string, size_t nmatch, regmatch_t matchptr [], int eflags)

//compiled 是已经用regcomp函数编译好的正则表达式。
//string 是目标文本串。
//nmatch 是regmatch_t结构体数组的长度。
//matchptr regmatch_t类型的结构体数组，存放匹配文本串的位置信息。
//eflags 有两个值
	//REG_NOTBOL 按我的理解是如果指定了这个值，那么’^’就不会从我们的目标串开始匹配。总之我到现在还不是很明白这个参数的意义；
	//REG_NOTEOL 和上边那个作用差不多，不过这个指定结束end of line。
	
//通常我们以数组的形式定义一组这样的结构。因为往往我们的正则表达式中还包含子正则表达式。数组0单元存放主正则表达式位置，后边的单元依次存放子正则表达式位置。	
typedef struct
{
   regoff_t rm_so;
   regoff_t rm_eo;
} regmatch_t;
//成员rm_so 存放匹配文本串在目标串中的开始位置，rm_eo 存放结束位置。

```

#### 3.regfree()

```
//如果是重新编译的话，一定要先清空regex_t结构体。
void regfree (regex_t *compiled)
```

#### 4.regerror

```
//当执行regcomp 或者regexec 产生错误的时候，就可以调用这个函数而返回一个包含错误信息的字符串。
size_t regerror (int errcode, regex_t *compiled, char *buffer, size_t length)

//errcode 是由regcomp 和 regexec 函数返回的错误代号。
//compiled 是已经用regcomp函数编译好的正则表达式，这个值可以为NULL。
//buffer 指向用来存放错误信息的字符串的内存空间。
//length 指明buffer的长度，如果这个错误信息的长度大于这个值，则regerror 函数会自动截断超出的字符串，但他仍然会返回完整的字符串的长度。所以我们可以用如下的方法先得到错误字符串的长度。
```

### void bzero(void \*s, int n);将内存块（字符串）的前n个字节清零

```
    char str_1[256] = "123456sdadasdadasdasda7454589";
    printf("str_1:%s\n",str_1);
    char str_2[256] = bzero(str_1,sizeof(str_1));
    printf("str_2:%s\n",str_2);

//bzero不是标准C库，逐渐被memset替代
```

### int atoi(const char *str)把参数 **str** 所指向的字符串转换为一个整数（类型为 int 型）

```
    char str_1[] = "2147483647";//int的整数值只能是4字节，所以不能超过2147483647
    printf("str_1:%s\n",str_1);

    int str_2 = atoi(&str_1);//转换成整数型
    printf("str_2:%d\n",str_2);
    printf("Size of int = %ld bytes \n", sizeof(str_2));//输出字符的字节长度
```

### 位、字节的关系

```
1 Byte = 8 bit
1 KB = 1024 Byte
1 MB = 1024 KB
1 GB = 1024 MB

8位无符号的最大值就是1111 1111，最大值是255，即就是 (2^8)-1。
然而有符号只是在此基础上让8位中的第一位充当符号位，0代表正数，1代表负数。
8位有符号的最大值就是0111 1111，最大值是+127，即就是 (2^7)-1，
8位有符号的最小值就是1000 0000，最小值是-128，即就是 -2^7，

4字节（32位）
最大值 （2^31）- 1   2147483647
最小值 - 2^31  -2147483648
```

### 指针配合while循环函数实现无限循环处理不同消息

    // 解析数据，定义*answer指针指向klipperAcceptBuffer接收数据
    char *answer = klipperAcceptBuffer;
    //进入字符读取过程循环，读完利用指针指向下一帧，继续开始循环
    while (answer)
    {
    	// 查找数据帧结束符‘0x03’
    	char *answerEnd = strchr(answer, 0x03);
    	
    	......
    	
    	// 继续处理下一个数据帧
    	answer = answerEnd + 1;  
    }

### strstr() 查找第一次出现字符串的位置

```
//在字符串 haystack 中查找第一次出现字符串 needle 的位置
char *strstr(const char *haystack, const char *needle) 
```

### snprintf()

```
//将可变参数(...)按照 format 格式化成字符串，并将字符串复制到 str 中，size 为要写入的字符的最大数目，超过 size 会被截断，最多写入 size-1 个字符。
int snprintf(char *str, size_t size, const char *format, ...) 
	//str -- 目标字符串，用于存储格式化后的字符串的字符数组的指针。
	//size -- 字符数组的大小。
	//format -- 格式化字符串。
```

