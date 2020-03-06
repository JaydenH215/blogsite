---
title: 浅谈cortexm调试方法
date: 2019-11-07 13:43:53
tags:
- cortex-m
- 内核
- 调试
- debug

categories:
- 术业专攻

---

当产品出现软件bug，出于各种原因，可能拿不到源码，这时候我们可以让客户提供编译生成的axf和map文件，配合jlink进行软件bug定位。
<!-- more --> 

前言
===

下文内容针对cortex-m内核，欢迎斧正，持续更新。

目录
===

<!-- TOC -->

- [1. 工具说明](#1-工具说明)
    - [1.1. JLINK](#11-jlink)
    - [1.2. MAP文件](#12-map文件)
    - [1.3. AXF文件](#13-axf文件)
- [2. 参考资料](#2-参考资料)

<!-- /TOC -->

# 1. 工具说明

## 1.1. JLINK

- JLink.exe（J-Link Commander）：
    - 获取cpu环境，推断当前状态
    - 通过r14判断当前是在handler mode还是thread mode
    - 通过pc判断当前是锁死保护还是正常跑飞（锁死保护为0xFFFFFFFE)
    - 通过sp判断栈指针位置，往前推跑飞时的pc
    - 有具体fault标志位的内核型号看一下标志位

如下图cortex-m的例子
![](jlink.png)

- J-Mem：
    - 获取ram内容和外设寄存器
    - 判断ram是否正确加载
    - 判断寄存器配置是否正确

如下图gpio翻转的例子。
![](gpio翻转.gif)

- J-Flash（其它专用flash dump工具）：
    - 获取flash内容
    - 判断flash是否烧录正确位置
    - 判断是否存在误操作擦写

## 1.2. MAP文件

- Image Symbol Table：
    - 判断是否有爆堆栈风险
    - 判断使用到的sram是否开启retention

![](map_image_table.png)

- Memory Map of the image：
    - 加载地址：可与flash内容对比判断是否烧录正常
    - 执行地址：提供变量地址给J-Mem

![](map_memory.png)

- Image component sizes：
    - Code (inc. data)
    - RO Data    
    - RW Data    
    - ZI Data      
    - Debug

![](map_image component_sizes.png)

## 1.3. AXF文件

- 反汇编：
    -  arm-none-eabi-objdump（gcc工具集）：

        arm-none-eabi-objdump -S -d xxx.axf > xxx.txt

    - fromelf（keil工具集）：
 
        fromelf -c xxx.axf -o xxx.txt

如下图地址0x1fff6258函数的反汇编：
![](反汇编.png)

> NOTE：如果保证编译工程为英文路径，编译器会添加源码路径等信息进axf文件，使得生成的txt文档包含源码。`arm-none-eabi-objdump -h`可以用来查看是否包含调试信息，如果没有需要在编译时候加入`-g`选项。

- section内容：
    -  arm-none-eabi-objdump（gcc工具集）：

        arm-none-eabi-objdump -D xxx.axf > xxx.txt

![](section.png)

# 2. 参考资料
- 《UM08001_JLink.pdf》
- [objdump(Linux)反汇编命令使用指南](https://blog.csdn.net/wwchao2012/article/details/79980514)
- 《DUI0377G_02_mdk_armlink_user_guide.pdf》
- [Linux：objdump命令解析](https://blog.csdn.net/q2519008/article/details/82349869)

