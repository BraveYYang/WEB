---
title: Json数据包收发测试
tag: Json
date: 2024-08-01
categories: C/C++
index_img: https://s2.loli.net/2024/07/31/erFMNIURnl5YdSk.jpg
---

# Json收发测试

## Json构建

首先定义一个Json的创建函数，函数的返回类型要是字符串格式，因为json最后的发送都是以字符串进行发送的

首先要设计好Json格式的内容层次，然后再去写代码，我的Json包的层次如下

```
|params
	|mtype：1
	|buf：
		|method：BELT_TEST
		|beltNum：
			|name_1：90
			|name_2：45
		|message:
			|error:0
			|other：aaaa
```

我们需要观察在这个层次里面，哪些是包含多个参数，哪些是只有一个参数的，如果是多个参数的，我们可以把它想成一个小的json包用来装载，如果只是单个参数的，直接进行装载进去就行

**记住json包的顺序定义，观察哪些是多参数的先定义，然后装载单参数，再从里往外装载多参数，最后转换格式进行发送**

根据上面定义的层次定义，首先要使用`json_object_new_object()`函数创建一个params的json结构，用来装载接下来的内容

```
    //创建一个json对象
    json_object *params = json_object_new_object();
    
    //创建json子对象buf
    json_object *buf = json_object_new_object();
    
    //创建json子对象beltNum
    json_object *beltNum = json_object_new_object();
    
    //创建json子对象message
    json_object *message = json_object_new_object();
```

之后观察哪些是单参数，依次装载进去即可，mtype相对于params是单参数，method相对于buf是单参数，name_1和name_2相对于beltNum是单参数，error和other相对于message是单参数，所以对单参数进行装载

这里面需要注意的是，如果装载的是字符，需要用json_object_new_string，如果是整数型，则用json_object_new_int

```
    //在params对象中加入键位mtype，并写入整数值1
    json_object_object_add(params, "mtype", json_object_new_int(1));
    
    //在buf对象中加入键位method，并写入字符格式“BELT_TEST”
    json_object_object_add(buf, "method", json_object_new_string("BELT_TEST"));
    
    //在beltNum对象中加入键位name_1，并写入整数值90
    json_object_object_add(beltNum, name_1, json_object_new_int(90));
    
    //在beltNum对象中加入键位name_2，并写入整数值45
    json_object_object_add(beltNum, name_2, json_object_new_int(45));
    
    //在message对象中加入键位error，并写入整数值0
    //json_object_object_add(message, "error", json_object_new_int(0));
    
    //在message对象中加入键位other，并写入字符格式“aaaaa”
    json_object_object_add(message, "other", json_object_new_string("aaaaa"));
```

之后，我们将多参数从层次最里面的往外进行装载即可

```
    //将belt_name的子对象加入到buf中
    json_object_object_add(buf, "beltNum", beltNum);
    
    //将message的子对象加入到buf中
    json_object_object_add(buf, "message", message);
    
    //将buf的子对象加入到params中
    json_object_object_add(params, "buf", buf);
```

最后，我们已经完成整个json包的构建，但是在json包的收发必须通过字符串格式，而不是通过json包的格式，所以需要对格式进行转换

```
	//将params转换成字符串格式
    const char* cmdJsonStr = json_object_to_json_string_ext(params, JSON_C_TO_STRING_PLAIN|JSON_C_TO_STRING_NOSLASHESCAPE);
    //JSON_C_TO_STRING_PLAIN: 生成紧凑的 JSON 字符串，不包含额外的空格或缩进。这有助于减少输出的大小。
    //JSON_C_TO_STRING_NOSLASHESCAPE: 不对斜杠字符（/）进行转义。在 JSON 标准中，斜杠可以被转义为 \/，但这个标志会使输出保持为 /
    
    //最后的输出格式为
    //cmdJsonStr = {"mtype":1,"buf":{"method":"BELT_TEST","beltNum":{"name_1":90,"name_2":"45"}"message":{"error":0,"other":"aaaaa"}}}
```

最终，我们的代码整体如下

