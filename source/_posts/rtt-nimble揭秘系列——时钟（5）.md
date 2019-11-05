---
title: rtt+nimble揭秘系列——时钟（5）
date: 2019-10-29 20:21:20
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

理解不对的地方欢迎斧正。

nimble的时钟管理主要由以下文件组合实现:`hal_timer.c`、`hal_timer.h`、、`os_cuptime.h`、`os_cuptime.c`、`board.c`、`npl_os_rtthread.c`、`nimble_npl.h`。

目录
===

<!-- TOC -->

- [1. 总览](#1-总览)
- [2. controller](#2-controller)
    - [2.1. hal_timer.c & hal_timer.h](#21-hal_timerc--hal_timerh)
        - [2.1.1. RTC0计时技巧](#211-rtc0计时技巧)
        - [2.1.2. RTC0中断处理函数](#212-rtc0中断处理函数)
    - [2.2. os_cputime.c & os_cputime.h](#22-os_cputimec--os_cputimeh)
- [3. host](#3-host)
    - [3.1. board.c](#31-boardc)
    - [3.2. npl_os_rtthread.c](#32-npl_os_rtthreadc)
- [总结](#总结)
- [4. 参考资料](#4-参考资料)

<!-- /TOC -->

# 1. 总览

![](总览.png)

rt-thread和nimble需要用到两类定时器服务：`实时性强`和`实时性弱`。

- 实时性强：
    - controller：需要软件完成实时性和优先级都**最高**的蓝牙基带内容，如：时间片分配，同步anchor、丢包重发机制等，这时RTC0的中断优先级应该全局最高。

- 实时性弱：
    - controller：链路层的控制过程，这些对于实时性要求弱。
    - rt-thread：需要一个系统时钟，这里用RTC1提供时间片。
    - host：需要用到定时服务叫callout，callout使用rt-thread的一个软件定时器实现（callout在npl_os_rtthread.c文件中实现）。

> NOTE：之所以不用Systick而是RTC提供rt-thread系统时钟是因为ble对功耗有要求，RTC功耗更低。

# 2. controller

## 2.1. hal_timer.c & hal_timer.h

基于RTC0和链表实现了软件定时器的功能，即维护一条链表：链表第一个元素即是将要到达的超时事件，第二个为晚一点的超时事件，第三个更晚一点的超时事件。

![](hal_timer.jpg)

hal_timer如果有读不明白的地方，可以参考下面两个小节的提示。

### 2.1.1. RTC0计时技巧

- 每次RTC0溢出（有效范围bit0~bit23），tmr_cntr += 0x1000000，即通过`unsigned int tmr_cntr`变量来扩展只有24bit的RTC。

- 定时器应用可以抽象出这么一种情景，即存在两个时刻，一个为目标时刻，另一个为实时时刻，需要比较这两个时刻谁先谁后。而实时时刻是一直在增加的变量，所以会存在溢出情况，则涉及实时时刻存在溢出情况，与目标时刻比较谁先谁后问题。这里用参考资料的[time_after和time_before](https://www.cnblogs.com/chaozhu/p/6183537.html)的方式巧妙解决。

### 2.1.2. RTC0中断处理函数

- RTC溢出
    - 需要进入中断更新计时变量tmr_cntr，若屏蔽这个中断，则无法知道RTC究竟跑了多长时间。
- COMPARE置位
    - 当需要延时的时间< 3 tick时候，人工置位COMPARE标志位触发中断
    - 当需要延时的时间≥ 3 tick时候，设置CC寄存器，硬件自动置位COMPARE标志位触发中断。

每次进入RTC0中断都会更新计时变量tmr_cntr，然后判断下一个即将要到达的超时事件事件是否< 3 tick，若成立，则立马调用下一个即将要到达超时事件的cb，否则将CC设置为下一个超时事件的时间。


## 2.2. os_cputime.c & os_cputime.h

基于hal_timer简单封装的一层。

# 3. host

## 3.1. board.c

```C
简化伪代码

void RTC1_IRQHandler(void)
{
    ...
    set_comparator(cur_timer + CYC_PER_TICK);
    rt_tick_increase(); //rtthread系统时钟tick++
    ...
}

void RTC1_init(void)
{
    ...
    lfclk_init();
    set_comparator(cur_timer + CYC_PER_TICK);
    ...
}
```

1. 配置lfclk时钟（提供RTCx的时钟源）。
2. 根据设置的rt-thread系统时钟频率来配置RTC1的compare值
3. 每次compare中断将rt-thread系统时钟tick++
4. 设置下一个compare值

> NOTE：需要注意RTC0也依赖lfclk时钟。

## 3.2. npl_os_rtthread.c

该文件主要定义一个简单的事件驱动机制：callout

该组件由定时器、事件、事件队列组成，这些功能在不同平台都具有共性，所以可以抽象出来接口。

callout广泛运用在host中，在controller也有，具体在下一篇中介绍。

# 总结

时钟抽象分层工作做的很不错，起码时钟这一块需要移植到其他os的东西很清晰。

# 4. 参考资料
- [jiffies溢出与时间先后比较---time_after,time_before](https://www.cnblogs.com/chaozhu/p/6183537.html)
