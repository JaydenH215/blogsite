---
title: rtt+nimble揭秘系列——移植nimble（3）
date: 2019-10-09 00:24:44
tags:
- rtthread
- nimble

categories:
- 一得之见
---

站在巨人的肩膀上可以看的更远。
<!-- more --> 

前言
===

移植目标：手机app可以搜索到ble广播，能成功连接并发现服务。

目录
===

<!-- TOC -->

- [1. 土方法](#1-土方法)
- [2. 移植host](#2-移植host)
    - [2.1. 添加nimble](#21-添加nimble)
    - [2.2. 添加空白SConscript](#22-添加空白sconscript)
    - [2.3. 移植阶段一](#23-移植阶段一)
        - [2.3.1. 无法打开头文件](#231-无法打开头文件)
        - [2.3.2. 无法打开头文件（缺少头文件）](#232-无法打开头文件缺少头文件)
        - [2.3.3. 无法打开头文件（存在多个同名头文件）](#233-无法打开头文件存在多个同名头文件)
        - [2.3.4. 缺少标识符定义](#234-缺少标识符定义)
    - [2.4. 移植阶段二](#24-移植阶段二)
        - [2.4.1. 缺少endian.c中实现的函数](#241-缺少endianc中实现的函数)
        - [2.4.2. 缺少mem.c中实现的函数](#242-缺少memc中实现的函数)
        - [2.4.3. 缺少os_mbuf.c实现的函数](#243-缺少os_mbufc实现的函数)
        - [2.4.4. 缺少os_mempool.c实现的函数](#244-缺少os_mempoolc实现的函数)
        - [2.4.5. 缺少nimble_port.c实现的函数](#245-缺少nimble_portc实现的函数)
        - [2.4.6. 缺少os_msys_init.c实现的函数](#246-缺少os_msys_initc实现的函数)
        - [2.4.7. 缺少tinycrypt加密组件](#247-缺少tinycrypt加密组件)
    - [2.5. 移植阶段三](#25-移植阶段三)
        - [2.5.1. 存在多个ble_hc_trans_xxxx同名函数](#251-存在多个ble_hc_trans_xxxx同名函数)
        - [2.5.2. 缺少ble_npl_xxx函数定义](#252-缺少ble_npl_xxx函数定义)
    - [2.6. 移植host总结](#26-移植host总结)
- [3. 移植controller](#3-移植controller)
    - [3.1. 移植阶段一](#31-移植阶段一)
        - [3.1.1. 无法打开头文件](#311-无法打开头文件)
    - [3.2. 移植阶段二](#32-移植阶段二)
        - [3.2.1. 无法打开头文件（存在多个同名头文件）](#321-无法打开头文件存在多个同名头文件)
        - [3.2.2. 缺少标识符定义](#322-缺少标识符定义)
        - [3.2.3. 无法识别内联汇编](#323-无法识别内联汇编)
        - [3.2.4. 缺少ble_hw.c中实现的函数](#324-缺少ble_hwc中实现的函数)
        - [3.2.5. 缺少os_cputime.c中实现的函数](#325-缺少os_cputimec中实现的函数)
        - [3.2.6. 缺少os_cputime_pwr2.c中实现的函数](#326-缺少os_cputime_pwr2c中实现的函数)
        - [3.2.7. 缺少hal_timer.c中实现的函数](#327-缺少hal_timerc中实现的函数)
        - [3.2.8. 缺少ble_npl_hw_set_isr函数定义](#328-缺少ble_npl_hw_set_isr函数定义)
        - [3.2.9. 缺少ble_npl_hw_is_in_critical函数定义](#329-缺少ble_npl_hw_is_in_critical函数定义)
    - [3.3. 移植controller总结](#33-移植controller总结)
- [4. 移植profile](#4-移植profile)
    - [4.1. 添加心率profile](#41-添加心率profile)
    - [4.2. 移植proifile总结](#42-移植proifile总结)
- [5. 启动运行](#5-启动运行)
    - [5.1. 缺少标识符定义](#51-缺少标识符定义)
    - [5.2. 修复bug](#52-修复bug)
    - [5.3. 添加调试](#53-添加调试)
    - [5.4. 编译烧录](#54-编译烧录)
- [6. 总结](#6-总结)

<!-- /TOC -->


# 1. 土方法

第一个问题，nimble的porting目录看不懂，钻牛角尖要不得，故跑到[大佬](https://github.com/Zero-Free/nrf52832-nimble)的github上寻求解决方案。

然后有了第一个解决思路：基于可以正常运行的工程，将部分nimble源文件从SConscript脚本中移除。

然后第二个问题来了，移除哪些源文件？

按照笔者对[nimble官方文档](http://mynewt.apache.org/latest/network/docs/ble_setup/ble_sync_cb.html)的了解，nimble支持host和controller解耦，那么第一步移植工作就先保留host代码，将controller部分全部移除掉。

# 2. 移植host

## 2.1. 添加nimble

将官网下载的nimble压缩包解压，并将生成的文件夹复制粘贴到myboard目录下，即`rt-thread\bsp\myboard\apache-mynewt-nimble-1.2.0`。

## 2.2. 添加空白SConscript

添加SConscript文件到nimble目录下，即`rt-thread\bsp\myboard\apache-mynewt-nimble-1.2.0\SConscript`。

## 2.3. 移植阶段一

把所有`rt-thread\bsp\myboard\apache-mynewt-nimble-1.2.0\nimble\host`里面除了mesh文件夹和test文件夹以外的.c文件加进刚创建的空白SConscript脚本。

> NOTE：ble mesh后面再挖坑，这里先不展开。

按照上述步骤操作后，编译过程中必然会一路报错，以下是解决方案，请大家对号入座一个个消掉。

### 2.3.1. 无法打开头文件

```
error:  #5: cannot open source input file "sysinit/sysinit.h": No such file or directory
```

错误原因：nimble包有该文件，只是没有将头文件目录添加进编译脚本  
解决方法：在nimble包中找到唯一对应的头文件，并在SConscript中添加路径，该例子需要添加的路径为`cwd + '/porting/nimble/include'`，同类报错可用同样方法解决。

### 2.3.2. 无法打开头文件（缺少头文件）

```
error:  #5: cannot open source input file "base64/base64.h": No such file or directory
```

错误原因：nimble包没有该文件  
解决办法：从mynewt的`encoding`目录下，拷贝`base64`到`rt-thread\bsp\myboard\apache-mynewt-nimble-1.2.0\ext\`，并在SConscript中添加路径，该例子需要添加的路径为`cwd + '/ext/base64/include'`。

### 2.3.3. 无法打开头文件（存在多个同名头文件）

```
error:  #5: cannot open source input file "nimble/nimble_npl_os.h": No such file or directory
```

错误原因：nimble包有多个该文件，只是没有将头文件目录添加进编译脚本  
解决方法：该文件定义了os的抽象接口，协议栈会调用这些接口，这些接口的实现每个os都不一样，我们这里参考[rt-thread官方做法](https://github.com/Zero-Free/nrf52832-nimble/tree/master/packages/NimBLE-latest/porting/npl/rtthread)，直接下载该文件夹到`rt-thread\bsp\myboard\apache-mynewt-nimble-1.2.0\porting\npl`下，并在SConscript中添加路径，该例子需要添加的路径为`cwd + '/porting/npl/rtthread/include'`。

### 2.3.4. 缺少标识符定义

```
error:  #20: identifier "MYNEWT_VAL_BLEUART_MAX_INPUT" is undefined
```

错误原因：缺少宏定义  
解决方法：将`nimble/host/services/bleuart/src/bleuart.c`从SConscript脚本中删除，暂不考虑实现该profile。

```
error:  #20: identifier "MYNEWT_VAL_BLE_SVC_DIS_MODEL_NUMBER_DEFAULT" is undefined
```

错误原因：缺少宏定义  
解决方法：将`nimble/host/services/dis/src/ble_svc_dis.c`从SConscript脚本中删除，暂不考虑实现该profile。

## 2.4. 移植阶段二

经过上节处理之后，编译均是以下错误：

```
Error: L6218E: Undefined symbol xxxx (referred from xxxx).
```

host文件夹内的代码会引用其他目录下的`公共组件`以及`公共函数`。组件如：内存池，函数如：大小端转换函数。

所以该阶段目标是将host文件夹依赖的`公共组件`和`公共函数`补充完整。

### 2.4.1. 缺少endian.c中实现的函数

解决方法：将`porting/nimble/src/endian.c`添加进SConscript脚本。

### 2.4.2. 缺少mem.c中实现的函数

解决方法：将`porting/nimble/src/mem.c`添加进SConscript脚本。

### 2.4.3. 缺少os_mbuf.c实现的函数

解决方法：将`porting/nimble/src/os_mbuf.c`添加进SConscript脚本。

### 2.4.4. 缺少os_mempool.c实现的函数

解决方法：将`porting/nimble/src/os_mempool.c`添加进SConscript脚本。

### 2.4.5. 缺少nimble_port.c实现的函数

解决方法：将`porting/nimble/src/nimble_port.c`添加进SConscript脚本。

### 2.4.6. 缺少os_msys_init.c实现的函数

解决方法：将`porting/nimble/src/os_msys_init.c`添加进SConscript脚本。


### 2.4.7. 缺少tinycrypt加密组件

解决方法：添加以下路径到SConscript文件。  
`ext/tinycrypt/src/aes_decrypt.c`  
`ext/tinycrypt/src/aes_encrypt.c`  
`ext/tinycrypt/src/utils.c`


## 2.5. 移植阶段三

经过上节处理之后，编译剩下以下错误：

```
1. Error: L6218E: Undefined symbol ble_hci_trans_xxxx (referred from xxxx).
2. Error: L6218E: Undefined symbol ble_npl_xxx (referred from xxxx).
```

前面做的移植操作，哪个平台都是一样的。而对于这两类错误，不同的应用场景和平台，解决方案会有所差异。

### 2.5.1. 存在多个ble_hc_trans_xxxx同名函数

全局搜索nimble包会发现这些函数在多个.c文件中都有实现，位置如下：

```
 apache-mynewt-nimble-1.2.0
 └──nimble         
     └──transport
            ├──da1469x
            │     └──src
            │         └──da1469x_ble_hci.c
            ├──emspi
            │    └──src
            │        └──ble_hci_emspi.c
            ├──ram
            │    └──src
            │        └──ble_hci_ram.c
            ├──socket
            │    └──src
            │        └──ble_hci_socket.c
            └──uart
                └──src
                    └──ble_hci_uart.c

```

这意味着我们需要做出选择，为这些函数接口选择具体实现。这时候引入新问题，“我们在选择什么”。

参考[这篇blog](https://jaydenh215.github.io/2019/10/09/%E6%B5%85%E8%B0%88BLE%E6%A0%B8%E5%BF%83%E6%9E%B6%E6%9E%84%E5%92%8C%E6%95%B0%E6%8D%AE%E5%B8%A7/)，可知：

host和controller可通过hci（host-controller-interface）接口互相通信。具体表现为：

- host通过hci向controller发送command
- controller通过hci向host发送event
- host和controller通过hci互传acl data

而编译报错缺失的ble_hc_trans_xxxx函数为：

- ble_hci_trans_reset
- ble_hci_trans_cfg_hs（该API注册接收evt和acl data的回调函数）
- ble_hci_trans_buf_alloc
- ble_hci_trans_buf_free
- ble_hci_trans_hs_cmd_tx
- ble_hci_trans_hs_acl_tx

也就是我们在选择用什么样的transport来实现hci功能，即：

- 如果我们需要在一颗芯片上运行host+controller，那么就要选ble_hci_ram.c文件，即host和controller之间用ram通信。
- 如果我们需要在一颗芯片上运行host，另一颗芯片运行controller，那么就可以选ble_hci_uart.c文件，即host和controller之间用uart通信。

> NOTE：市场上一些简单的智能设备，如手环、心率带、无线传感器，都会选择在一颗芯片上运行完整的协议栈，即host+controller，无论在成本还是开发上都会简单很多。但是一些比较复杂如多协议网关、机顶盒的产品，则需要一个强大的cpu跑linux，并用linux实现host（bluez、bluedroid），然后外挂一颗性能和成本较低的蓝牙芯片跑controller。

这里我们选择ble_hci_ram.c文件，将`nimble/transport/ram/src/ble_hci_ram.c`和`cwd + '/nimble/transport/ram/include'`加进SConscript。

### 2.5.2. 缺少ble_npl_xxx函数定义

npl全称为nimble porting layer，顾名思义，这些函数是协议栈需要调用的接口，协议栈不会实现这部分内容，需要我们自己去实现，协议栈用到的所有接口在`nimble_npl.h`中。

这部分nimble没有注释说明接口该如何实现，移植者只能参考其他os的实现以及nimble协议栈调用接口代码，去猜这个函数做了什么，我不喜欢这一点。

这里我们将rt-thread官方已经移植好的代码加进SConscript，即`porting/npl/rtthread/src/npl_os_rtthread.c`。

> NOTE：需要注释掉npl_os_rtthread.c里面的ble_npl_task_init函数。

## 2.6. 移植host总结

总的来说，nimble中移植host功能需要关注以下几种类型代码。

1. host协议核心代码
2. 公用组件和公用函数
3. hci transport
4. os特定代码

[myboard文件夹下载地址](https://github.com/JaydenH215/rtt_nimble/tree/master/myboard_with_nimble_host)

# 3. 移植controller

## 3.1. 移植阶段一

把所有`rt-thread\bsp\myboard\apache-mynewt-nimble-1.2.0\nimble\controller`里面除了test文件夹以外的.c文件加进之前的SConscript脚本。

按照上述步骤操作后，编译过程中必然会一路报错，以下是解决方案，请大家对号入座一个个消掉。

### 3.1.1. 无法打开头文件

```
error:  #5: cannot open source input file "controller/ble_ll.h": No such file or directory
```

错误原因：nimble包有该文件，只是没有将头文件目录添加进编译脚本  
解决方法：在nimble包中找到唯一对应的头文件，并在SConscript中添加路径，该例子需要添加的路径为`cwd + '/nimble/controller/include'`，同类报错可用同样方法解决。

## 3.2. 移植阶段二

前面做的移植操作，无论哪个平台都是一样的。而接下来的操作，不同的应用场景和平台，解决方案会有所差异。

### 3.2.1. 无法打开头文件（存在多个同名头文件）

```
error:  #5: cannot open source input file "ble/xcvr.h": No such file or directory
```
xcvr英文原名为transceiver，即既有包含发射器，又有接收器的radio设备。

1. xcvr正常运行前需要有一段rampup时间
2. xcvr在发射器和接收器之间切换，需要软件执行时间

因为不同设备，这两个参数会有比较大差异，而ble对于时间的要求又是us级别的，所以需要移植者根据自己的硬件设备来实现。

这里我们选择nrf52的xcvr文件，将`nimble/drivers/nrf52/src/ble_phy.c`和`cwd + '/nimble/drivers/nrf52/include'`加进SConscript。

### 3.2.2. 缺少标识符定义

```
error:  #20: identifier "MYNEWT_VAL_BLE_SVC_DIS_MODEL_NUMBER_DEFAULT" is undefined
```

协议栈要正常运行必须有一些默认参数，否则编译不通过，如：射频发射功率、支持最大连接数等，参数以及参数的说明可以从目录对应的`syscfg.yml`文件获知。

由于我们不使用rt-thread提供的软件包功能，所以将用[该文件](https://github.com/JaydenH215/rtt_nimble/blob/master/myboard_with_nimble_host_controller_profile/apache-mynewt-nimble-1.2.0/porting/npl/rtthread/include/config/config.h)替换如下路径同名文件：`rt-thread\bsp\myboard\apache-mynewt-nimble-1.2.0\porting\npl\rtthread\include\config\config.h`。

同时在`rt-thread\bsp\myboard\apache-mynewt-nimble-1.2.0\porting\nimble\include\syscfg\syscfg.h`文件开头位置中include该头文件：

```C

#ifndef H_MYNEWT_SYSCFG_
#define H_MYNEWT_SYSCFG_

...

#define MYNEWT_VAL(x)                           MYNEWT_VAL_ ## x

#include "config/config.h"
...

#endif
```

> NOTE：因为通过宏定义配置参数的文件都会include <syscfg/syscfg.h>文件，将include "config/config.h"放在该文件前可以起到全局配置的作用。


### 3.2.3. 无法识别内联汇编

```
error:  #18: expected a ")"
```

用c来实现内联汇编的功能，用[该文件](https://github.com/JaydenH215/rtt_nimble/blob/master/myboard_with_nimble_host_controller_profile/apache-mynewt-nimble-1.2.0/nimble/drivers/nrf52/src/ble_phy.c)替换如下路径同名文件，即：`rt-thread\bsp\myboard\apache-mynewt-nimble-1.2.0\nimble\drivers\nrf52\src\ble_phy.c`。

### 3.2.4. 缺少ble_hw.c中实现的函数

解决方法：将`nimble/drivers/nrf52/src/ble_hw.c`添加进SConscript脚本。

### 3.2.5. 缺少os_cputime.c中实现的函数

解决方法：将`porting/nimble/src/os_cputime.c`添加进SConscript脚本。

### 3.2.6. 缺少os_cputime_pwr2.c中实现的函数

解决方法：将`porting/nimble/src/os_cputime_pwr2.c`添加进SConscript脚本。

### 3.2.7. 缺少hal_timer.c中实现的函数

解决方法：将`porting/nimble/src/hal_timer.c`添加进SConscript脚本。

### 3.2.8. 缺少ble_npl_hw_set_isr函数定义

```
Error: L6218E: Undefined symbol ble_npl_hw_set_isr (referred from ble_phy.o).
```

该接口用于设置中断号以及对应的中断服务函数。

这里我们将`porting/npl/rtthread/src/nrf5x_isr.c`添加进SConscript脚本。

### 3.2.9. 缺少ble_npl_hw_is_in_critical函数定义

在`rt-thread\bsp\myboard\apache-mynewt-nimble-1.2.0\porting\npl\rtthread\src\npl_os_rtthread.c`中修改如下实现：

```C

volatile int ble_npl_in_critical = 0;


uint32_t ble_npl_hw_enter_critical(void)
{
    ++ble_npl_in_critical;
    return rt_hw_interrupt_disable();
}

void ble_npl_hw_exit_critical(uint32_t ctx)
{
    --ble_npl_in_critical;
    rt_hw_interrupt_enable(ctx);
}

bool ble_npl_hw_is_in_critical()
{
    return (ble_npl_in_critical > 0);
}
```

## 3.3. 移植controller总结

总的来说，nimble中移植controller功能需要关注以下几种类型代码。

1. controller协议核心代码
2. 硬件平台差异文件（ble_phy.c/ble_hw.c/nrf5x_isr.c）
3. 公用组件和公用函数(hal_timer.c/...)
4. 应用宏定义配置文件（config.h/syscfg.yml）

[myboard文件夹下载地址](https://github.com/JaydenH215/rtt_nimble/tree/master/myboard_with_nimble_host_controller)

# 4. 移植profile

不同的应用场景对于蓝牙核心协议栈（host+controller）的用法不一样，为了规范化和互联互通，针对某些应用场景，会有一套蓝牙核心协议栈的操作指南，这份指南称为profile。

## 4.1. 添加心率profile

下载[rt-thread官方推荐的应用代码](https://github.com/Zero-Free/nrf52832-nimble/blob/master/packages/NimBLE-latest/apps/blehr/src/blehr.c)，添加进`rt-thread\bsp\myboard\apache-mynewt-nimble-1.2.0\apps\blehr\src\`文件夹。

1. 将`apps/blehr/src/gatt_svr.c`添加进SConscript脚本。
2. 将`apps/blehr/src/blehr.c`添加进SConscript脚本。
2. 将`cwd + '/apps/blehr/src'`添加进SConscript脚本。

## 4.2. 移植proifile总结

添加profile的过程比较杂乱，但这没办法，因为profile和os、nimble的启动方式等等都有耦合，特别是rtthread这里启动nimble的方式有点绕。


# 5. 启动运行

经过上节处理之后，编译报以下错误：

```
Error: L6218E: Undefined symbol ble_hs_thread_startup (referred from blehr.o).
```

解决方法：将`porting/npl/rtthread/src/nimble_port_rtthread.c`添加进SConscript脚本。

通过阅读`nimble_port_rttherad.c`文件得知，如果需要支持controller，还需要先增加宏定义`NIMBLE_CFG_CONTROLLER=1`，所以将该宏定义添加进SConscript脚本。

## 5.1. 缺少标识符定义

```
error:  #20: identifier "MYNEWT_VAL_BLE_CTLR_THREAD_STACK_SIZE" is undefined

error:  #20: identifier "MYNEWT_VAL_BLE_CTLR_THREAD_PRIORITY" is undefined

error:  #20: identifier "MYNEWT_VAL_BLE_HOST_THREAD_STACK_SIZE" is undefined

error:  #20: identifier "MYNEWT_VAL_BLE_HOST_THREAD_PRIORITY" is undefined
```

在`porting/npl/rtthread/include/config/config.h`文件添加以下内容：

```C
//thread config
#ifndef MYNEWT_VAL_BLE_HOST_THREAD_STACK_SIZE
#define MYNEWT_VAL_BLE_HOST_THREAD_STACK_SIZE     (2048)
#endif

#ifndef MYNEWT_VAL_BLE_HOST_THREAD_PRIORITY
#define MYNEWT_VAL_BLE_HOST_THREAD_PRIORITY       (5)
#endif


#ifndef MYNEWT_VAL_BLE_CTLR_THREAD_STACK_SIZE
#define MYNEWT_VAL_BLE_CTLR_THREAD_STACK_SIZE       (2048)
#endif

#ifndef MYNEWT_VAL_BLE_CTLR_THREAD_PRIORITY
#define MYNEWT_VAL_BLE_CTLR_THREAD_PRIORITY         (4) // must higher than MYNEWT_VAL_BLE_HOST_TASK_PRIORITY, numerically smaller
#endif
```

## 5.2. 修复bug

将`rt-thread\bsp\myboard\apache-mynewt-nimble-1.2.0\porting\nimble\include\os\endian.h`文件
```C
#if __BYTE_ORDER__ == __ORDER_BIG_ENDIAN__
```

改为：

```C
#if defined (__BYTE_ORDER__) && (__BYTE_ORDER__ == __ORDER_BIG_ENDIAN__)
```

## 5.3. 添加调试

将`porting/npl/rtthread/src/modlog.c`添加进SConscript脚本。

用[该文件](https://github.com/JaydenH215/rtt_nimble/blob/master/myboard_with_nimble_host_controller_profile/apache-mynewt-nimble-1.2.0/porting/npl/rtthread/src/modlog.c)替换`rt-thread\bsp\myboard\apache-mynewt-nimble-1.2.0\porting\npl\rtthread\src\modlog.c`。

用[该文件](https://github.com/JaydenH215/rtt_nimble/blob/master/myboard_with_nimble_host_controller_profile/apache-mynewt-nimble-1.2.0/porting/nimble/include/modlog/modlog.h)替换`rt-thread\bsp\myboard\apache-mynewt-nimble-1.2.0\porting\nimble\include\modlog\modlog.h`。

## 5.4. 编译烧录

[myboard文件夹下载地址](https://github.com/JaydenH215/rtt_nimble/tree/master/myboard_with_nimble_host_controller_profile)

生成工程、编译、烧录后，在串口调试助手输入`ble_hr`，设备开始广播。

![](ble_hr.gif)

# 6. 总结

通过这一章节的学习，对于nimble的目录结构、参数配置方式，有更深刻的认识。













