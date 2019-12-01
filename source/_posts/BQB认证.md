---
title: BQB认证
date: 2019-11-28 22:56:39
tags:
- BQB
- 认证

categories:
- 术业专攻

---

整理一下BQB认证相关资料，方便跟认证机构沟通。
<!-- more --> 

前言
===

大部分内容参考乐鑫公众号的一篇介绍BQB认证的文章。

目录
===

<!-- TOC -->

- [1. 简介](#1-简介)
- [2. 认证类型](#2-认证类型)
    - [2.1. component](#21-component)
    - [2.2. subsystem](#22-subsystem)
    - [2.3. end product](#23-end-product)
    - [2.4. 总结](#24-总结)
- [3. 认证流程](#3-认证流程)
    - [3.1. QDID](#31-qdid)
    - [3.2. DID](#32-did)
    - [3.3. 总结](#33-总结)
- [4. 认证方案](#4-认证方案)
    - [4.1. 做模块，以后需要被其他产品引用](#41-做模块以后需要被其他产品引用)
    - [4.2. 做产品，包含模块，无须layout rf部分](#42-做产品包含模块无须layout-rf部分)
    - [4.3. 做产品，包含芯片，重新layout rf部分](#43-做产品包含芯片重新layout-rf部分)
    - [4.4. 蓝牙dtm模式/信令模式](#44-蓝牙dtm模式信令模式)

<!-- /TOC -->

# 1. 简介

BQB全称bluetooth qualification body，如果产品希望在包装、广告以及整个营销过程中使用蓝牙文字标志及蓝牙徽标，产品都要通过BQB认证。

# 2. 认证类型

## 2.1. component

认证软件层面上的东西，不包含硬件。

- 最小认证单元，不能被DID直接引用
- 可以被subsystem和end product combine
- 如有需要，可以修改重新测试

实际认证案例：

- host（搜索QDID：122684）
- controller（搜索QDID：）
- linklayer（搜索QDID：122146）

## 2.2. subsystem

认证完整的系统，包含软件硬件。

- 一个完整的子系统，可以被DID直接引用
- 可以直接做测试，也可以combine已做过认证的component

实际认证案例：

- host subsystem（搜索QDID：119517）
- controller subsystem（搜索QDID：101395）
- profile subsystem（搜索QDID：）

## 2.3. end product

- 必须是一个完整系统（包括host+controller+硬件），可以被DID直接引用。
- 可以直接做测试，也可以combine已做过认证的component
- 如有需要可以修改测试项，重新测试

## 2.4. 总结

一个能够通过BQB认证的蓝牙设备肯定是以下其中一种：
- subsystem设备：reference各种subsystem，可能会combine各种component
- end product设备：直接测试end product，可能会combine各种component

# 3. 认证流程

BQB认证中有两类ID，一类称为QDID（Qualified Design ID），另一类称为DID（Declaration ID），QDID可以被DID引用。

## 3.1. QDID

获得这个ID需要经过测试机构一系列的测试。

## 3.2. DID

用钱买这个ID

## 3.3. 总结

一个能够通过认证的蓝牙设备肯定拥有至少一个QDID，一个DID，其中拥有QDID的方式可以是：
- reference旧的QDID
- 重新认证一个新的end product的QDID，然后reference这个新的QDID
- 重新认证一个新的subsystem的QDID，然后reference这个新的QDID

一个DID可以reference一个或多个QDID，一个QDID可以被一个或多个DID reference。

使用DID好处是可以使终端厂商的认证过程简单化。

# 4. 认证方案

## 4.1. 做模块，以后需要被其他产品引用

- 认证测试一个新的host component QDID（或者combine一个已过测试的host component的QDID）
- 认证测试一个新的controller/linklayer component QDID（或者combine一个已过测试的controller/linklayer component的QDID）
- 认证测试一个新的end product的QDID

## 4.2. 做产品，包含模块，无须layout rf部分

- 买一个DID
- reference模块的end product QDID

## 4.3. 做产品，包含芯片，重新layout rf部分

- 买一个DID
- 认证测试一个新的host component QDID（或者combine一个已过测试的host component的QDID）
- 认证测试一个新的controller/linklayer component QDID（或者combine一个已过测试的controller/linklayer component的QDID）
- 认证测试一个新的end product的QDID

## 4.4. 蓝牙dtm模式/信令模式

做BQB认证需要烧录dtm固件，然后用蓝牙综测仪接设备，进行测试。

dtm模式的蓝牙设备和蓝牙综测仪有hci和2-wire两种接口协议，需要确认dtm固件支持哪种协议，在蓝牙综测仪上面要选对配置。

测试机构说的信令模式一般是hci接口协议。