---
title: Python-类和实例补充
tag: Python
date: 2024-08-13
categories: Python
index_img: https://s2.loli.net/2024/08/02/CqTcyiJGYHR8k3L.jpg
---

# Python-类和实例补充

## 类和实例的补充

其实在学习类的过程中还是非常的懵逼，主要感觉讲的并不是很好，所以特地再重新学习补充一下

类其实就是一大类，就比如学生这就是一个类，而中学生，大学生，却是这个类里面的一个子类，但是他们也具备学生的特性，只不过因为阶段不同，所以存在差异性，因此我们可以将假设的类定义一下

```
class Student(Object):
```

其中，这个Object类似什么呢，学生是不是人类，那肯定是人类，所以学生这个类其实也包含在一个大类里面，这个大类就是Object，也就是程序中最高的大类

那这个类定义好之后，我们要开始对他进行初始化，初始化是什么呢，初始化其实就是初始化学生的一些基本信息，比如姓名、年龄、性别，这些是在输入到这个数据结构中，必须要提前标识好的，如果没有标识好，就会出现信息不足，导致没办法区别这个人，因此我们就可以进行函数初始化

```
class Student(object):
    def __init__(self, name, age, sex):
        self.name = name
        self.age = age
        self.sex = sex 
```

这样的话，这个调用这个类，他就必须要有这三个参数，否则就会出现报错，那self是什么呢，我觉得他是代表这个类的数据结构，如果你将数据前面加一个self的定义，那么这个数据就存到了这个类里面，他就可以在这个类里面随便使用了

之后呢，我们想要对中学生、大学生进行区分开，分别填入他们相应的数据，比如中学生要面对高考痛苦，大学生要面对就业痛苦，因此我们就可以根据他们面对不同的问题进行在这个类里面进行区分

我们再来分析一下中学生，他们的高考痛苦来源于语数英...我们这边列举两门作为示例，比如他们的数学和英语，他们看重成绩，那是不是在评价这个学生的时候，要看看这个学生的成绩，所以这两门的成绩成为了他们的必选参数，但是大学生不需要这个参数啊，那我们就在中学生的这个实例里面定义一个传参的初始化，其实也就是把中学生的数据塞到self数据中，虽然大学生也能用，但是并不是必选参数

```
    def Middle_Exam_pain(self, English, Math):
        self.English = English
        self.Math = Math
        if self.English >=90 and self.Math >= 90:
            print("%s英语考%d、数学考%d，一点也不痛苦" % (self.name, self.English, self.Math))
        elif self.English >=80 and self.English < 90 and self.Math >= 80 and self.Math < 90:
            print("%s英语考%d、数学考%d，有点痛苦" % (self.name, self.English, self.Math))
        else:
            print("%s英语考%d、数学考%d，超级痛苦" % (self.name, self.English, self.Math))
```

我们就可以将英语和数学的成绩单独赋值出来，只对中学生进行调用，这样中学生就可以自己进行判断了

那大学生的痛苦来自哪里呢，大学生的痛苦有来自考研考上和没考上，找工作找没找到，以及他们的月薪是多少，那我们就可以类似的，再创建大学生的实例来表达

```
    def College_pain(self, Graduate=None, School=None, Employment=None, wage=None):
        self.Graduate = Graduate
        self.School = School
        self.Employment = Employment
        self.wage = wage
        if self.Graduate is not None and self.School is not None:
            if self.Graduate > 400:
                print("%s考研考了%d分，考上了%s，一点也不痛苦" % (self.name, self.Graduate, self.School))
            else:
                print("%s考研考了%d分，考上了%s，有点痛苦" % (self.name, self.Graduate, self.School))
        elif self.Graduate is not None and self.School == None:
            print("%s考研考了%d分，没考上学习，非常痛苦" % (self.name, self.Graduate))
        elif self.Employment is not None and self.Graduate == None and self.School == None:
            if self.wage >10000:
                print("%s找到了工作，在%s上班，工资是%s，一点也不痛苦" % (self.name, self.Employment, self.wage))
            else:
                print("%s找到了工作，在%s上班，工资是%s，有点痛苦" % (self.name, self.Employment, self.wage))
        elif self.Employment == None and self.wage == None and self.Graduate == None and self.School == None:
            print("%s没找到工作，非常痛苦" % self.name)
```

首先对要传入的数据进行初始化，这样我们就可以对其传参了，然后先判断是考研还是找工作，再去判断相对应考研或者找工作的类别，其实这两个凑一起是比较混乱的，可以考虑新建一个实例，如果是考研就调用考研的，如果是找工作的就调用找工作的

完成类的初始化和实例设定后，我们可以来验证我们的代码是什么样子的，有没有bug之类的，首先，需要将实例复制给一个叫lihua的学生，当然别忘记我们的init里面要强制输入的参数

```
lihua = Student("lihua", 18, "boy")
```

然后我们就可以在传入lihua的中学生参数，比如，他英语考了90分，数学考了99分，将数据传入这个函数

```
lihua.Middle_Exam_pain(90,99)
lihua.Middle_Exam_pain(89,87)
lihua.Middle_Exam_pain(77,66)

#output
lihua英语考90、数学考99，一点也不痛苦
lihua英语考89、数学考87，有点痛苦
lihua英语考77、数学考66，超级痛苦
```

验证大学生，假设一个大学生kangkang，先初始化固定参数

```
kangkang = Student("kangkang", 22, "boy")
```

之后我们可以传入他的一些内容，比如他是考研的，或者是找工作的

