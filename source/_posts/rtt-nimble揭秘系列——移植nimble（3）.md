---
title: rtt+nimble揭秘系列——移植nimble（3）
date: 2019-10-09 00:24:44
tags:
- rtthread
- nimble

categories:
- 一得之见
---


本篇会谈一谈大家平常少接触的核心规范中关于ble有趣的部分，不深究，点到即止；同时，谈谈nimble的移植过程。
<!-- more --> 

前言
===

移植目标：手机app可以搜索到ble广播。

目录
===

<!-- TOC -->

- [1. ble核心规范简介](#1-ble核心规范简介)
    - [1.1. ble核心系统架构](#11-ble核心系统架构)
        - [1.1.1. host](#111-host)
        - [1.1.2. controller](#112-controller)
        - [1.1.3. hci](#113-hci)
    - [1.2. ble传输载体](#12-ble传输载体)
        - [1.2.1. le link layer到LE-C的signaling数据帧](#121-le-link-layer到le-c的signaling数据帧)
        - [1.2.2. le link layer到ADVB-C的signaling数据帧](#122-le-link-layer到advb-c的signaling数据帧)
        - [1.2.3. l2cap manager的signaling数据帧](#123-l2cap-manager的signaling数据帧)
        - [1.2.4. 更高层协议的signaling数据帧](#124-更高层协议的signaling数据帧)
        - [1.2.5. 可靠的异步用户数据帧](#125-可靠的异步用户数据帧)
        - [1.2.6. 不可靠的异步用户数据](#126-不可靠的异步用户数据)
        - [1.2.7. 举例](#127-举例)

<!-- /TOC -->


# 1. ble核心规范简介

下面从两个角度来总览ble，一个是从`协议层级划分`角度，即下文ble核心系统架构章节，另一个是`协议数据流向`角度，即下文ble传输载体章节。

> NOTE：运输载体英文原文是traffic bearers，有种小船之于水流的感觉，我不知道如何翻译才贴切，欢迎各位提建议。

## 1.1. ble核心系统架构

下图为ble核心系统架构（截取自蓝牙核心规范，将低功耗蓝牙部分PS了出来），ble核心架构可划分为两部分，分别是host和controller。

![](ble_core_architecture.png)

### 1.1.1. host
- gatt
- att
- gap
- smp
- l2cap

### 1.1.2. controller
- device manager
- link manager
- baseband resource manager
- link controller
- phy

### 1.1.3. hci
host和controller可通过hci（host-controller-interface）接口互相通信。具体表现为：
- host通过hci向controller发送command
- controller通过hci向host发送event
- host和controller通过hci互传acl data

## 1.2. ble传输载体

下图为ble传输载体示意图（截取自蓝牙核心规范，将低功耗蓝牙部分PS了出来），参考上一节左边的系统架构，`Application`表示host中除l2cap以外部分的集合，`Bluetootch core`表示l2cap和整个controller。

![](ble_core_traffic_bearers.png)


结合上图，ble协议中流动的所有数据帧如下，即使分包也是由以下部分帧拆分而来：

### 1.2.1. le link layer到LE-C的signaling数据帧
- 如ll control pdu的LL_FEATURE_REQ

### 1.2.2. le link layer到ADVB-C的signaling数据帧
- 如scanning pdu的SCAN_REQ
- 如initiating pdu的CONNECT_IND

### 1.2.3. l2cap manager的signaling数据帧
- 如l2cap的connection parameter update request

### 1.2.4. 更高层协议的signaling数据帧
- 如smp的pairing request

### 1.2.5. 可靠的异步用户数据帧
- 如gatt的write without response，其中att_payload包含用户数据
    
### 1.2.6. 不可靠的异步用户数据
- 如advertising的ADV_IND，其中adv_payload包含ad type格式的用户数据

### 1.2.7. 举例
针对上述帧举一个例子详细说明，例子中有个角色，一个是嵌入式蓝牙设备，称为device，另一个是手机，称为app，如图所示从左到右为角色的每一层，从上到下为时间线。

！！！！此处应该有一张大图片，后面补！！！！！

- 双方处于未连接状态

    1. 某个时刻device's gap通过hci发送command，让device's controller开始发送ADV_IND，即开始广播，其中ADV_IND中包含device的设备名。
    2. 某个时刻app's gap通过hci发送command，让app's controller开始发送SCAN_REQ，即开始扫描周边蓝牙设备。
    3. app's controller发现了正在广播的device，随后通知app's gap，紧接着app's gap通过hci发送command，让app's controller开始发送CONNECT_IND，即发起连接请求

- 双方刚刚建立连接状态
    
    1. app's controller向device 's controller发送LL_FEATURE_REQ，希望知道刚刚连上的device支持哪些特性。

- 双方已经连接了一段时间状态

    1. 连接一段时间后，device觉得频繁与app通信影响功耗，所以device's l2cap manager将connection parameter update request封装成acl data，通过hci发送给device's controller，device's controller随即将数据发给app's l2cap manager，即连接参数更新请求，该请求要求加大连接间隔。
    2. 某个时刻app向发送用户数据给device，app's gatt将write without response发送给l2cap，l2cap将write without response封装成acl data通过hci发送给app's controller，让app's controller开始发送write without respon给device's gatt。
    3. 某个时刻app觉得传输明文用户数据太危险，即app's smp发起了pairing request给l2cap，l2cap将pairing request封装成acl data通过hci发送给app's controller，让app's controller开始发送pairing request给device's smp。


