---
title: 3D打印机运行逻辑
tag: 3D打印机
date: 2024-07-26
categories: 3D打印机
index_img: https://s2.loli.net/2024/07/31/hRrLCWXmb3HwpID.png
---

# 3D打印机运行逻辑

### 打印机初始化

1.获取机器，连接网络，并且在设置里面打开root权限，获得root的账号和密码

```
root  //账号
root  //密码
```

2.需要获取机器的IP地址，然后用mobaxtem的ssh进行连接，连接时输入IP地址和账号

```
172.xx.xxx.xx  //IP地址
```

3.拷贝所需的文件夹进行安装

```
cp /tmp/udisk/sda1/fluidd/fluidd.sh /usr/data  //拷贝脚本文件
cp /tmp/udisk/sda1/fluidd/fluidd.tar /usr/data  //拷贝配置文件
	
cd /usr/data  //进入该文件夹下
	
./fluidd.sh install  //执行安装指令
```

4.等待结束，即可登录fluidd网站

```
172.xx.xxx.xx:4408  //fluidd的网址
```

### 打印机操作

SET_VELOCITY_LIMIT 改变打印机配置文件中指定的速度限制。

```
SET_VELOCITY_LIMIT [VELOCITY=<value>] [ACCEL=<value>] [MINIMUM_CRUISE_RATIO=<value>] [SQUARE_CORNER_VELOCITY=<value>]

[VELOCITY=<value>]  //速度
[ACCEL=<value>]  //加速度
[MINIMUM_CRUISE_RATIO=<value>]  //最小巡航速度
[SQUARE_CORNER_VELOCITY=<value>]  //角速度
[ACCEL_TO_DECEL]  //加速减速
```

3D打印机Z轴正值是向下，负值是向上

```
G1 Z1  //K1-MAX向下位移1mm
G1 Z-.5  //K1-MAX向上位移到-0.5mm
```

3D打印机X轴正值是向右，负值是向左

```
G1 X1  //K1-MAX向右位移1mm
G1 X-.5  //K1-MAX向左位移到-0.5mm
```

3D打印机X轴正值是向后，负值是向前

```
G1 X1  //K1-MAX向后位移1mm
G1 X-.5  //K1-MAX向前位移到-0.5mm
```

EXCLUDE_OBJECT_START 表示当前层上一个对象的gcode开始

EXCLUDE_OBJECT_END 表示对象在该层的代码的结束

```
EXCLUDE_OBJECT_START NAME=对象名称
EXCLUDE_OBJECT_END [NAME=对象名称]
```

EXCLUDE_OBJECT_DEFINE

```
EXCLUDE_OBJECT_DEFINE [NAME =对象名称[中心= X, Y][多边形= [(X, Y),……[reset =1] [json =1]  //提供文件中一个对象的摘要。
//如果没有提供参数，这将列出Klipper已知的已定义对象。返回字符串列表，除非给出了JSON参数，否则它将以JSON格式返回对象详细信息。

<NAME>  //当包含NAME参数时，将定义要排除的对象。

<CENTER>  //对象的 X，Y 坐标。

<POLYGON>  //提供对象轮廓的 X,Y 坐标数组

//当提供RESET参数时，将清除所有已定义的对象，并重置[exclude_object]模块。
```

### G-code 打印流程

```
//这个干嘛没搞明白
EXCLUDE_OBJECT_DEFINE  NAME=Square_columns_Z_axis.stl_id_0_copy_0 CENTER=163.166,143.573 POLYGON=[[58.702,39.1091],[267.631,39.1091],[267.631,248.038],[58.702,248.038],[58.702,39.1091]]

//报告当前进度给控制器，P0是进度为0%，R1210是当前设定的剩余时间
M73 P0 R1210

//关闭P0风扇，没有P参数，默认P0，S代表风扇的模拟量为0，范围是0~255，
M106 S0

//关闭P2风扇
M106 P2 S0

//关闭热床温度，不进行等待
M140 S0

//关闭喷头温度，不进行等待，继续执行
M104 S0 

//执行宏定义，在gcode_macro.cfg配置文件中有宏定义函数，作用为开始打印，对打印机进行自动，调节热床和喷头温度
START_PRINT EXTRUDER_TEMP=220 BED_TEMP=50

//设定机箱温度为35摄氏度
M141 S35

//将打印机三轴设为绝对坐标系
G90

//设置移动单位为毫米
G21

//设置挤出头为绝对坐标系
M83

//关闭P0风扇
M106 S0

//关闭P2风扇
M106 P2 S0

//设置挤出头当前位置为零点
G92 E0

//使用G1直线移动方式，将挤出头回抽0.6mm，速度为2400mm/min
G1 E-.6 F2400

//设置加速度为1000mm/min，加减速速度为1000mm/min
SET_VELOCITY_LIMIT ACCEL=1000 ACCEL_TO_DECEL=1000

//设置角加速度为20rad/min
SET_VELOCITY_LIMIT SQUARE_CORNER_VELOCITY=20

//表示当前层开始打印
EXCLUDE_OBJECT_START NAME=Square_columns_Z_axis.stl_id_0_copy_0

//开始打印，G1直线位移到坐标为(49.789，32.953)位置，速度为30000mm/min
G1 X49.789 Y32.952 F30000

//Z轴向下移动0.4mm
G1 Z.4

//开始重复性打印操作
G1 X50.408 Y32.135 E.037
G1 X51.103 Y31.381 E.03702
G1 X51.866 Y30.696 E.03702
G1 X52.691 Y30.087 E.03702
G1 X53.57 Y29.56 E.03702
G1 X54.494 Y29.119 E.03697
G1 X55.682 Y28.701 E.04545
G1 X56.731 Y28.45 E.03897
G1 X57.747 Y28.311 E.03702
G1 X58.707 Y28.268 E.0347
G1 X267.625 Y28.268 E7.5432
G1 X268.655 Y28.317 E.03721
G1 X269.669 Y28.463 E.03702
G1 X270.665 Y28.706 E.03702
G1 X271.632 Y29.041 E.03696
G1 X272.832 Y29.612 E.04796
G1 X273.788 Y30.196 E.04045
...

//打印进度和时长标记位
M73 P0 R1208

//表示上一层已经打印结束
EXCLUDE_OBJECT_END NAME=Square_columns_Z_axis.stl_id_0_copy_0

//表示当前层开始打印
EXCLUDE_OBJECT_START NAME=Square_columns_Z_axis.stl_id_0_copy_0

//关闭P0、P2风扇
M106 S0
M106 P2 S0

//结束打印，在gcode_macro.cfg配置文件中有宏定义函数，表示结束打印
END_PRINT

//设置机箱温度为0
M141 S0

//上传打印进度
M73 P100 R0
```

