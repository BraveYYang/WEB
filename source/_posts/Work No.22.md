---
title: Python学习-2
tag: Python
date: 2024-08-12
categories: Python
index_img: https://s2.loli.net/2024/08/02/1wdIJmxaylcsK9j.jpg
---

# Python学习-2

## 感谢博主：

[数据类型和变量 - Python教程 - 廖雪峰的官方网站 (liaoxuefeng.com)](https://liaoxuefeng.com/books/python/basic/data-types/index.html)

## 切片

假设一个列表或者元组有如下的元素

```
>>> qiepian = [1,2,3,4,5,6,7,8,9]    
>>> qiepian
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

现在我想访问第五个元素

```
>>> qiepian[4]
5
```

我想访问第五个到第七个元素

```
>>> [qiepian[5],qiepian[6],qiepian[7]]
[6, 7, 8]
```

有点过于繁琐，所以python有一个切片的方式去访问所需的元素，比如我想访问第五个到第七个元素

```
>>> qiepian[5:8] 
[6, 7, 8]
```

从索引`5`开始取，直到索引`8`为止，但不包括索引`8`。即索引`5`，`6`，`7`，正好是3个元素。

切片操作十分有用。我们先创建一个0-99的数列：

```bash
>>> L = list(range(100))
>>> L
[0, 1, 2, 3, ..., 99]
```

可以通过切片轻松取出某一段数列。比如前10个数：

```bash
>>> L[:10]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

后10个数：

```bash
>>> L[-10:]
[90, 91, 92, 93, 94, 95, 96, 97, 98, 99]
```

前11-20个数：

```bash
>>> L[10:20]
[10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
```

前10个数，每两个取一个：

```bash
>>> L[:10:2]
[0, 2, 4, 6, 8]
```

所有数，每5个取一个：

```bash
>>> L[::5]
[0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95]
```

甚至什么都不写，只写`[:]`就可以原样复制一个list：

```bash
>>> L[:]
[0, 1, 2, 3, ..., 99]
```

tuple也是一种list，唯一区别是tuple不可变。因此，tuple也可以用切片操作，只是操作的结果仍是tuple：

```bash
>>> (0, 1, 2, 3, 4, 5)[:3]
(0, 1, 2)
```

字符串`'xxx'`也可以看成是一种list，每个元素就是一个字符。因此，字符串也可以用切片操作，只是操作结果仍是字符串：

```bash
>>> 'ABCDEFG'[:3]
'ABC'
>>> 'ABCDEFG'[::2]
'ACEG'
```

## 迭代

如果给定一个`list`或`tuple`，我们可以通过`for`循环来遍历这个`list`或`tuple`，这种遍历我们称为迭代（Iteration）。

在Python中，迭代是通过`for ... in`来完成的

列表和元组的迭代比较简单，直接定义一个值，然后把列表和元组塞到for循环里面打印遍历就行

```
>>> for list in classmates:
...     print(list)
...
kangkang
jack
xiaoming
zhengzheng
>>> classmates
['kangkang', 'jack', 'xiaoming', 'zhengzheng']
```

但是dist字典的存储不想列表和元组那样有顺序，他是键和值一起结合在一起的，所以直接使用迭代的话只能迭代出键的值， 如果想要迭代出值，需要如下操作：

```
>>> for key in dict:
...     print(key)
...
lihua
kangkang
liuyang
>>> dict
{'lihua': 94, 'kangkang': 55, 'liuyang': 94}


>>> for value in dict.values(): 
...     print(value) 
...
94
55
94

>>> for dict, value in dict.items(): 
...     print(dict,value) 
...
lihua 94
kangkang 55
liuyang 94

```

## 类和实例

面向对象最重要的概念就是类（Class）和实例（Instance），必须牢记类是抽象的模板，比如Student类，而实例是根据类创建出来的一个个具体的“对象”，每个对象都拥有相同的方法，但各自的数据可能不同。

`class`后面紧接着是类名，即`Student`，类名通常是大写开头的单词，紧接着是`(object)`，表示该类是从哪个类继承下来的，通常，如果没有合适的继承类，就使用`object`类，这是所有类最终都会继承的类。

python的class（类）相当于一个多个函数组成的家族，如果在这个Myclass大家族里有一个人叫f，假如这个f具有print天气的作用，那么如果有一天我需要这个f来print一下今天的天气，那么我必须叫他的全名MyClass.f才可以让他给我print，即在调用他的时候需要带上他的家族名称+他的名称。

```
#Myclass家族，但是这个家族只有一个人f
class MyClass:   
  """一个简单的类实例"""    
  i = 12345    
  def f(self):        
    return 'hello world'
# 实例化类
x = MyClass() 
# 访问类的属性和方法
print("MyClass 类的属性 i 为：", x.i) #家族x + 物品名i
print("MyClass 类的方法 f 输出为：", x.f()) #家族x + 人名f
```

假如init()也是人，但是他是家族和外界联络员，当外界的人想调用自己家族的人，就必须要先告诉他，所以只要家族的人被调用，那么init()就会被先执行，然后由他去告诉那个被调用的人，执行被调用的。

```
class Complex:
    def __init__(self, realpart, imagpart): #必须要有一个self参数，
        self.r = realpart
        self.i = imagpart
x = Complex(3.0, -4.5)
print(x.r, x.i)   # 输出结果：3.0 -4.5
```

在类的内部，使用 def 关键字来定义一个方法，与一般函数定义不同，类方法**必须**包含参数`self`, 且为第一个参数，`self`代表的是类的实例。

self：类的方法与普通的函数只有一个特别的区别——必须有一个额外的第一个参数名称, 按照惯例它的名称是self。
类的实例：是将类应用在实例场景之中，比如有个类里的函数是f，假如这个f具有print某一时刻的天气状况的能力，那么如果我需要这个f来print一下今天12点的天气，那么让他打印今天12点的天气这个动作，就是类的实例化，让类中的函数具有的能力变成真实的动作。

```
#类定义
class people:
    #定义基本属性
    name = ''
    age = 0
    #定义私有属性,私有属性在类外部无法直接进行访问
    #定义构造方法
    def __init__(self,n,a):
        self.name = n
        self.age = a
    def speak(self):
        print("%s 说: 我 %d 岁。" %(self.name,self.age))

# 实例化类
p = people('Python',10,30)
p.speak()
```

#### 继承

假如有两个家族，有一个家族A开始没落了，另一个新兴的家族B想继承A家族的物资和佣人，那么就可以通过如下的方式实现继承，在这里，家族A即是父类，家族B是子类。在用法上，如果B家族可以任意使用A家族的物品和佣人。

```
class [子类]([父类]):
```

python还支持**多继承**，即可以继承多个父类。继承方式和单继承方式一样，方式如下：

```
class [子类]([父类]1, [父类]2, [父类]3):
```

```
#类定义
class people:
    #定义基本属性
    name = ''
    age = 0
    #定义私有属性,私有属性在类外部无法直接进行访问
    __weight = 0
    #定义构造方法
    def __init__(self,n,a,w):
        self.name = n
        self.age = a
        self.__weight = w
    def speak(self):
        print("%s 说: 我 %d 岁。" %(self.name,self.age))
 
#单继承示例
class student(people):
    grade = ''
    def __init__(self,n,a,w,g):
        #调用父类的构函
        people.__init__(self,n,a,w)
        self.grade = g
    #覆写父类的方法
    def speak(self):
        print("%s 说: 我 %d 岁了，我在读 %d 年级"%(self.name,self.age,self.grade))
 
#另一个类，多继承之前的准备
class speaker():
    topic = ''
    name = ''
    def __init__(self,n,t):
        self.name = n
        self.topic = t
    def speak(self):
        print("我叫 %s，我是一个演说家，我演讲的主题是 %s"%(self.name,self.topic))
 
#多继承
class sample(speaker,student):
    a =''
    def __init__(self,n,a,w,g,t):
        student.__init__(self,n,a,w,g)
        speaker.__init__(self,n,t)
 
test = sample("Tim",25,80,4,"Python")
test.speak()   #方法名同，默认调用的是在括号中参数位置排前父类的方法
```

在类里面，还可以动态的添加参数

```
class Student(object):
    def __init__(self, name):
        self.name = name

s = Student('Bob')
s.score = 90
```

但是这种动态的添加参数并不能改变类的格式，只能储存在类的该实例当中，这个数据只为这个实例而存在，并不会改变类的结构

还可以给这个实例绑定一个方法

```
>>> def set_age(self, age): # 定义一个函数作为实例方法
...     self.age = age
...
>>> from types import MethodType
>>> s.set_age = MethodType(set_age, s) # 给实例绑定一个方法
>>> s.set_age(25) # 调用实例方法
>>> s.age # 测试结果
25
```

#### 限制实例的属性

为了达到限制的目的，Python允许在定义`class`的时候，定义一个特殊的`__slots__`变量，来限制该`class`实例能添加的属性：

```
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称
```

#### 数据限制

如果要让内部属性不被外部访问，可以把属性的名称前加上两个下划线`__`，在Python中，实例的变量名如果以`__`开头，就变成了一个私有变量（private），只有内部可以访问，外部不能访问

```
class Student(object):
    def __init__(self, name, score):
        self.__name = name
        self.__score = score

    def print_score(self):
        print('%s: %s' % (self.__name, self.__score))
```

改完后，对于外部代码来说，没什么变动，但是已经无法从外部访问`实例变量.__name`和`实例变量.__score`了，这样就确保了外部代码不能随意修改对象内部的状态，这样通过访问限制的保护，代码更加健壮。

如果外部想要获取内部变量，可以定义两个实例，用于将内部变量引出，从而达到访问的目的

```
class Student(object):
    ...

    def get_name(self):
        return self.__name

    def get_score(self):
        return self.__score
```

#### type()判断对象类型

基本类型都可以用`type()`判断：

```
>>> type(123)
<class 'int'>
>>> type('str')
<class 'str'>
>>> type(None)
<type(None) 'NoneType'>
```

如果一个变量指向函数或者类，也可以用`type()`判断：

```plain
>>> type(abs)
<class 'builtin_function_or_method'>
>>> type(a)
<class '__main__.Animal'>
```
