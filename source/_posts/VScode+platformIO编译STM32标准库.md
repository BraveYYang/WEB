---
title: VScode+platformIO编译STM32标准库
tag: VScode
date: 2024-09-04
categories: STM32
index_img: https://s2.loli.net/2024/08/15/GtV85YzoJU9NnjI.jpg
---

# VScode+platformIO编译STM32标准库

## 参考博主

https://juejin.cn/post/7393313322210426915

https://blog.csdn.net/maomaochong666/article/details/129240827

## 配置主要有四个步骤：

#### 1.创建STM32项目，创建的芯片后缀带Generic的即可，选择CMSIS框架

![image.png](https://s2.loli.net/2024/09/04/xN5agDtl6RfTYI9.png)

#### 2.拷录文件放入STM32项目中

以野火为模板，将library文件user文件拷入到项目当中的src文件夹下，不要分开使用，直接全部拷入，方便使用，删除start启动文件和system_stm32f10x.c文件，因为在初始化的时候已经自动生成

<img src="https://s2.loli.net/2024/09/04/ItUnakMPEJAlwrW.png" alt="image.png" style="zoom:50%;" />

#### 3.修改platformIO.ini文件

```
//烧录和调试配置
upload_protocol = cmsis-dap
debug_tool = cmsis-dap

// 添加头文件链接
build_flags = 
    -Wl,-u,_printf_float
    -Wl,-Map,output.map
    -O0
    -Isrc/Libraries
    -Isrc/Libraries/CMSIS
    -Isrc/Libraries/FWlib/inc
    -Isrc/User
    -Isrc/User/DMA
    -Isrc/User/EXTI
    -Isrc/User/I2C
    -Isrc/User/LED
    -Isrc/User/RCC
    -Isrc/User/SPI
    -Isrc/User/Systick
    -Isrc/User/USART

    -D STM32F10X_HD
    -D USE_STDPERIPH_DRIVER

// 定义串口通信，使用motion进行串口调试
monitor_speed = 115200
monitor_echo = yes
monitor_rts = 0
monitor_dtr = 0

// 将烧录文件转为hex格式
extra_scripts = export_hex.py
```

#### 4.修改core_cm3.c文件内容

修改core_cm3.c里面的内容（736、753行）

![image.png](https://s2.loli.net/2024/09/04/AiZRHY1Kd4wjWnb.png)

#### 5.创建烧录文件hex转换脚本

export_hex.py

```
Import("env")
 
# # Custom HEX from ELF
 
env.AddPostAction(
 
    "$BUILD_DIR/${PROGNAME}.elf",
 
    env.VerboseAction(" ".join([
 
        "$OBJCOPY", "-O", "ihex", "-R", ".eeprom",
 
        '"$BUILD_DIR/${PROGNAME}.elf"', '"$BUILD_DIR/${PROGNAME}.hex"'  # 加个单引号
 
    ]), "Building $BUILD_DIR/${PROGNAME}.hex")
 
)
```

然后就可以编译并且烧录到单片机中，烧录过程查看博主内容
