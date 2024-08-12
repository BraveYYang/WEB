---
title: Python学习-1
tag: Python
date: 2024-08-12
categories: Python
index_img: https://s2.loli.net/2024/08/02/Saf2h5bePzUvB4A.jpg
---

# Python学习-1

## 感谢博主：

[数据类型和变量 - Python教程 - 廖雪峰的官方网站 (liaoxuefeng.com)](https://liaoxuefeng.com/books/python/basic/data-types/index.html)

## Python 简介

**Python 是一种解释型语言：** 这意味着开发过程中没有了编译这个环节。类似于PHP和Perl语言

**Python 是交互式语言：** 这意味着，您可以在一个 Python 提示符 **>>>** 后直接执行代码。

**Python 是面向对象语言:** 这意味着Python支持面向对象的风格或代码封装在对象的编程技术。

**Python 是初学者的语言：**Python 对初级程序员而言，是一种伟大的语言，它支持广泛的应用程序开发，从简单的文字处理到 WWW 浏览器再到游戏。

## 数据类型和变量

转义字符`\`可以转义很多字符，比如`\n`表示换行，`\t`表示制表符，字符`\`本身也要转义，所以`\\`表示的字符就是`\`

这种**变量本身类型不固定的语言称之为动态语言，与之对应的是静态语言**。静态语言在定义变量时必须指定变量类型，如果赋值的时候类型不匹配，就会报错。Python是动态语言，根据上下文判断变量赋值

在Python中，通常用全部大写的变量名表示常量

`/`除法计算结果是浮点数，即使是两个整数恰好整除，结果也是浮点数：

```python
>>> 9 / 3
3.0
```

还有一种除法是`//`，称为地板除，两个整数的除法仍然是整数：

```python
>>> 10 // 3
3
```

整数的地板除`//`永远是整数，即使除不尽。要做精确的除法，使用`/`就可以。

因为`//`除法只取结果的整数部分，所以Python还提供一个余数运算，可以得到两个整数相除的余数：

```python
>>> 10 % 3
1
```

## 列表List

Python内置的一种数据类型是列表：list。list是一种有序的集合，可以随时添加和删除其中的元素。

定义一个列表`classmates`，里面数据包含了班级同学的名字，其实列表等于c语言里面的数组

```python
classmates =['kangkang','lihua',"xiaoming","zhengzheng",'liuyang']
```

求这个列表的长度

```python
>>> len(classmates)
3
```

用索引来访问list中每一个位置的元素，记得索引是从`0`开始的：

```python
>>> classmates[1] 
'lihua'
>>> classmates[0] 
'kangkang'
```

如果索引值是负号，说明索引是倒数执行的

```python
>>> classmates[-2]  
'zhengzheng'
>>> classmates[-1] 
'liuyang'
```

list是一个可变的有序表，所以，可以往list中追加元素到末尾：

```python
>>> classmates.append('chenyu') 
>>> classmates
['kangkang', 'lihua', 'xiaoming', 'zhengzheng', 'liuyang', 'chenyu']
```

也可以把元素插入到指定的位置，比如索引号为`2`的位置：

```python
>>> classmates.insert(2,'yangfeng') 
>>> classmates
['kangkang', 'lihua', 'yangfeng', 'xiaoming', 'zhengzheng', 'liuyang', 'chenyu']
```

要删除list末尾的元素，用`pop()`方法：

```python
>>> classmates.pop()
'chenyu'
>>> classmates 
['kangkang', 'lihua', 'yangfeng', 'xiaoming', 'zhengzheng', 'liuyang']
```

要删除指定位置的元素，用`pop(i)`方法，其中`i`是索引位置：

```python
>>> classmates.pop(2) 
'yangfeng'
>>> classmates
['kangkang', 'lihua', 'xiaoming', 'zhengzheng', 'liuyang']
```

要把某个元素替换成别的元素，可以直接赋值给对应的索引位置：

```python
>>> classmates[1]='jack' 
>>> classmates
['kangkang', 'jack', 'xiaoming', 'zhengzheng', 'liuyang']
```

list里面的元素的数据类型也可以不同，比如：

```python
>>> str= [1,2,3,'clast',3.001,-2]        
>>> str
[1, 2, 3, 'clast', 3.001, -2]
```

list元素也可以是另一个list，比如：

```python
>>> str.insert(4,classmates) 
>>> str
[1, 2, 3, 'clast', ['kangkang', 'jack', 'xiaoming', 'zhengzheng', 'liuyang'], 3.001, -2]
```

## 元组tuple

另一种有序列表叫元组：tuple。tuple和list非常类似，**但是tuple一旦初始化就不能修改**

因为tuple不可变，所以代码更安全。如果可能，能用tuple代替list就尽量用tuple。

当你定义一个tuple时，在定义的时候，tuple的元素就必须被确定下来

如果要定义一个空的tuple，可以写成`()`

要定义一个只有1个元素的tuple，如果你这么定义：

```python
>>> yuanzu = (1) 
>>> yuanzu
1
```

定义的不是tuple，是`1`这个数！这是因为括号`()`既可以表示tuple，又可以表示数学公式中的小括号，这就产生了歧义，因此，Python规定，这种情况下，按小括号进行计算，计算结果自然是`1`。

所以，只有1个元素的tuple定义时必须加一个逗号`,`，来消除歧义：

```python
>>> yuanzu = (1,) 
>>> yuanzu
(1,)
```

查找元组的位置跟列表一致

```python
>>> yuanzu[0] 
1
```

元组里面的数据也是可以添加列表和元组

```python
>>> yuanzu = (0,1,3.33,-5,str)   
>>> yuanzu
(0, 1, 3.33, -5, [1, 2, 3, 'clast', ['kangkang', 'jack', 'xiaoming', 'zhengzheng', 'liuyang'], 3.001, -2])
```

查找元组的个数也跟列表一样，但是如果元组里面包好列表或者元组，这个只会被当作一个元素

```python
>>> yuanzu
(0, 1, 3.33, -5, [1, 2, 3, 'clast', ['kangkang', 'jack', 'xiaoming', 'zhengzheng', 'liuyang'], 3.001, -2])
>>> len(yuanzu) 
5
```

元组可以查找元组单元里面的元组子数据或者列表子数据，只能查找两重

```python
>>> yuanzu         
(0, 1, 3.33, -5, [1, 2, 3, 'clast', ['kangkang', 'jack', 'xiaoming', 'zhengzheng', 'liuyang'], 3.001, -2])
>>> yuanzu[4][4] 
['kangkang', 'jack', 'xiaoming', 'zhengzheng', 'liuyang']
>>>
```

如果元组里面包含了列表，那元组里面的列表值是可以进行修改的

```python
>>> yuanzu[4][2] = 'x' 
>>> yuanzu             
(0, 1, 3.33, -5, [1, 2, 'x', 'clast', ['kangkang', 'jack', 'xiaoming', 'zhengzheng', 'liuyang'], 3.001, -2])
```

元组的数组是不能直接增删的，但是我们可以对里面的列表做增删改查，元组的数据也会相应的变化

```python
>>> yuanzu.pop([4][2]) 
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'tuple' object has no attribute 'pop'
>>> classmates.pop()
'liuyang'
>>> yuanzu
(0, 1, 3.33, -5, [1, 2, 'x', 'clast', ['kangkang', 'jack', 'xiaoming', 'zhengzheng'], 3.001, -2])
```

## 判断和输入

根据Python的缩进规则，如果`if`语句判断是`True`，就把缩进的两行print语句执行了，否则，什么也不做。

#### if、elif、else

也可以给`if`添加一个`else`语句，意思是，如果`if`判断是`False`，不要执行`if`的内容，去把`else`执行了：

`elif`是`else if`的缩写，完全可以有多个`elif`

```python
if`语句执行有个特点，它是从上往下判断，如果在某个判断上是`True`，把该判断对应的语句执行后，就忽略掉剩下的`elif`和`else
```

`input()`读取用户的输入，这样可以自己输入，但是input()的输入值的定义是字符串，如果直接进行比较，就会报错

```python
birth = input('birth: ')
if birth < 2000:
    print('00前')
else:
    print('00后')
```

所以就需要进行字符的转换，用到了int()

```python
s = input('birth: ')
birth = int(s)
if birth < 2000:
    print('00前')
else:
    print('00后')
```

#### match case

多个匹配的时候就需要用到match case语句，和c语言的switch case一样的用法

```python
match score:
    case 'A':
        print('score is A.')
    case 'B':
        print('score is B.')
    case 'C':
        print('score is C.')
    case _: # _表示匹配到其他任何情况
        print('score is ???.')
```

`match`语句还可以匹配列表，功能非常强大。

```python
args = ['gcc', 'hello.c', 'world.c']
# args = ['clean']
# args = ['gcc']

match args:
    # 如果仅出现gcc，报错:
    case ['gcc']:
        print('gcc: missing source file(s).')
    # 出现gcc，且至少指定了一个文件:
    case ['gcc', file1, *files]:
        print('gcc compile: ' + file1 + ', ' + ', '.join(files))
    # 仅出现clean:
    case ['clean']:
        print('clean')
    case _:
        print('invalid command.')
        
第一个case ['gcc']表示列表仅有'gcc'一个字符串，没有指定文件名，报错；

第二个case ['gcc', file1, *files]表示列表第一个字符串是'gcc'，第二个字符串绑定到变量file1，后面的任意个字符串绑定到*files（符号*的作用将在函数的参数中讲解），它实际上表示至少指定一个文件；
如果第一个不是gcc，那么就不能进行判断

第三个case ['clean']表示列表仅有'clean'一个字符串；

最后一个case _表示其他所有情况。
```

## 循环

#### for...in

Python的循环有两种，一种是for...in循环，依次把list或tuple中的每个元素迭代出来

```python
names = ['Michael', 'Bob', 'Tracy']
for x in names:
    print(x)
```

所以`for x in ...`循环就是把每个元素代入变量`x`，然后执行缩进块的语句。

再比如我们想计算1-10的整数之和，可以用一个`sum`变量做累加：

```python
numbers = [1,2,3,4,5,6,7,8,9]
sum = 0
for x in numbers:
    sum =sum + x
    print(sum)
```

#### range()

Python提供一个`range()`函数，可以生成一个整数序列，再通过`list()`函数可以转换为list。比如`range(5)`生成的序列是从0开始小于5的整数：

```python
>>> list(range(5)) 
[0, 1, 2, 3, 4]
>>> list(range(101))
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100]
```

所以就可以直接用if in来累加

```python
sum = 0
for x in range(101):
    sum =sum + x
    print(sum)
```

#### while

第二种循环是while循环，只要条件满足，就不断循环，条件不满足时退出循环。比如我们要计算100以内所有奇数之和，可以用while循环实现：

```python
sum = 0
n = 99
while n > 0:
    sum = sum + n
    n = n - 2
print(sum)
```

在循环中，`break`语句可以提前退出循环。

```python
n = 1
while n <= 100:
    if n > 10: # 当n = 11时，条件满足，执行break语句
        break # break语句会结束当前循环
    print(n)
    n = n + 1
print('END')
```

在循环过程中，也可以通过`continue`语句，跳过当前的这次循环，直接开始下一次循环。

```python
n = 0
while n < 10:
    n = n + 1
    if n % 2 == 0: # 如果n是偶数，执行continue语句
        continue # continue语句会直接继续下一轮循环，后续的print()语句不会执行
    print(n)
```

## 字典

Python内置了字典：dict的支持，dict全称dictionary，在其他语言中也称为map，使用键-值（key-value）存储，具有极快的查找速度。

字典其实也是类似于C语言的结构体，但是使用方法会比结构体更加简单一些

```python
>>> dict = {'lihua': 94,'yangyang': 44,"kangkang": 55} 
>>> dict
{'lihua': 94, 'yangyang': 44, 'kangkang': 55}
```

所以我们就可以根据前面的键值直接找到后面的值

```python
>>> dict['lihua'] 
94
```

为什么dict查找速度这么快？因为dict的实现原理和查字典是一样的。假设字典包含了1万个汉字，我们要查某一个字，一个办法是把字典从第一页往后翻，直到找到我们想要的字为止，这种方法就是在list中查找元素的方法，list越大，查找越慢。

第二种方法是先在字典的索引表里（比如部首表）查这个字对应的页码，然后直接翻到该页，找到这个字。无论找哪个字，这种查找速度都非常快，不会随着字典大小的增加而变慢。

dict就是第二种实现方式，给定一个名字，比如`'Michael'`，dict在内部就可以直接计算出`Michael`对应的存放成绩的“页码”，也就是`95`这个数字存放的内存地址，直接取出来，所以速度非常快。

把数据放入dict的方法，除了初始化时指定外，还可以通过key放入：

```python
>>> dict
{'lihua': 94, 'yangyang': 44, 'kangkang': 55}
>>> dict["liuyang"] = 97
>>> dict                 
{'lihua': 94, 'yangyang': 44, 'kangkang': 55, 'liuyang': 97}
```

由于一个key只能对应一个value，所以，多次对一个key放入value，后面的值会把前面的值冲掉：

```python
>>> dict                 
{'lihua': 94, 'yangyang': 44, 'kangkang': 55, 'liuyang': 97}
>>> dict['liuyang'] = 94
>>> dict
{'lihua': 94, 'yangyang': 44, 'kangkang': 55, 'liuyang': 94}
```

二是通过dict提供的`get()`方法，如果key不存在，可以返回`None`，或者自己指定的value。

```python
>>> dict
{'lihua': 94, 'yangyang': 44, 'kangkang': 55, 'liuyang': 94}
>>> dict.get('yangyang') 
44
```

要想删除字典里面的值，直接用pop选取该字典里面的键值就可以把键和值都删掉

```python
>>> dict
{'lihua': 94, 'yangyang': 44, 'kangkang': 55, 'liuyang': 94}
>>> dict.pop('yangyang')
44
>>> dict                 
{'lihua': 94, 'kangkang': 55, 'liuyang': 94}
```

### set

set和dict类似，也是一组key的集合，但不存储value。由于key不能重复，所以，在set中，没有重复的key。

要创建一个set，用`{x,y,z,...}`列出每个元素：

```python
>>> s = {1, 2, 3}
>>> s
{1, 2, 3}
```

或者提供一个list作为输入集合：

```python
>>> s = set([1, 2, 3])
>>> s
{1, 2, 3}
```

注意，传入的参数`[1, 2, 3]`是一个list，而显示的`{1, 2, 3}`只是告诉你这个set内部有1，2，3这3个元素，显示的顺序也不表示set是有序的。

重复元素在set中自动被过滤：

```python
>>> s = {1, 1, 2, 2, 3, 3}
>>> s
{1, 2, 3}
```

通过`add(key)`方法可以添加元素到set中，可以重复添加，但不会有效果：

```python
>>> s.add(4)
>>> s
{1, 2, 3, 4}
>>> s.add(4)
>>> s
{1, 2, 3, 4}
```

通过`remove(key)`方法可以删除元素：

```python
>>> s.remove(4)
>>> s
{1, 2, 3}
```

set可以看成数学意义上的无序和无重复元素的集合，因此，两个set可以做数学意义上的交集、并集等操作：

```python
>>> s1 = {1, 2, 3}
>>> s2 = {2, 3, 4}
>>> s1 & s2
{2, 3}
>>> s1 | s2
{1, 2, 3, 4}
```

set和dict的唯一区别仅在于没有存储对应的value，但是，set的原理和dict一样，所以，同样不可以放入可变对象，因为无法判断两个可变对象是否相等，也就无法保证set内部“不会有重复元素”。试试把list放入set，看看是否会报错。

## 函数

在Python中，定义一个函数要使用`def`语句，依次写出函数名、括号、括号中的参数和冒号`:`，然后，在缩进块中编写函数体，函数的返回值用`return`语句返回。

假设我们自己去写一个绝对值判断函数

```python
def abs_self(x):
    if x>0:
        print(x)
        return x
    elif x<0:
        print(-x)
        return -x

print(abs_self(-10))
```

函数体内部的语句在执行时，一旦执行到`return`时，函数就执行完毕，并将结果返回。因此，函数内部通过条件判断和循环可以实现非常复杂的逻辑。

如果没有`return`语句，函数执行完毕后也会返回结果，只是结果为`None`。`return None`可以简写为`return`。

在Python交互环境中定义函数时，注意Python会出现`...`的提示。函数定义结束后需要按两次回车重新回到`>>>`提示符下

```python
>>> def zzzz(x):
...     if x>0:
...             return x
...     if x<0:
...             return -x
...
>>> zzzz(-20) 
20
```

如果想定义一个什么事也不做的空函数，可以用`pass`语句：

```python
def nop():
    pass
```

`pass`语句什么都不做，那有什么用？实际上`pass`可以用来作为占位符，比如现在还没想好怎么写函数的代码，就可以先放一个`pass`，让代码能运行起来。

`pass`还可以用在其他语句里，比如：

```python
if age >= 18:
    pass
```

缺少了`pass`，代码运行就会有语法错误。

当传入了不恰当的参数时，内置函数`abs`会检查出参数错误，而我们定义的`zzzz`没有参数检查，会导致`if`语句出错，出错信息和`abs`不一样。所以，这个函数定义不够完善。

修改一下`zzzz`的定义，对参数类型做检查，只允许整数和浮点数类型的参数。数据类型检查可以用内置函数`isinstance()`实现：

```python
>>> def zzzz(x):
...     if not isinstance(x,(int,float)):
...             raise TypeError('Error')
...     if x>0:
...             return x
...     if x<0:
...             return -x
...

>>> zzzz('a') 
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in zzzz
TypeError: Error
```

函数也可以返回多个参数值，只需要在return的时候进行多个值的指定

```python
def text(x,y):
    a=10
    if x>y:
        print(x)
        return x
    elif x<y:
        print(y)
        return x
    elif x==y:
        print(x,y)
        return x ,y,a
print(text(10,10))

//output
10 10
(10, 10, 10)
```

#### 函数的默认参数

很多时候，我们在定义函数的时候，参数的类型很多，因此每次我们调用函数都需要填入多个参数，如果少一个就会报错，因此我们可以在定义函数的时候添加默认参数，这样的话我们只需要调用函数的时候直接输入没有默认的参数，就可以实现函数的调用

```python
def xx(x,n=2):
    c=1
    for i in range(n):
        c=c*x
    return c

print(xx(5))//调用默认参数
print(xx(7,3))//定义其他参数
```

有多个默认参数时，调用的时候，既可以按顺序提供默认参数，比如调用`enroll('Bob', 'M', 7)`，意思是，除了`name`，`gender`这两个参数外，最后1个参数应用在参数`age`上，`city`参数由于没有提供，仍然使用默认值。

也可以不按顺序提供部分默认参数。当不按顺序提供部分默认参数时，需要把参数名写上。比如调用`enroll('Adam', 'M', city='Tianjin')`，意思是，`city`参数用传进去的值，其他默认参数继续使用默认值。

在定义函数的时候，会选择默认参数，默认参数必须是指向不变的值，如果它指向的是可变的值，那么会出现该参数数据混乱的情况，所以使用none来指定一些可变参数转换成固定参数

```python
def add_end(L=None):
    if L is None:
        L = []
    L.append('END')
    return L
```

#### 函数的可变参数

在Python函数中，还可以定义可变参数。顾名思义，可变参数就是传入的参数个数是可变的，可以是1个、2个到任意个，还可以是0个。

常规办法的话可以将函数的参数传入定义成一个列表或者元组，这样我们可以从里面填充这些可变值进行传入，就可以实现类似的可变参数，但是这样的可变参数操作起来比较复杂，需要很多步骤

所以，我们把函数的参数改为可变参数：

```python
def calc(*numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
```

定义可变参数和定义一个list或tuple参数相比，仅仅在参数前面加了一个`*`号。在函数内部，参数`numbers`接收到的是一个tuple，因此，函数代码完全不变。但是，调用该函数时，可以传入任意个参数，包括0个参数：

```python
>>> calc(1, 2)
5
>>> calc()
0
```

如果已经定义好可变参数函数，但是我又想导入一个列表和元组，就可以在列表和元组上添加*符号，代表可变参数，函数会自动遍历该值，进行输出

```python
>>> nums = [1, 2, 3]
>>> calc(*nums)
14
```

`*nums`表示把`nums`这个list的所有元素作为可变参数传进去。这种写法相当有用，而且很常见。

#### 关键字参数

而关键字参数允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict。

```python
def person(name, age, **kw):
    print('name:', name, 'age:', age, 'other:', kw)
```

函数`person`除了必选参数`name`和`age`外，还接受关键字参数`kw`。在调用该函数时，可以只传入必选参数：

```python
>>> person('Michael', 30)
name: Michael age: 30 other: {}
```

也可以传入任意个数的关键字参数：

```python
>>> person('Bob', 35, city='Beijing')
name: Bob age: 35 other: {'city': 'Beijing'}
>>> person('Adam', 45, gender='M', job='Engineer')
name: Adam age: 45 other: {'gender': 'M', 'job': 'Engineer'}
```

和可变参数类似，也可以先组装出一个dict，然后，把该dict转换为关键字参数传进去：

```
>>> extra = {'city': 'Beijing', 'job': 'Engineer'}
>>> person('Jack', 24, city=extra['city'], job=extra['job'])
name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
```

当然，上面复杂的调用可以用简化的写法：

```
>>> extra = {'city': 'Beijing', 'job': 'Engineer'}
>>> person('Jack', 24, **extra)
name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
```

`**extra`表示把`extra`这个dict的所有key-value用关键字参数传入到函数的`**kw`参数，`kw`将获得一个dict，注意`kw`获得的dict是`extra`的一份拷贝，对`kw`的改动不会影响到函数外的`extra`。

#### 命令关键字参数

主要对关键字进行限制，关键字可以传入任意的参数，而命令关键字对他进行限定，限定只能传入限定的关键字

```
def person(name, age, *, city, job):
    print(name, age, city, job)
```

和关键字参数`**kw`不同，命名关键字参数需要一个特殊分隔符`*`，`*`后面的参数被视为命名关键字参数。

如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要一个特殊分隔符`*`了

```
def person(name, age, *args, city, job):
    print(name, age, args, city, job)
```

命名关键字参数必须传入参数名。如果没有传入参数名，调用将报错：

```
>>> person('Jack', 24, 'Beijing', 'Engineer')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: person() missing 2 required keyword-only arguments: 'city' and 'job'
```

由于调用时缺少参数名`city`和`job`，Python解释器把前两个参数视为位置参数，后两个参数传给`*args`，但缺少命名关键字参数导致报错。

命名关键字参数可以有缺省值，从而简化调用：

```
def person(name, age, *, city='Beijing', job):
    print(name, age, city, job)
```

#### 参数组合

在Python中定义函数，可以用必选参数、默认参数、可变参数、关键字参数和命名关键字参数，这5种参数都可以组合使用。但是请注意，参数定义的顺序必须是：必选参数、默认参数、可变参数、命名关键字参数和关键字参数。

```
def f1(a, b, c=0, *args, **kw):
    print('a =', a, 'b =', b, 'c =', c, 'args =', args, 'kw =', kw)

def f2(a, b, c=0, *, d, **kw):
    print('a =', a, 'b =', b, 'c =', c, 'd =', d, 'kw =', kw)
    
>>> f1(1, 2)
a = 1 b = 2 c = 0 args = () kw = {}
>>> f1(1, 2, c=3)
a = 1 b = 2 c = 3 args = () kw = {}
>>> f1(1, 2, 3, 'a', 'b')
a = 1 b = 2 c = 3 args = ('a', 'b') kw = {}
>>> f1(1, 2, 3, 'a', 'b', x=99)
a = 1 b = 2 c = 3 args = ('a', 'b') kw = {'x': 99}
>>> f2(1, 2, d=99, ext=None)
a = 1 b = 2 c = 0 d = 99 kw = {'ext': None}
```

#### 递归函数

在函数内部，可以调用其他函数。如果一个函数在内部调用自身本身，这个函数就是递归函数。

举个例子，我们来计算阶乘`n! = 1 x 2 x 3 x ... x n`，用函数`fact(n)`表示，可以看出：

*f**a**c**t*(*n*)=*n*!=1×2×3×⋅⋅⋅×(*n*−1)×*n*=(*n*−1)!×*n*=*f**a**c**t*(*n*−1)×*n*

所以，`fact(n)`可以表示为`n x fact(n-1)`，只有n=1时需要特殊处理。

于是，`fact(n)`用递归的方式写出来就是：

```python
def fact(n):
    if n==1:
        return 1
    return n * fact(n - 1)
```

其实递归函数就是在return的地方假设函数所需的运算，返回的自然就是想要的值

使用递归函数需要注意防止栈溢出。在计算机中，函数调用是通过栈（stack）这种数据结构实现的，每当进入一个函数调用，栈就会加一层栈帧，每当函数返回，栈就会减一层栈帧。由于栈的大小不是无限的，所以，递归调用的次数过多，会导致栈溢出。
