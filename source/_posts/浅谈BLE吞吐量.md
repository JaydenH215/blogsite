---
title: 浅谈BLE吞吐量
date: 2020-01-02 18:28:06
tags:
- ble

categories:
- 术业专攻
---

怎样才能提高BLE吞吐量？一文读懂BLE的吞吐量。
<!-- more --> 

前言
===

减少连接间隔是唯一提高BLE传输速率方法吗？ATT_MTU会影响传输速率吗？长包指的是什么？这篇文章就是解答这些问题的。

由于影响吞吐量的因素有许多，而篇幅有限，以下条件先不考虑：
1. 加密数据包
2. att protocol以外的l2cap channel
3. link layer丢包重发
4. le coded phy
5. cpu处理速度的影响
6. radio ramp-up的影响
7. hci通讯接口的吞吐量限制
8. 多链路

目录
===

<!-- TOC -->

- [1. BLE协议栈框架](#1-ble协议栈框架)
- [2. GATT](#2-gatt)
- [3. ATT PROTOCOL](#3-att-protocol)
    - [3.1. Write Command format](#31-write-command-format)
    - [3.2. Handle Value Notification format](#32-handle-value-notification-format)
    - [3.3. ATT_MTU的定义](#33-att_mtu的定义)
- [4. L2CAP SPEC](#4-l2cap-spec)
    - [4.1. SDU(Service Data Unit)](#41-sduservice-data-unit)
    - [4.2. B-frame](#42-b-frame)
    - [4.3. MTU(Maximum Transmission Unit), min 23](#43-mtumaximum-transmission-unit-min-23)
    - [4.4. MPS(Maximum PDU payload Size), max 65535](#44-mpsmaximum-pdu-payload-size-max-65535)
    - [4.5. Segmentation and Reassembly](#45-segmentation-and-reassembly)
    - [4.6. Fragmentation and Recombination](#46-fragmentation-and-recombination)
- [5. LINK LAYER（未完）](#5-link-layer未完)
- [6. PHY LAYER（未完）](#6-phy-layer未完)
- [7. 总结](#7-总结)
- [8. 参考资料](#8-参考资料)

<!-- /TOC -->

# 1. BLE协议栈框架

![](arch.png)

手机下传write without repsonse的一个procedure数据流向：

    gatt(client) -> l2cap -> hci(option) -> link layer -> phy -> phy -> link layer -> hci(option) -> l2cap -> gatt(server)

设备上传notication的一个procedure数据流向：

    gatt(server) -> l2cap -> hci(option) -> link layer -> phy -> phy -> link layer -> hci(option) -> l2cap -> gatt(client)

# 2. GATT

gatt规定ATT_MTU默认支持不能小于23，ATT_MTU理论最大65535（MTU Exchange Request里面MTU size field只有2个字节），但是实际最大值由l2cap实现限制，l2cap的payload边界受ic资源和协议栈实现影响。

![](gatt_att_mtu.png)

gatt要求l2cap channel默认配置参数如下：

![](gatt_l2cap_config_param.png)

这里MTU是一个默认值，表示此时此刻支持的最大传输单元，并不代表local设备最大只支持到这个值。连接之后可以通过exchange mtu procedure来更新ATT_MTU，而ATT bearer就是该l2cap channel，所以协商ATT_MTU过程相当于协商MTU的过程，与BR/EDR的差异是BR/EDR通过l2cap channel configuration procedure来配置MTU的。

# 3. ATT PROTOCOL

## 3.1. Write Command format

![](write_command.png)

用于gatt的write without response。

## 3.2. Handle Value Notification format

![](notification.png)

用于gatt的notifications。

## 3.3. ATT_MTU的定义

ATT_MTU is defined as the maximum size of any packet sent between a client and a server. A higher layer specification defines the default ATT_MTU value. ATT_MTU is a negotiated value.

# 4. L2CAP SPEC

## 4.1. SDU(Service Data Unit)

来自上层的数据，不包含任何l2cap的信息。

## 4.2. B-frame

![](b_frame.png)

ATT PROTOCOL的数据就放在b-frame的information payload中，Length field占2个字节，所以理论payload最大只有65535字节，因为资源限制，实际实现还会再小一点，通过MTU表现出来。

## 4.3. MTU(Maximum Transmission Unit), min 23

MTU is not a negotiated value, it is an informational parameter that each device can specify independently and indicates to the remote device that the local device can receive, in this channel, an MTU larger than the minimum required.

l2cap MTU表示local device的最大接收能力，不是一个协商值。（通过查阅协议栈实现代码，我觉得MTU有两个意思，一个是local MTU，另一个是actual MTU，该段话应该想表示前者，后者表示ATT_MTU，是协商出来的。）

## 4.4. MPS(Maximum PDU payload Size), max 65535

规范规定在Basic L2CAP mode的B-frame的MPS等于MTU。

Note: In the absence of segmentation, or in the Basic L2CAP Mode, the Maximum Transmission Unit is the equivalent to the Maximum PDU payload Size and shall be made equal in the configuration parameters.

## 4.5. Segmentation and Reassembly

![](l2cap_sdu_segmentation.png)

当MPS小于MTU的时候，需要segmentation；但是对于ATT消息, MPS总是等于ATT_MTU，所以ATT消息从不需要segmented，但是可能会因为链路层限制而fragmented。

## 4.6. Fragmentation and Recombination

![](l2cap_fragmentation.png)

由于链路层的接收限制，l2cap的帧有时候需要拆开发送，用链路层包头的LLID来负责标识拆包和重组。

![](LLID.png)

下面看两个例子。

如果BLE4.0，ATT_MTU最后client和server协商出来结果是23，那么发送20字节（ATT_payload）不需要拆包：

    1字节前导码 + 4字节访问地址 + 2字节链路帧头（其中LLID = 10b，表示一个没有fragmentation的完整包） + 4字节L2CAP帧头 + 3字节ATT帧头 + 20字节的ATT_payload + 3字节CRC

如果BLE4.0，ATT_MTU最后client和server协商出来假设是63，那么你发送60字节数据（ATT_payload），L2CAP会将它拆成3包（拆包的原因是因为BLE4.0链路层PDU的限制），最终从空中发出去的包长这样：

    1）1字节前导码 + 4字节访问地址 + 2字节链路帧头（其中LLID = 10b，表示这是第一个连续包） + 4字节L2CAP帧头 + 3字节ATT帧头 + 20字节的ATT_payload + 3字节CRC
    2）1字节前导码 + 4字节访问地址 + 2字节链路帧头（其中LLID = 01b，表示这是下一包连续包） + 27字节的ATT_payload + 3字节CRC
    3）1字节前导码 + 4字节访问地址 + 2字节链路帧头（其中LLID = 01b，表示这是最后一个连续包） + 13字节的ATT_payload + 3字节CRC
上面10b、01b中的“b”表示二进制格式，中间的包和最后的包都的LLID都是01b，对方怎么知道什么时候收完包呢？靠的是第一包的4字节L2CAP帧头，这里面会说明后面一共有多少数据待接收。

# 5. LINK LAYER（未完）


- Link layer packet
- PDU
    - LL Data PDU
    - LL Control PDU

- Connection event
- Connection interval
- T_IFS
- LE data length extension

update data channel PDU payload length



# 6. PHY LAYER（未完）

- 1M PHY

The mandatory symbol rate is 1 megasymbol per second (Msym/s), where 1 symbol represents 1 bit therefore supporting a bit rate of 1 megabit per second (Mb/s), which is referred to as the LE 1M PHY

- 2M PHY

An optional symbol rate of 2 Msym/s may be supported, with a bit rate of 2 Mb/s, which is referred to as the LE 2M PHY. 


# 7. 总结


# 8. 参考资料

- 《RW-BLE-HOST-SW-FS_2mbps》
- 《Core_v5.0》
- nimble源码