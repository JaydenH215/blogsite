---
title: 调试利器——axf文件
date: 2019-11-07 13:43:53
tags:
- cortex-m
- 内核
- 调试
- debug

categories:
- 术业专攻

---

-
<!-- more --> 

前言
===

未完。

目录
===

<!-- TOC -->

- [1. 反汇编](#1-反汇编)
- [参考资料](#参考资料)

<!-- /TOC -->

# 1. 反汇编

- objdump

arm-none-eabi-objdump -S -d xxx.axf > xxx.txt

> NOTE：需要保证编译工程为英文路径，否则编译器添加源码路径等信息进axf文件有可能路径不对，使得生成的txt文档没有包含源码。`arm-none-eabi-objdump -h`可以用来查看是否包含调试信息，如果没有需要在编译时候加入`-g`选项。

- fromelf

fromelf -c xxx.axf -o xxx.txt

暂时还不知道怎么带源码。

# 参考资料
- [objdump(Linux)反汇编命令使用指南](https://blog.csdn.net/wwchao2012/article/details/79980514)