```
static const char* Jsontest(char *name)
{
    //创建一个json对象params
    json_object *params = json_object_new_object();
    
    //创建json子对象buf
    json_object *buf = json_object_new_object();
    
    //创建json子对象beltNum
    json_object *beltNum = json_object_new_object();
    
    //创建json子对象message
    json_object *message = json_object_new_object();

    //在params对象中加入键位mtype，并写入整数值1
    json_object_object_add(params, "mtype", json_object_new_int(1));
    
    //在buf对象中加入键位method，并写入字符格式“BELT_TEST”
    json_object_object_add(buf, "method", json_object_new_string("BELT_TEST"));
    
    //在beltNum对象中加入键位name_1，并写入整数值90
    json_object_object_add(beltNum, name_1, json_object_new_int(90));
    
    //在beltNum对象中加入键位name_2，并写入整数值45
    json_object_object_add(beltNum, name_2, json_object_new_int(45));
    
    //在message对象中加入键位error，并写入整数值0
    //json_object_object_add(message, "error", json_object_new_int(0));
    
    //在message对象中加入键位other，并写入字符格式“aaaaa”
    json_object_object_add(message, "other", json_object_new_string("aaaaa"));
    
    //将belt_name的子对象加入到buf中
    json_object_object_add(buf, "beltNum", beltNum);
    
    //将message的子对象加入到buf中
    json_object_object_add(buf, "message", message);
    
    //将buf的子对象加入到params中
    json_object_object_add(params, "buf", buf);
    
    //将params转换成字符串格式
    const char* cmdJsonStr = json_object_to_json_string_ext(params, JSON_C_TO_STRING_PLAIN|JSON_C_TO_STRING_NOSLASHESCAPE);
    //JSON_C_TO_STRING_PLAIN: 生成紧凑的 JSON 字符串，不包含额外的空格或缩进。这有助于减少输出的大小。
    //JSON_C_TO_STRING_NOSLASHESCAPE: 不对斜杠字符（/）进行转义。在 JSON 标准中，斜杠可以被转义为 \/，但这个标志会使输出保持为 /
    
    //输出日志作为测试
    CrLogI("cmdJsonStr = %s\n",cmdJsonStr);

    //返回cmdJsonStr字符
    return cmdJsonStr;
}
```

## Json的解析

解析json数据包的时候要找关键信息，因为数据包的有些数据并不会在该函数进行解析，可能会在别的函数去判断，所以我们首先要找准我们要哪些数据

首先我需要获取这个数据包里面的mtype作为标志位，来判断是否接收到了这个数据包，然后去解析buf中的message的error，看看是否包含了所需的数据，然后进行判断，罗列我们所要的数据

```
|params
	|mtype：1
	|buf：
		|message:
			|error:0
```

根据上面所列，我们开始对json数据进行解析

首先使用对整个json数据包进行解析，因为发送过来的json包如果不解析的话，无法使用接下来的json语句抓取其中的关键值

```
//首先使用上面的发送函数，来获取一个json包
const char* str_2 = Jsontest(name);

//定义一个json格式的值用来储存json包
json_object *recv = json_tokener_parse(str_2);
```

之后我们获得了整个json的数据，我们只需要对里面的内容进行查找即可，解包只需要一次，之后只需要不断查找判断即可

我们获取的数据需要对他进行验证判断，首先需要判断是否包含该键，之后验证其格式是否正确，其携带的值是什么

```
//定义一个值来储存从recv中查找"mtype"键所携带的值
json_object *mtype = json_object_object_get(recv,"mtype");

//判断是否查找到该键，该键的格式是否为整数型，该键的值是否为1，如果有一个不符合，说明解包错误
if(mytpe != NULL && json_object_is_type(mtype, json_type_int) && json_object_get_int(mytpe) == 1)
```

刚刚的mytpe是单参数的，多参数的也类似，不过也需要一层一层的解开，而不是直接解到最底层

```
//定义一个值来储存从recv中查找"buf"键所携带的值
json_object *buf = json_object_object_get(recv,"buf");

//判断是否查找到该键，该键的格式是否为多参数型，如果有一个不符合，说明解包错误
if(buf != NULL && json_object_is_type(buf, json_type_object))
{
	//完成第一层解包后，需要进行第二层解包，定义一个值来储存从buf中查找"message"键所携带的值
	json_object *message = json_object_object_get(buf,"message");
	
	//判断是否查找到该键，该键的格式是否为多参数型，如果有一个不符合，说明解包错误
	if(message != NULL && json_object_is_type(message, json_type_object))
	{
		//继续解包，定义一个值来储存从message中查找"error"键所携带的值
		json_object *error = json_object_object_get(message,"error");
		if(error == NULL && json_object_get_int(error) == 0)
		{
			CrLogI("belt tension no match\n");//有error键值，说明不合格
		}
	}
	json_object_put(buf);//释放json空间
}
```

最后，我们就完成了整个json的解读，获取关键值进行判断，其中如果多参数的键，我们不能让其输出值，而是需要判断其格式是否符合多参数的格式，如果是单参数的键，可以利用`json_object_get_int(error) == 0`输出值并进行比较。
