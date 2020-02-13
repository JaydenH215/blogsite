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

减少连接间隔是唯一提高BLE传输速率方法吗？ATT_MTU会影响传输速率吗？长包指的是什么？这篇文章就是解答这些问题的，有不对的地方，欢迎斧正。

由于影响吞吐量的因素有许多，而篇幅有限，以下条件先不考虑：
1. 加密数据包

加密数据包会影响LL PDU的长度。

2. att protocol以外的l2cap channel

比如smp这种对吞吐量没有需求的channel就不考虑。

3. link layer丢包重发

同一个LL packet发多遍当然会影响吞吐量，这里假设射频环境很好，不会有这个影响。

4. le coded phy

只考虑1M和2M

5. cpu处理速度的影响

假设cpu处理速度足够快，不会被其他中断影响协议栈的行为。

6. hci通讯接口的吞吐量限制

hci接口有uart/spi/ram等多种通讯方式，不同通讯方式本身也会有速率限制，这里不考虑这个限制。

7. 多链路

多链路分时复用radio，吞吐量会被降低，这里只考虑单链路情况。


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
- [5. LINK LAYER](#5-link-layer)
    - [5.1. Link layer packet](#51-link-layer-packet)
        - [5.1.1. ble4.0](#511-ble40)
        - [5.1.2. ble4.2](#512-ble42)
    - [5.2. BLE连接传输原理说明](#52-ble连接传输原理说明)
- [6. PHY LAYER](#6-phy-layer)
    - [1M PHY和2M PHY的差异](#1m-phy和2m-phy的差异)
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

gatt规定ATT_MTU默认支持不能小于23，ATT_MTU理论最大65535（因为MTU Exchange Request里面表示ATT_MTU大小的MTU size field只有2个字节）。

![](gatt_att_mtu.png)

gatt要求l2cap channel默认配置参数如下：

![](gatt_l2cap_config_param.png)

表格中MTU是一个默认值，表示此时此刻att protocol channel的L2CAP payload支持的最大传输单元是23字节，但并不代表local设备最大接收能力。连接之后可以通过exchange mtu procedure来更新ATT_MTU，所以协商ATT_MTU过程相当于协商MTU的过程，与BR/EDR的差异是BR/EDR通过l2cap channel configuration procedure来配置MTU，而不是exchange mtu procedure。

# 3. ATT PROTOCOL

## 3.1. Write Command format

![](write_command.png)

用于gatt的write without response。

## 3.2. Handle Value Notification format

![](notification.png)

用于gatt的notifications。

## 3.3. ATT_MTU的定义

ATT_MTU is defined as the maximum size of any packet sent between a client and a server. A higher layer specification defines the default ATT_MTU value. ATT_MTU is a negotiated value.

> 连接之后，client可以向server发起mtu exchange procedure，即client告诉server自己的最大接收能力ATT_MTU（client's local att protocol channel MTU），server也会告诉client自己的最大接收能力ATT_MTU（server's local att protocol channel MTU），最后取两者的最小值为这条通道的ATT_MTU（actual att protocol channel MTU）。

# 4. L2CAP SPEC

## 4.1. SDU(Service Data Unit)

来自上层的数据，不包含任何l2cap的信息，这里为ATT的消息。

## 4.2. B-frame

![](b_frame.png)

ATT PROTOCOL的数据就放在b-frame的information payload中，Length field占2个字节，所以理论payload最大只有65535字节，因为受芯片的ram大小限制，实际实现还会再小一点，通过MTU具体实现表现出来。

## 4.3. MTU(Maximum Transmission Unit), min 23

MTU is not a negotiated value, it is an informational parameter that each device can specify independently and indicates to the remote device that the local device can receive, in this channel, an MTU larger than the minimum required.

l2cap MTU表示每条l2cap channel的最大接收能力，不是一个协商值。

当l2cap channel是att protocol channel的时候，协议栈实现如下：

比如说，local设备的local att protocol channel MTU最大能支持到2048字节，但是peer设备的local att protocol channel MTU只能支持到23字节，经过gatt的mtu exchange procedure之后，取两者的最小值23为这条通道的ATT_MTU（即actual att protocol MTU）。尽管这时候att protocol作了长度限制不能超过23字节，但是peer设备要是发送了非法长度数据，att protocol本身是没有机制去检测出来的。所以在mtu exchange procedure之后协议栈实现就需要将local设备的att protocol channel MTU更新为23，那么当local设备收到一包att protocol消息之后，在l2cap层就可以判定该数据包是合法长度的数据了。

所以att protocol channel MTU也常常被误认为是ATT_MTU，但他们还是有点区别的。

## 4.4. MPS(Maximum PDU payload Size), max 65535

规范规定在Basic L2CAP mode的B-frame的MPS等于MTU。

Note: In the absence of segmentation, or in the Basic L2CAP Mode, the Maximum Transmission Unit is the equivalent to the Maximum PDU payload Size and shall be made equal in the configuration parameters.

## 4.5. Segmentation and Reassembly

![](l2cap_sdu_segmentation.png)

当MPS小于MTU的时候，需要segmentation；但是对于ATT消息, MPS总是等于ATT_MTU，所以ATT消息从不需要segmented，但是可能会因为链路层限制而需要fragmented。

经典蓝牙某些场合可能需要用到segmented，不太清楚。

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
上面10b、01b中的“b”表示二进制格式，中间的包和最后的包的LLID都是01b，对方怎么知道什么时候收完包呢？靠的是第一包的4字节L2CAP帧头，这里面会说明后面一共有多少数据待接收。

# 5. LINK LAYER

## 5.1. Link layer packet

### 5.1.1. ble4.0

![](ble40_ll_pkt.png)

可以看到LL PDU的长度最大为39字节，LL PDU又分为两种：一种是ADV PDU广播通道包，最大长度为39个字节；另一种是DATA PDU数据通道包，最大为33字节，后者是连接之后通讯的数据包。

![](ble40_ll_data_pdu.png)

展开DATA PDU，前2个字节为帧头，后4个字节为MIC（用于加密），就算不加密这4个字节也不能用于传输上层数据，即payload最多能容纳27字节的上层数据（上层指的是l2cap层）。

> The Length field of the Header indicates the length of the Payload and MIC if included. The length field has the range of 0 to 31 octets. The Payload field shall be less than or equal to 27 octets in length. The MIC is 4 octets in length.

### 5.1.2. ble4.2

后来因为27字节payload实在太少了，所以在ble4.2引入了新的特性，即LE data length extension（数据长度扩展，业内也叫长包）。

![](ble50_ll_pkt.png)

可以看到LL PDU的长度最大为257字节，LL PDU又分为两种：一种是ADV PDU广播通道包，最大长度为257个字节；另一种是DATA PDU数据通道包，最大为257字节，后者是连接之后通讯的数据包。

![](ble50_ll_data_pdu.png)

展开DATA PDU，前2个字节为帧头，后4个字节为MIC（用于加密），就算不加密这4个字节也不能用于传输上层数据，即payload最多能容纳251字节的上层数据（上层指的是l2cap层）。

> The Length field of the Header indicates the length of the Payload and MIC if included. The length field has the range of 0 to 255 octets. The Payload field shall be less than or equal to 251 octets in length. The MIC is 4 octets in length

## 5.2. BLE连接传输原理说明

![](ble_ll.png)

上图为 BLE 连接传输的一个例子，其中每个 Connection Event 包含 6 个主机和 6 个从机的 Link Layer Packet，一个 Connection Interval 只有一个 Connection Event；如果每个 Link Layer Pakcet 限制最多包含 20 字节用户数据，那么上图例子中每次 Connection Interval， BLE主机最多可以发送 120 字节用户数据， BLE 从机最多可以发送 120 字节用户数据。而这个
Connection Interval 就是我们常说的“连接间隔”。所以可知， BLE链路层传输速率的快慢主要由以下因素决定：

1. Connection Interval 的大小；
2. 每个 Connection Event 可以发送多少个 Link Layer Packet；
3. 每个 Link Layer Packet 可以负载多少用户数据；
4. 相邻 Link Layer Packet之间的时间间隔T_IFS。

# 6. PHY LAYER

![](phy.png)

上图为1M PHY和2M PHY在2404频点发送数据0b1010的对比图。

BLE的调制方式为GFSK，BLE定义在信道中传输的是二进制符号（波形），即1个符号（波形）表示1个比特，其中相对频点正偏代表数字基带信号1，相对频点负偏的代表数字基带信号0。

之所以速率提高了，本质应该是2M PHY的码长变小了，即发送一个符号（波形）所需的时间变小了。

> BLE规范原文：A binary one shall be represented by a positive frequency deviation, and a binary zero shall be represented by a negative frequency deviation.  
    > 1M PHY：
    The mandatory symbol rate is 1 megasymbol per second (Msym/s), where 1 symbol represents 1 bit therefore supporting a bit rate of 1 megabit per second (Mb/s), which is referred to as the LE 1M PHY  
    > 2M PHY：
    An optional symbol rate of 2 Msym/s may be supported, with a bit rate of 2 Mb/s, which is referred to as the LE 2M PHY. 

## 1M PHY和2M PHY的差异

- 1M PHY的正偏和负偏至少需要达到185Khz，2M PHY的正偏和负偏至少需要达到370Khz
- 2M PHY的比特率是1M PHY的两倍，即2M PHY传输完0b1010的时间是1M PHY的一半



# 7. 总结

提高吞吐量的途径有以下几种方式：

- GATT
    - client用write without response而不要用write characteristic value
    - server用notification而不要用indication
    - ATT_MTU尽可能大（这样可以减少L2CAP的帧头数量，通过Fragmentation来尽可能占据带宽，提高吞吐量）

- L2CAP
    - NULL（应用层没啥可以做的）

- LINK LAYER
    - 减少连接间隔（connection interval）
    - 增大连接事件（connection event）
    - 启用数据包扩展特性（link layer packet）

- PHY LAYER
    - 启用2M PHY特性（BLE5.0可选特性）


# 8. 参考资料

- 《RW-BLE-HOST-SW-FS_2mbps》
- 《Core_v5.0》
- nimble源码
- 《ZLG52810P0-1-TC透传模块速率参考手册 V1.01》
- [波特率和比特率及吞吐率三者概念](https://wenku.baidu.com/view/86a89ffbc67da26925c52cc58bd63186bceb92e5.html)
- [《通信原理》曹丽娜-西安电子科技大学-第一章、第六章](https://www.bilibili.com/video/av28491396/?p=27&t=40)
- [BLE 5 之 物理层](https://blog.csdn.net/ZQ07506149/article/details/85225731)
- 《RF-PHY.TS.5.0.3》