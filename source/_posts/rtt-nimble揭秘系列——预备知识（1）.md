---
title: rtt+nimble揭秘系列——预备知识（1）
date: 2019-09-25 01:12:17
tags:
- rtthread
- nimble

categories:
- 一得之见
---


磨刀不误砍柴工，前期多做点理论积累，后期才能少掉坑。
<!-- more --> 

前言
===

[rt-thread简介](https://www.rt-thread.org/document/site/tutorial/quick-start/introduction/introduction/)

[nimble简介](http://mynewt.apache.org/latest/network/docs/index.html#)

目录
===
<!-- TOC -->

- [1. 导读](#1-导读)
- [2. 系统](#2-系统)
- [3. 蓝牙](#3-蓝牙)
- [4. 硬件](#4-硬件)
- [5. 软件](#5-软件)

<!-- /TOC -->

# 1. 导读

知识点根据需要认知的程度分为：`了解`、`理解`、`专业`，对于知识点不建议强行记忆，建议多看、多想和多做，所谓读书百遍其意自现。

| **例子** | **程度**  | **备注** |
| :-- | :--: | :-- |
| 信号量 | 了解 | 两个函数之间的共享变量，像全局变量标志位 | 
|         | 理解 | 工作机制、API使用方法，无须深究源码 |
|         | 专业 | 略 

一般从网上搜一篇文章就可以达到`了解`的程度，`理解`则需要多看几篇。

刚接触一个复杂的软件的时候，不建议深究源码，我个人的习惯是按照：

1. 软件可以运行
2. 理解裁剪
3. 软件刚刚好可以运行
4. 研究细节

就是想说学习方法很重要，不要钻牛角尖。

# 2. 系统

关于rt-thread的入门资料，[rt-thread文档中心](https://www.rt-thread.org/document/site/)这部分做的很cool，其中“内核”、“Env工具”、“设备和驱动”、“代码贡献-软件包开发”章节，建议至少看一遍。

- 知识点1（理解）：内核整个章节
- 知识点2（了解）：env的安装（只是为了运行scons和menuconfig）
- 知识点3（了解）：menuconfig的使用
    - Kconfig（该文件决定rt_config.h包含哪些宏定义，rt_config.h会被nimble协议栈include）
- 知识点4（了解）：scons的使用
    - `scons --target=mdk5`（keil编译和仿真）
    - SConscript（修改该脚本，配合`scons --target=mdk5`来理解协议栈）
- 知识点5（理解）：设备和驱动
    - 移植芯片驱动（尽量理解，后面移植章节介绍）
- 知识点6（了解）：软件包
    - Kconfig文件

这里除了`知识点3`和`知识点5`，其他知识点rt-thread官网已提供足够资料学习。


# 3. 蓝牙

[**蓝牙核心规范下载地址**](https://www.bluetooth.com/specifications/bluetooth-core-specification/)

蓝牙核心规范（bluetooth-core-specification），也常称为蓝牙协议，是一份公开的文档。

不同厂家根据自家芯片的特性，用代码去实现这份文档的内容，这些代码称为蓝牙协议栈。

基于蓝牙协议栈二次开发的工作，称为蓝牙应用开发。

学习蓝牙软件门槛不高，但是会有瓶颈：
- 它不难理解，不像人工智能需要有数学底蕴；它只是稍微复杂一点，花时间都能看懂，所以说门槛不高。
- 因为协议栈不开源，看不到底层实现，所以会遇到瓶颈。

nimble低功耗蓝牙协议栈，层层划分清晰，便于我们理解复杂的实现，而全开源解决了学习蓝牙软件的瓶颈问题，是学习蓝牙规范的一个好工具。

但要是商用就要斟酌一番，毕竟开源不收钱，出现bug只能自力更生。

- 知识点1：（理解）学习蓝牙软件不难，商用nimble需谨慎。
- 知识点2：（了解）蓝牙应用开发可通过博客/Q群/论坛等途径学习

> 推荐红旭论坛：http://bbs.wireless-tech.cn

# 4. 硬件

只需要掌握一个硬件平台的知识点即可。

- 硬件：红旭 | HX-DK夏开发板

    - 知识点1（了解）：学习ble用到的外设GPIO/RTC/UART/PPI/TIMER/RADIO
    - 知识点2（了解）：从官方资料了解串口TX和RX引脚编号、LED灯引脚编号以及外围电路  

- 硬件：nordic | nrf52 dk开发板

    - 知识点1（了解）：学习ble用到的外设GPIO/RTC/UART/PPI/TIMER/RADIO
    - 知识点2（了解）：从开发板背面了解串口TX和RX引脚编号、LED灯引脚编号及外围电路

# 5. 软件

- [rt-thread源码下载地址（后面文章使用v4.0.0版本）](https://www.rt-thread.org/page/download.html)
- [nimble源码下载地址（后面文章使用v1.2.0版本）](http://mynewt.apache.org/download/)
- [mynewt源码下载地址](http://mynewt.apache.org/download/)
- [nrf52_sdk包下载地址（后面文章使用nRF5_SDK_15.3.0_59ac345版本）](https://www.nordicsemi.com/Software-and-Tools/Software/nRF5-SDK)
- [nrf的Device_Family_Pack下载地址（后面文章使用8.24.1版本）](http://www.keil.com/dd2/Pack/#/third-party-download-dialog)
- [rt-thread官方移植nimble源码下载地址](https://github.com/Zero-Free/nrf52832-nimble)
