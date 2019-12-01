---
title: rtt+nimble揭秘系列——phy（6）
date: 2019-11-07 23:04:37
tags:
- rtthread
- nimble

categories:
- 一得之见
---

主要介绍phy层的射频工作机制。
<!-- more --> 

前言
===

物理层与芯片本身强相关，这里仅分析nrf52的phy层调度相关内容。

目录
===

<!-- TOC -->

- [1. 术语](#1-术语)
- [2. 总览](#2-总览)
    - [2.1. 框图](#21-框图)
    - [2.2. 广播调度例子](#22-广播调度例子)
- [3. phy对象的内部属性](#3-phy对象的内部属性)
    - [3.1. xcvr clock settle time](#31-xcvr-clock-settle-time)
    - [3.2. various radio timings](#32-various-radio-timings)
        - [3.2.1. ramp-up times](#321-ramp-up-times)
        - [3.2.2. delay between EVENTS_READY and start of tx](#322-delay-between-events_ready-and-start-of-tx)
        - [3.2.3. delay between EVENTS_END and end of txd packet](#323-delay-between-events_end-and-end-of-txd-packet)
        - [3.2.4. delay between rxd access address and EVENTS_ADDRESS](#324-delay-between-rxd-access-address-and-events_address)
        - [3.2.5. delay between end of rxd packet and EVENTS_END](#325-delay-between-end-of-rxd-packet-and-events_end)
- [4. phy对象的对外方法](#4-phy对象的对外方法)
    - [4.1. ble_phy_tx_set_start_time](#41-ble_phy_tx_set_start_time)
    - [4.2. ble_phy_rx_set_start_time](#42-ble_phy_rx_set_start_time)
    - [4.3. ble_phy_set_txend_cb](#43-ble_phy_set_txend_cb)
    - [4.4. ble_phy_tx](#44-ble_phy_tx)
    - [4.5. ble_phy_rx](#45-ble_phy_rx)
    - [4.6. ble_phy_rxpdu_copy](#46-ble_phy_rxpdu_copy)
- [5. phy对象使用其他对象的方法](#5-phy对象使用其他对象的方法)
    - [5.1. ble_ll_rx_start](#51-ble_ll_rx_start)
    - [5.2. ble_ll_rx_end](#52-ble_ll_rx_end)
    - [5.3. ble_ll_state_get](#53-ble_ll_state_get)
    - [5.4. ble_ll_wfr_timer_exp](#54-ble_ll_wfr_timer_exp)
- [6. phy对象的内部实现](#6-phy对象的内部实现)
    - [6.1. 概述](#61-概述)
        - [6.1.1. 在指定的时刻发送射频数据](#611-在指定的时刻发送射频数据)
        - [6.1.2. 在指定的时刻接收射频数据，若在后续指定时间段内没收到，则认为接收超时，若收到则传递给上层处理](#612-在指定的时刻接收射频数据若在后续指定时间段内没收到则认为接收超时若收到则传递给上层处理)
        - [6.1.3. 切换射频发送和射频接收](#613-切换射频发送和射频接收)
    - [6.2. 详细说明](#62-详细说明)
- [7. 补充说明](#7-补充说明)
    - [7.1. 时间补偿](#71-时间补偿)
    - [7.2. radio数据包配置](#72-radio数据包配置)
- [8. 关闭timer0的时机](#8-关闭timer0的时机)
- [9. 总结](#9-总结)

<!-- /TOC -->

# 1. 术语

- wfr：wait for response（用于计算radio rx超时时间）
- xcvr：transceiver（发射接收器）
- ll：link layer（链路层）

# 2. 总览

## 2.1. 框图
phy对象内部会使用其他对象的方法。

![](phy.png)

## 2.2. 广播调度例子

下图是一个广播例子，简单展示了phy对象对内对外是如何协同工作，完成调度的。

![](adv_sch.jpg)

# 3. phy对象的内部属性

## 3.1. xcvr clock settle time

因为需要保持低功耗，所以不需要使用xcvr时候，应该将其时钟也一同关闭，当需要在某个时刻再次打开xcvr时候，需要考虑时钟的settle time，即需提前先打开xcvr时钟。

## 3.2. various radio timings

radio外设相关的延时

### 3.2.1. ramp-up times

ramp-up times即操作射频收发寄存器后，直到真正发出射频信号或者已准备好接收射频信号的所需时间。

需考虑这部分时间确保在预定时刻发送射频信号。

### 3.2.2. delay between EVENTS_READY and start of tx

产生EVENTS_READY后，到真正开始发送射频信号的延时。

因为启动射频是由ppi+radio+timer0+rtc0来联动的，即timer0时间到触发RADIO->TASKS_TXEN，然后延时ramp-up time，接着产生EVENTS_READY，由于radio short，直接TASKS_START。需考虑这部分时间确保在预定时刻发送射频信号。

### 3.2.3. delay between EVENTS_END and end of txd packet

产生EVENTS_END后，到真正发送完射频信号的延时。

切换成接收/关闭射频时，需考虑这部分时间确保射频数据发送完整，防止发送数据断尾。

### 3.2.4. delay between rxd access address and EVENTS_ADDRESS

收到合法access addr到产生EVENTS_ADDRESS的延时。

当启动radio接收超时定时器时，需考虑这部分时间，确保在指定时间内真的没有收到合法的access addr。

当计算当前数据包何时被捕获时，需考虑这部分时间，用于反推。

### 3.2.5. delay between end of rxd packet and EVENTS_END

接收一包结束到产生EVENTS_END的延时。

当从接收切换成发送，计算上一次捕获到数据包最后一个bit的时刻，然后增加T_IFS启动发送，需考虑这部分时间确保在预定时刻发送射频信号。

# 4. phy对象的对外方法

## 4.1. ble_phy_tx_set_start_time

nimble中射频发射有两种应用场景：

1. rx结束，根据T_IFS切换到tx
2. 链路层发起tx

第1中情况会在phy的`ble_phy_rx_end_isr`中实现，而第2种实现就是调用`ble_phy_tx_set_start_time`函数了，该函数用于设定一个定时时间（粗略定时rtc0+精准定时timer0），radio会在该时刻发送preamble的第一个bit。

## 4.2. ble_phy_rx_set_start_time

nimble中射频接收有两种应用场景：

1. tx结束，根据T_IFS切换到rx
2. 链路层发起rx

第1中情况会在phy的`ble_phy_tx_end_isr`中实现，而第2种实现就是调用`ble_phy_rx_set_start_time`函数了，该函数用于设定一个定时时间（粗略定时rtc0+精准定时timer0），radio会在该时刻开始接收射频数据。

## 4.3. ble_phy_set_txend_cb

该函数输入参数有一个是函数指针，在`ble_phy_tx_end_isr`中打钩子函数，用于通知链路层发送结束。

## 4.4. ble_phy_tx

用于配置射频发送的参数，比如发送buffer、radio的shortcut配置和发送结束后是否需要自动切换到rx，但是不会真正启动射频发送。

## 4.5. ble_phy_rx

用于配置射频接收的参数，比如接收buffer，radio的shortcut配置，但是不会真正启动射频接收。

## 4.6. ble_phy_rxpdu_copy

用于获取上一次phy rx的pdu数据。

    
# 5. phy对象使用其他对象的方法

## 5.1. ble_ll_rx_start

收到pdu第一个字节之后，phy调用该方法，根据pdu的第一个字节即pdu type，来决定是否继续接收下面的帧，以及接收完应该如何处理。

## 5.2. ble_ll_rx_end

收完一帧，开始处理这一帧，phy根据该方法返回结果判断是否要切换到tx，还是关闭射频外设。

## 5.3. ble_ll_state_get

获取当前链路层的状态，封装进`ble mbuf header`，在ble_ll_rx_end中调用，让链路层知道传递上去的帧是在哪个状态时接收到的。

## 5.4. ble_ll_wfr_timer_exp

如果接收超时，就会调用该函数。比如，在37信道发送可连接可发现广播，切换到接收后，没有在指定时间内收到CONNECT_IND或者SCAN_REQ，就会调用这个函数。

# 6. phy对象的内部实现

内部实现主要涉及`isr`和`ppi`、`timer0`、`rtc0`，下面先介绍它们之间的关系，然后分别介绍每个中断和外设的功能作用，最后举例在一些常见应用场景中是如何协同工作的。

## 6.1. 概述

phy核心功能可以用以下几句话总结：
1. 在指定的时刻发送射频数据。
2. 在指定的时刻接收射频数据，若在后续指定时间段内没收到，则认为接收超时，若收到则传递给上层处理。
3. 切换射频发送和射频接收

### 6.1.1. 在指定的时刻发送射频数据
将`指定时刻`拆成尽可能多的`省电系统时钟tick`和一个`耗电精准时钟tick`，rtc0的CC[0]设置为省电tick，timer0的CC[0]设置为耗电tick。

当rtc0的CC[0]事件触发时，配合ppi启动timer0。
当timer0的CC[0]事件触发时，配合ppi启动射频发送。

### 6.1.2. 在指定的时刻接收射频数据，若在后续指定时间段内没收到，则认为接收超时，若收到则传递给上层处理

将`指定时刻`拆成尽可能多的`省电系统时钟tick`和尽可能少的`耗电精准时钟tick`，rtc0的CC[0]设置为尽可能多的省电tick，timer0的CC[0]设置为尽可能少的耗电tick。将`接收超时事件`拆成N个`耗电精准时钟tick`，timer0的CC[3]设置为N个耗电tick。

当rtc0的CC[0]事件触发时，配合ppi启动timer0。
当timer0的CC[0]事件触发时，配合ppi启动射频接收。
当timer0的CC[3]事件触发时，配合ppi关闭射频接收。

### 6.1.3. 切换射频发送和射频接收

- 链路层在启动射频发射时会设置`g_ble_phy_data.phy_transition`变量，在phy的ble_phy_tx_end_isr中会根据该变量判断下一步是切换到rx，还是关闭射频外设。

- 在ble_ll_wfr_timer_exp或ble_phy_rx_end_isr中会把接收结果传递给链路层，链路层决定下一步是切换到tx，还是关闭射频外设。

## 6.2. 详细说明

- ble_phy_rx_start_isr：收到access addr以及第一个字节后，使能RADIO_INTENSET_END_Msk中断。
- ble_phy_rx_end_isr：接收一帧后会进入该分支，传递给链路层刚刚收到的那一帧。
- ble_ll_wfr_timer_exp：接收超时会进入该分支，表示指定时间内没收到有效数据包。
- ble_phy_tx_end_isr：发送玩一帧后会进入该分支，通知链路层发送完成，判断下一步是切换到rx，还是关闭射频外设。

- ch4：当收到rx的RADIO->EVENT_ADDRESS时，触发NRF_TIMER0->TASKS_CAPTURE[3]，取消wfr定时器。

- ch5：当收到NRF_TIMER0->EVENTS_COMPARE[3]，即wfr定时器超时，则触发NRF_RADIO->TASKS_DISABLE，关闭射频。

- ch20：当收到TIMER0->EVENTS_COMPARE[0]时，触发RADIO->TASKS_TXEN，即通过timer0来硬件自动打开射频发射。

- ch21：当收到TIMER0->EVENTS_COMPARE[0]时，触发RADIO->TASKS_RXEN，即通过timer0来硬件自动打开射频接收。

- ch26：当收到tx/rx的RADIO->EVENT_ADDRESS时，触发TIMER0->TASK_CAPTURE[1]，因为ADDRESS域是固定长度，所以可以用来计算捕获tx/rx时刻。

- ch27：当收到tx/rx的RADIO->EVENT_END时，触发TIMER0->TASK_CAPTURE[2]，用于捕获tx/rx结束（即收到CRC域最后一个bit）时刻。

- ch31：当收到RTC0->EVENTS_COMPARE[0]时，触发TIMER0->TASKS_START，即通过RTC0来控制启动timer0，具体为用户指定一个时刻（RTC0 tick + timer0 us），当收到RTC0->EVENTS_COMPARE[0]时，说明tick（31us/tick）已经计算完成，打开timer0计算剩下的时间（1us/tick）。

# 7. 补充说明

## 7.1. 时间补偿

cpu_time和rem_usecs，cpu_time表示从调用函数到目标时刻需要的tick个数，rem_usecs表示除了cpu_time之外，还需要多少个us才能达到目标时刻，且rem_usecs应该小于31。cpu_time与rtc外设有关，rem_usecs与timer外设有关。

## 7.2. radio数据包配置

```C
#define NRF_LFLEN_BITS          (8)     //对应pdu header的8位lsb
#define NRF_S0LEN               (1)     //对应pdu header的8位msb
#define NRF_S1LEN_BITS          (0)     //这个域的长度为0，即舍弃该域
#define NRF_MAXLEN              (255)   //对应pdu payload
#define NRF_BALEN               (3)     //对应llpkt的4字节access address

#define NRF_PCNF0               (NRF_LFLEN_BITS << RADIO_PCNF0_LFLEN_Pos) | \
                                (RADIO_PCNF0_S1INCL_Msk) | \
                                (NRF_S0LEN << RADIO_PCNF0_S0LEN_Pos) | \
                                (NRF_S1LEN_BITS << RADIO_PCNF0_S1LEN_Pos)
#define NRF_PCNF0_1M            (NRF_PCNF0) | \
                                (RADIO_PCNF0_PLEN_8bit << RADIO_PCNF0_PLEN_Pos) //对应llpkt的1字节preamble
                                
```

# 8. 关闭timer0的时机

发送完射频数据、接收完一帧射频数据、接收超时之后，都是关闭timer0的时机。

# 9. 总结

还有许多细节没关注到，总体问题不大，后面学习链路层再复习巩固。









