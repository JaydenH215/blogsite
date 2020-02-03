---
title: rtt+nimble揭秘系列——controller（7）
date: 2019-12-02 22:42:35
tags:
- rtthread
- nimble

categories:
- 一得之见
---


多角度学习controller，uml还不熟练，欢迎斧正。
<!-- more --> 

前言
===

只学习slave部分，master可以根据slave的思路去看。

目录
===

<!-- TOC -->

- [1. 伪系统框图](#1-伪系统框图)
- [2. 用例图](#2-用例图)
- [3. 类图（未完成）](#3-类图未完成)
- [4. 活动图（未完成）](#4-活动图未完成)
- [5. 状态图（未完成）](#5-状态图未完成)
- [6. 时序图（未完成）](#6-时序图未完成)

<!-- /TOC -->

# 1. 伪系统框图

笔者觉得正常的系统框图不能表现每个组件之间的关系，所以加上一些箭头来表示controller涉及到的组件，箭头指向`被调用的组件`。

![](controller伪系统框图.png)

# 2. 用例图

![](controller用例图.png)

用用例图简单描述系统的功能需求是一个不错的工具，controller的功能比图中的要多，但是我们只针对图中的几个重点功能展开学习，其他细枝末节有需要再看。

# 3. 类图（未完成）

- ble_ll_conn_sm和ble_ll_adv_sm继承了ble_ll_sched_item

gatt 

gap 

l2cap

adv

conn

ll

hci

ll_task

sched

os_cputime

phy（interface）


# 4. 活动图（未完成）

主要从几个主要场景来分析：

1. 广播收到无效包
2. 广播收到扫描请求包
3. 广播收到收到连接请求包
4. 维持连接（丢包重发）
5. 连接收发数据（md）

- 分析sched机制

    - 发送hci->ble_ll_adv.c->ble_ll_sched.c->os_cputime.c->ble_ll_adv.c->ble_phy.c
    - 接收
    ble_phy.c->ble_ll.c->ble_ll_adv.c

    - 从peripheral来切入
        - 1. adv之后的机制
        - 2. 连接之后的机制
    - 调度器提供给不同的链路状态机API，不同链路状态机只能调用sched组件的API，每次调用这些API都会重新调整一下调度队列，实现分时复用，多链路并存。

- 收到消息之后的动作，即从数据输入和输出角度
    - queue角度，evt角度，生产者和消费者

- 消息分配到不同链路层的状态机，分析ll_xx
    - 细致分析每个链路状态机的状态
    - adv event调度解析

- 分析ble_ll_rx_end和ble_ll_rx_start的机制
    - 讲核心规范对于链路层处于不同状态下对空包的crc判断处理策略

# 5. 状态图（未完成）

1. 分析init和task
    - 稳定状态下，没有消息的运行情况，改变这种状态由谁决定

# 6. 时序图（未完成）

配合同步和异步介绍