```
kangkang = Student("kangkang", 22, "boy")
kangkang.College_pain(410,'北京大学')
kangkang.College_pain(250,'野鸡大学')
kangkang.College_pain(200)
kangkang.College_pain(None,None,"华为", 20000)
kangkang.College_pain(None,None,"美团外卖", 4000)
kangkang.College_pain(None,None,None, None)

#output
kangkang考研考了410分，考上了北京大学，一点也不痛苦
kangkang考研考了250分，考上了野鸡大学，有点痛苦
kangkang考研考了200分，没考上学习，非常痛苦
kangkang找到了工作，在华为上班，工资是20000，一点也不痛苦
kangkang找到了工作，在美团外卖上班，工资是4000，有点痛苦
kangkang没找到工作，非常痛苦
```

## 整个程序完整代码

```
class Student(object):
    def __init__(self, name, age, sex):
        self.name = name
        self.age = age
        self.sex = sex
    def Middle_Exam_pain(self, English, Math):
        self.English = English
        self.Math = Math
        if self.English >=90 and self.Math >= 90:
            print("%s英语考%d、数学考%d，一点也不痛苦" % (self.name, self.English, self.Math))
        elif self.English >=80 and self.English < 90 and self.Math >= 80 and self.Math < 90:
            print("%s英语考%d、数学考%d，有点痛苦" % (self.name, self.English, self.Math))
        else:
            print("%s英语考%d、数学考%d，超级痛苦" % (self.name, self.English, self.Math))

    def College_pain(self, Graduate=None, School=None, Employment=None, wage=None):
        self.Graduate = Graduate
        self.School = School
        self.Employment = Employment
        self.wage = wage
        if self.Graduate is not None and self.School is not None:
            if self.Graduate > 400:
                print("%s考研考了%d分，考上了%s，一点也不痛苦" % (self.name, self.Graduate, self.School))
            else:
                print("%s考研考了%d分，考上了%s，有点痛苦" % (self.name, self.Graduate, self.School))
        elif self.Graduate is not None and self.School == None:
            print("%s考研考了%d分，没考上学习，非常痛苦" % (self.name, self.Graduate))
        elif self.Employment is not None and self.Graduate == None and self.School == None:
            if self.wage >10000:
                print("%s找到了工作，在%s上班，工资是%s，一点也不痛苦" % (self.name, self.Employment, self.wage))
            else:
                print("%s找到了工作，在%s上班，工资是%s，有点痛苦" % (self.name, self.Employment, self.wage))
        elif self.Employment == None and self.wage == None and self.Graduate == None and self.School == None:
            print("%s没找到工作，非常痛苦" % self.name)

#output
lihua = Student("lihua", 16, "boy")
lihua.Middle_Exam_pain(90,99)
lihua.Middle_Exam_pain(89,87)
lihua.Middle_Exam_pain(77,66)
kangkang = Student("kangkang", 22, "boy")
kangkang.College_pain(410,'北京大学')
kangkang.College_pain(250,'野鸡大学')
kangkang.College_pain(200)
kangkang.College_pain(None,None,"华为", 20000)
kangkang.College_pain(None,None,"美团外卖", 4000)
kangkang.College_pain(None,None,None, None)
```

## 继承

继承的话，其实简单来说就是可以用父类里面的实例和参数，当然如果想用参数，需要在init里面去继承父类的init才可以传入参数，如果实例没有参数的话，可以直接引用

```
class Student(object):
    def __init__(self, name, age, sex):
        self.name = name
        self.age = age
        self.sex = sex
	def father(self):
        print("继承了父类")
        
class Primary(Student):
    def __init__(self):
        pass
        
jack = Primary()
jack.father()

#output
继承了父类
```

当然如果里面有参数，那就要在init里面进行引用

```
class Student(object):
    def __init__(self, name, age, sex):
        self.name = name
        self.age = age
        self.sex = sex
    def Middle_Exam_pain(self, English, Math):
        self.English = English
        self.Math = Math
        if self.English >=90 and self.Math >= 90:
            print("%s英语考%d、数学考%d，一点也不痛苦" % (self.name, self.English, self.Math))
        elif self.English >=80 and self.English < 90 and self.Math >= 80 and self.Math < 90:
            print("%s英语考%d、数学考%d，有点痛苦" % (self.name, self.English, self.Math))
        else:
            print("%s英语考%d、数学考%d，超级痛苦" % (self.name, self.English, self.Math))

class Primary(Student):
    def __init__(self, name, age, sex):
        super().__init__(name, age, sex)

jack = Primary("jack",18,"boy")
jack.Middle_Exam_pain(90,100)

#output
jack英语考90、数学考100，一点也不痛苦
```

如果是多重继承的话，其实有个比较简单的套娃方式

```
class Student(object):
    def __init__(self, name, age, sex):
        self.name = name
        self.age = age
        self.sex = sex 
    def father(self):
        print("继承了Student")
        
class Teacher(Student):
    def __init__(self, name, age, sex):
        super().__init__(name, age, sex)
    def talk(self):
        print("继承了Teacher")
        
class Primary(Teacher):
    def __init__(self, name, age, sex):
        super().__init__(name, age, sex)
```

然后，我们就可以顺利的使用Student和Teacher的所有内容了

```
jack = Primary("jack",18,"boy")
jack.father()
jack.talk()
jack.Middle_Exam_pain(90,100)

#output
继承了Student
继承了Teacher
jack英语考90、数学考100，一点也不痛苦
```

