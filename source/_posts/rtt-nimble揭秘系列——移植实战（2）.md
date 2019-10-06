---
title: rtt+nimble揭秘系列——移植实战（2）
date: 2019-10-06 01:36:23
tags:
- rtthread
- nimble

categories:
- 一得之见
---

-
<!-- more --> 

前言
===

博主不喜欢从零开始造轮子，但是有“不试一下鬼知道做了什么”强迫症，因此钟爱搬运移植文件，用一些土方法来理清软件之间的关系以及换一个平台怎么办的思路。

“移植”目的：

1. 了解rt-thread的构建系统/设备和驱动
2. 移植nordic的nrfx模块须知
3. 了解nimble的目录结构/host和controller运行机制

目录
===

<!-- TOC -->

- [1. 建立项目框架](#1-建立项目框架)
    - [1.1. 创建bsp文件夹](#11-创建bsp文件夹)
    - [1.2. Kconfig](#12-kconfig)
    - [1.3. SConscript/SConstruct](#13-sconscriptsconstruct)
    - [1.4. rtconfig.py](#14-rtconfigpy)
    - [1.5. 创建模板工程](#15-创建模板工程)
- [2. 移植nrfx](#2-移植nrfx)
- [3. 移植nimble](#3-移植nimble)
    - [3.1. 主机（host）](#31-主机host)
    - [3.2. 控制器（controller）](#32-控制器controller)

<!-- /TOC -->

# 1. 建立项目框架

## 1.1. 创建bsp文件夹

在bsp目录下创建文件夹myboard，即`rt-thread\bsp\myboard`。

## 1.2. Kconfig

将`rt-thread\bsp\stm3210x\Kconfig`搬运到myboard目录下，删除Kconfig文件中不需要部分。

```
config SOC_STM32F1
bool
select ARCH_ARM_CORTEX_M3
default y
source "$BSP_DIR/drivers/Kconfig"
```

在myboard目录下执行menuconfig会根据`rt-thread\bsp\myboard\Kconfig`中的语句生成`rt-thread\bsp\myboard\rtconfig.h`，该头文件包含各种宏定义。

Kconfig可引用其他目录下的Kconfig文件，协助其生成所需的宏定义，如以下例子：


- Kconfig文件的目录路径：

```
 rt-thread
 ├──Kconfig             # root Kconfig 
 ├──src
 │   └──Kconfig         # kernal src Kconfig
 └──bsp         
     └──myboard
           └──Kconfig   # bsp Kconfig
```

- `rt-thread\bsp\myboard\Kconfig`引用`rt-thread\Kconfig`语句：

```
config RTT_DIR
    string
    option env="RTT_ROOT"
    default "../.."

source "$RTT_DIR/Kconfig"  
```

- `rt-thread\Kconfig`引用`rt-thread\src\Kconfig`语句：

```
source "$RTT_DIR/src/Kconfig"
```

- `rt-thread\src\Kconfig`协助生成宏定义内容：

```
config RT_TICK_PER_SECOND
    int "Tick frequency, Hz"
    range 10 1000
    default 100
    help
        System's tick frequency, Hz.
```

- 在myboard目录下执行menuconfig配置后，会在`rt-thread\bsp\myboard\rtconfig.h`生成如下宏定义：

```C
#ifndef RT_CONFIG_H__
#define RT_CONFIG_H__

#define RT_TICK_PER_SECOND 100

#endif
```

- 最后其他`.c`文件可引用该头文件，并使用该宏定义，即

```C
#include <rtconfig.h>

void SystemClock_Config(void)
{
    SysTick_Config(SystemCoreClock / RT_TICK_PER_SECOND);
    NVIC_SetPriority(SysTick_IRQn, 0);
}
```

## 1.3. SConscript/SConstruct

将`rt-thread\bsp\nrf52832\SConscript`和`rt-thread\bsp\nrf52832\SConstruct`搬运到myboard目录下。

## 1.4. rtconfig.py

将`rt-thread\bsp\nrf52832\rtconfig.py`搬运到myboard目录下。

## 1.5. 创建模板工程

> [rt-thread官方NOTE](https://www.rt-thread.org/document/site/programming-manual/scons/scons/)：要生成 MDK 或者 IAR 的工程文件，前提条件是 BSP 目录存在一个工程模版文件，然后 scons 才会根据这份模版文件加入相关的源码，头文件搜索路径，编译参数，链接参数等。而至于这个工程是针对哪颗芯片的，则直接由这份工程模版文件指定。所以大多数情况下，这个模版文件是一份空的工程文件，用于辅助 SCons 生成 project.uvprojx 或者 project.eww。

在调用`scons --target=mdk5`前，需在`rt-thread\bsp\myboard`先提供一个mdk模板工程：

下面简单演示工程创建过程。

![](create_template_mdk.gif)

# 2. 移植nrfx

文件夹路径：nRF5_SDK_15.3.0_59ac345\modules\nrfx

nrfx文件夹包含nordic提供的一系列的外设驱动，让用户无需集成整个标准SDK就能把芯片跑起来，是一个轻量级的驱动库。




# 3. 移植nimble

## 3.1. 主机（host）

## 3.2. 控制器（controller）