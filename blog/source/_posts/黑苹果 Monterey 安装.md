---
title: 黑苹果 Monterey 安装
date: 2021-12-23 23:46:06
tags:
- 日记
categories:
- 日记
---

为了实现自己安装黑苹果，特意将之前的 AMD 2600x 和 x470 主板换掉，更换了成了 Intel 的 i7-10700 CPU 和 b560m 主板。
参考国光的配置和博客，经过一周的学习和尝试，终于成功了安装了 Monterey 系统。
在此记录下遇到的问题以及解决办法。
<!-- more -->
### 配置 
我使用的配置如图：

| 配置 | 型号 |
| --- | --- | 
| CPU | Intel I7 10700 |
| 主板 | 微星 B560M 迫击炮（无 WiFi） |
| 内存 | 枭鲸 3200 16G * 2 |
| 显卡 | 蓝宝石 RX 590 8 GB 超白金极光特别版 |
| SSD | 三星 970 Evo 500G |
| 网卡 | 奋威 BCM94360CD 4 天线版本 |

配置是照着 [国光博客](https://www.sqlsec.com/2021/08/b560m.html) 进行的选择，但是由于我想安装 Monterey 系统，所以使用的 OpenCore 的版本是 0.7.6，和博客里的 EFI 版本不同，我直接使用出现了一些问题，所以我单独进行的配置。

[我配置的 EFI 地址](https://github.com/hGhostD/EFIConfig)

### 概览

成功安装 12.1
![](https://github.com/hGhostD/MarkDownPhotos/blob/master/%E9%BB%91%E8%8B%B9%E6%9E%9C/Pasted%20image%2020211228001312.png?raw=true)

#### Hackintool
设备读取正常
![](https://github.com/hGhostD/MarkDownPhotos/blob/master/%E9%BB%91%E8%8B%B9%E6%9E%9C/Pasted%20image%2020211228001421.png?raw=true)

驱动正常读取
![](https://github.com/hGhostD/MarkDownPhotos/blob/master/%E9%BB%91%E8%8B%B9%E6%9E%9C/Pasted%20image%2020211228001518.png?raw=true)

独显核显也成功读取，支持硬件加速
![](https://github.com/hGhostD/MarkDownPhotos/blob/master/%E9%BB%91%E8%8B%B9%E6%9E%9C/Pasted%20image%2020211228001557.png?raw=true)

USB 使用的是国光定制过的驱动，由于我和他使用的是同一品牌型号的主板，只是我的是无 WiFi 版本，所以我直接拿来使用了，经过测试，我前后背板的 USB 接口都能成功读取到，并且 Type-C 接口也可以正常工作。

遇到睡眠之后无法通过鼠标键盘唤起的问题，只能按一次电源开机键才能唤醒，觉得影响不大，就没有处理了。

### 遇到的问题
这是第一次完全自己动手安装黑苹果，所以遇到了很多问题，这里记录一下是如何解决的。