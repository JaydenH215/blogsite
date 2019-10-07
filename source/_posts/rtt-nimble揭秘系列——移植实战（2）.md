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

移植目标：开发板通过串口打印hello world。

目录
===

<!-- TOC -->

- [1. 建立项目框架](#1-建立项目框架)
    - [1.1. 认识nrfx](#11-认识nrfx)
    - [1.2. rt-thread构建环境](#12-rt-thread构建环境)
        - [1.2.1. 创建bsp文件夹](#121-创建bsp文件夹)
        - [1.2.2. 添加bsp的模板工程](#122-添加bsp的模板工程)
        - [1.2.3. 添加bsp的Kconfig](#123-添加bsp的kconfig)
        - [1.2.4. 添加bsp的rtconfig.py](#124-添加bsp的rtconfigpy)
        - [1.2.5. 添加bsp的SConscript/SConstruct](#125-添加bsp的sconscriptsconstruct)
        - [1.2.6. 添加nrfx](#126-添加nrfx)
        - [1.2.7. 添加设备驱动实现](#127-添加设备驱动实现)
        - [1.2.8. 添加应用层代码](#128-添加应用层代码)
    - [1.3. 验证环境构建结果](#13-验证环境构建结果)
- [2. 移植nimble](#2-移植nimble)
    - [2.1. 主机（host）](#21-主机host)
    - [2.2. 控制器（controller）](#22-控制器controller)

<!-- /TOC -->

# 1. 建立项目框架

## 1.1. 认识nrfx

文件夹路径：nRF5_SDK_15.3.0_59ac345\modules\nrfx

nrfx由nordic提供的一系列外设驱动组成，无需集成厚重的标准SDK就能把芯片跑起来，是一个轻量级的驱动库。

移植只需要关注以下内容：
- 芯片类型宏定义
- nrfx_config.h
- nrfx_glue.h
- nrfx_log.h
- CMSIS/include

**芯片类型宏定义**：  
在编译阶段必须要加入至少一款芯片类型宏定义，否则编译会报错，后面会在SConscript中加入芯片类型宏定义。

```C
/* Device selection for device includes. */
#if defined (NRF51)
    #include "nrf51.h"
    #include "nrf51_bitfields.h"
    #include "nrf51_deprecated.h"
#elif defined (NRF52810_XXAA)
    #include "nrf52810.h"
    #include "nrf52810_bitfields.h"
    #include "nrf51_to_nrf52810.h"
    #include "nrf52_to_nrf52810.h"
#elif defined (NRF52811_XXAA)
    #include "nrf52811.h"
    #include "nrf52811_bitfields.h"  
    #include "nrf51_to_nrf52810.h"
    #include "nrf52_to_nrf52810.h"
    #include "nrf52810_to_nrf52811.h" 
#elif defined (NRF52832_XXAA) || defined (NRF52832_XXAB)
    #include "nrf52.h"
    #include "nrf52_bitfields.h"
    #include "nrf51_to_nrf52.h"
    #include "nrf52_name_change.h"
#elif defined (NRF52840_XXAA)
    #include "nrf52840.h"
    #include "nrf52840_bitfields.h"
    #include "nrf51_to_nrf52840.h"
    #include "nrf52_to_nrf52840.h"
    
#elif defined (NRF9160_XXAA)
    #include "nrf9160.h"
    #include "nrf9160_bitfields.h"
    
#else
    #error "Device must be defined. See nrf.h."
#endif /* NRF51, NRF52810_XXAA, NRF52811_XXAA, NRF52832_XXAA, NRF52832_XXAB, NRF52840_XXAA, NRF9160_XXAA */
```

**nrfx_config.h**:  
该文件可以配置nrfx驱动，可以在keil在界面配置，如下图所示。

![](nrfx_config.png)

**nrfx_glue.h**  
该文件由`未实现的宏定义`组成，`未实现宏定义`可以看成`钩子函数`，nrfx驱动代码会调用这些`钩子函数`，如果需要用到该钩子函数，则实现，若不需要则留空。

如需要用到临界区，防止嵌套中断发生，nrfx_glue.h文件如下：

```C
/**
 * @brief Macro for entering into a critical section.
 */
#define NRFX_CRITICAL_SECTION_ENTER() {unsigned int ctx; ctx = nrfx_enter_critical();

/**
 * @brief Macro for exiting from a critical section.
 */
#define NRFX_CRITICAL_SECTION_EXIT() nrfx_exit_critical(ctx);}
```

如不需要用到临界区，不考虑嵌套中断发生的情况，nrfx_glue.h文件如下：

```C
/**
 * @brief Macro for entering into a critical section.
 */
#define NRFX_CRITICAL_SECTION_ENTER() 

/**
 * @brief Macro for exiting from a critical section.
 */
#define NRFX_CRITICAL_SECTION_EXIT() 
```


**nrfx_log.h**  
同上

**CMSIS/include**  

根据nrfx根目录README提示，可用doxygen生成文档，生成的文档中有下图红框要求。直接[下载](https://github.com/ARM-software/CMSIS/tree/master/CMSIS/Include)该文件夹即可，不需要修改。

![](cmsis_include.png)

## 1.2. rt-thread构建环境

了解nrfx后，接下来就要将其与rt-thread的构建环境关联起来。

### 1.2.1. 创建bsp文件夹

在bsp目录下创建文件夹myboard，即`rt-thread\bsp\myboard`。


### 1.2.2. 添加bsp的模板工程

> [rt-thread官方NOTE](https://www.rt-thread.org/document/site/programming-manual/scons/scons/)：要生成 MDK 或者 IAR 的工程文件，前提条件是 BSP 目录存在一个工程模版文件，然后 scons 才会根据这份模版文件加入相关的源码，头文件搜索路径，编译参数，链接参数等。而至于这个工程是针对哪颗芯片的，则直接由这份工程模版文件指定。所以大多数情况下，这个模版文件是一份空的工程文件，用于辅助 SCons 生成 project.uvprojx 或者 project.eww。

在调用`scons --target=mdk5`前，需在`rt-thread\bsp\myboard`先提供一个mdk模板工程：

下面简单演示模板工程创建过程。

![](create_template_mdk.gif)

### 1.2.3. 添加bsp的Kconfig

将`rt-thread\bsp\stm3210x\Kconfig`搬运到myboard目录下，删除Kconfig文件中不需要部分。

```
config SOC_STM32F1
bool
select ARCH_ARM_CORTEX_M3
default y
source "$BSP_DIR/drivers/Kconfig"
```

若在myboard目录下执行menuconfig，则menuconfig会根据`rt-thread\bsp\myboard\Kconfig`中的语句生成`rt-thread\bsp\myboard\rtconfig.h`，该头文件包含各种宏定义。

`rt-thread\bsp\myboard\Kconfig`文件可通过source语句引用其他目录下的Kconfig文件，协助其生成所需的宏定义，如以下例子：


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

- 最后其他`.c`文件可引用该头文件，并使用`RT_TICK_PER_SECOND`宏定义。实现在`rt-thread\bsp\myboard`用menuconfig就可以配置整个工程的静态设置，如：

```C
#include <rtconfig.h>

void SystemClock_Config(void)
{
    SysTick_Config(SystemCoreClock / RT_TICK_PER_SECOND);
    NVIC_SetPriority(SysTick_IRQn, 0);
}
```

### 1.2.4. 添加bsp的rtconfig.py

将`rt-thread\bsp\nrf52832\rtconfig.py`搬运到myboard目录下。

当调用`sconsc --target=mdk5`生成mdk工程时候，脚本会根据`rt-thread\bsp\myboard\rtconfig.py`文件中的变量`CPU='cortex-m4'`选择对应的文件添加进工程，如下图针对性添加了m4内核的上下文切换文件。

![](rtconfig_keil.png)

### 1.2.5. 添加bsp的SConscript/SConstruct

将`rt-thread\bsp\nrf52832\SConscript`和`rt-thread\bsp\nrf52832\SConstruct`搬运到myboard目录下。

删除SConscript文件中不需要部分。

```
objs = objs + SConscript(os.path.join(cwd, 'nRF5_SDK_13.0.0_04a0bfd/components/SConscript'))
```

### 1.2.6. 添加nrfx

参考官方的[做法](https://github.com/Zero-Free/nrf52832-nimble/tree/master/nordic)创建一个nordic文件夹，并在`myboard\nordic`目录下添加以下内容：
1. `CMSIS/include`文件夹
2. `nrfx`文件夹
3. SConscript文件

### 1.2.7. 添加设备驱动实现

参考官方的[做法](https://github.com/Zero-Free/nrf52832-nimble/tree/master/drivers)创建一个drivers文件夹，并在`myboard\drivers`目录下添加以下内容：
1. board.c和board.h
2. drv_gpio.c和drv_gpio.h
3. drv_uart.c和drv_uart.h
4. SConscript文件

### 1.2.8. 添加应用层代码

参考官方的[做法](https://github.com/Zero-Free/nrf52832-nimble/tree/master/applications)创建一个applications文件夹，并在`myboard\applications`目录下添加以下内容：
1. application.c
2. SConscript文件

其中application.c文件的main函数中包含打印hello world的代码，至于是如何从上电运行到main函数的，可参考[启动流程](https://www.rt-thread.org/document/site/programming-manual/basic/basic/#rt-thread_1)。

## 1.3. 验证环境构建结果

至此，最简洁的功能移植已经完成，后面验证是否移植成功，若成功串口助手将打印hello world。

进入`rt-thread\bsp\myboard`路径下，右键打开env。

1. 用usb线将板子与PC连接
2. 验证menuconfig功能，同时配置console名字
3. 验证scons --target=mdk5
4. 编译，烧录，观察串口打印信息

![](port_done.gif)

> NOTE：rt_kprintf依赖rt_console_set_device函数  
1、若用rt_kprintf函数打印信息，则需要一个标记为`console设备`的通信接口设备。  
2、rt_console_set_device(console_name)函数从众多已注册通信接口设备中找到与参数console_name相同名字的设备，并将该设备标记为`console设备`。  
3、rt_hw_serial_register(uart0)函数注册一个名字为`uart0`的通信接口设备。  
所以生成mdk工程前，需要用menuconfig配置console_name为`uart0`，否则rt_kprintf就会由于找不到通信接口设备，导致无法打印信息。

# 2. 移植nimble

## 2.1. 主机（host）

## 2.2. 控制器（controller）