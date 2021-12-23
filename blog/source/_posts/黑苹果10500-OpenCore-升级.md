---
layout:
  - 黑苹果安装
title: 黑苹果10500 OpenCore 升级
date: 2021-12-22 14:02:40
tags:
- 日记
categories:
- 日记
---

Monterey 出来也已经有一段时间了，这次决定手动升级一下。
之前是在淘宝花费 200 元进行安装的 Big Sur，使用的 OpenCore 也是 0.6.3 版本。虽然可以升级到 11.6.1 版本，但是无法继续升级到 Monterey 了，必须要更新 OpenCore 重新安装了。

<!-- more -->

之前的准备工作就不再赘述，准备好了 U 盘以及镜像文件。重点在更新一下 EFI 文件。
### 配置 
我使用的配置如图：
| 配置 | 型号 |
| ---- | ---- | 
| CPU | Intel I5 10500 |
| 主板 | [Asrock B460m-ITX/AC](https://www.asrock.com/mb/Intel/B460M-ITXac/) |
| 内存 | 枭鲸 2666 16G * 2 |
| 显卡 | Radeon RX 580 4 GB |
| SSD | 铠侠 RC10 500G |
| 网卡 | 奋威 BCM94352Z |

之前也是按照黑果小兵的长期维护配置清单中选择的配置进行购买，所以现在这套配置的 EFI 还是有人进行维护的。
EFI 参照地址：**https://github.com/ansonliao/Asrock-B460m-ITX-AC-OC-EFI**

### 替换过程
他的这套配置使用的是核显进行显示，处理了一下 dp 接口会黑屏的问题。虽然我使用了免驱独立显卡，但是这个补丁我还是保留了。
之前在黑果小兵论坛上看到安装 Monterey 存在蓝牙的问题，需要增加驱动补丁。所以我直接也加上了。

> Mac OS monterey 第三方非白果卡蓝牙解决方法 
> 使用“每日构建版的BrcmPatchRAM驱动” 
> 注：BCM和英特尔都需要 
> 注：Z3和Z4只需要BlueToolFixup 
> 设置： 
> BlueToolFixup最低内核限制”21.0.0“ 
> 设置： 
> BCM的“BrcmBluetoothInjector” 
> 英特尔的“IntelBluetoothInjector” 
> 最高内核限制”20.99.99“ 
> 每日构建版下载链接：https://dortania.github.io/builds/ 
> 非白果卡 12蓝牙关了打不开 
> sudo pkill bluetoothd 
> 终端来一次 
> 12蓝牙关了打不开大神已经发pr了等待通过 
> https://github.com/acidanthera/BrcmPat> chRAM/pull/18
> 下载之后重新生成一下机器的三码并保存。

### 遇到的问题
在安装过程中遇到了图中问题。
![](https://github.com/hGhostD/MarkDownPhotos/blob/master/%E9%BB%91%E8%8B%B9%E6%9E%9C/WechatIMG103.jpeg?raw=true)
开始以为是 USB 定制的问题，尝试自己修改 USB 端口驱动文件。在安装的过程中突然想起来可能是前置 USB 端口的问题，之前查到过资料说安装黑苹果时，**不能使用 USB 3.0 接口，需要使用 USB 2.0 接口进行安装。** 又更换到后面板重新进行安装，顺利进入到了恢复界面。由于之前已经成功安装过黑苹果系统，所以不用再对磁盘进行抹除，选择升级的磁盘，直接进行系统升级。
期间重启了 3 次，由于我的 windows 引导被我修改过。需要每次手动从 BIOS 选择 U 盘启动，这个问题在升级成功后重新再 windows 处理。

### 概览

成功安装 12.1
![](https://github.com/hGhostD/MarkDownPhotos/blob/master/%E9%BB%91%E8%8B%B9%E6%9E%9C/Snipaste_2021-12-23_15-31-14.jpg?raw=true)
#### Hackintool

设备正常读取
![](https://github.com/hGhostD/MarkDownPhotos/blob/master/%E9%BB%91%E8%8B%B9%E6%9E%9C/Snipaste_2021-12-23_15-33-08.jpg?raw=true)

CPU 相关参数读取正常
![](https://github.com/hGhostD/MarkDownPhotos/blob/master/%E9%BB%91%E8%8B%B9%E6%9E%9C/Snipaste_2021-12-23_15-50-23.jpg?raw=true)

驱动正常读取
![驱动读取](https://github.com/hGhostD/MarkDownPhotos/blob/master/%E9%BB%91%E8%8B%B9%E6%9E%9C/Snipaste_2021-12-23_16-06-39.jpg?raw=true)

独显核显也成功读取
![双硬解](https://github.com/hGhostD/MarkDownPhotos/blob/master/%E9%BB%91%E8%8B%B9%E6%9E%9C/Snipaste_2021-12-23_16-07-47.jpg?raw=true)

### 遇到的问题
1. 第一次升级过后发现读取 OpenCore 版本不正确，没有及时更新，在晚上找到先关解决办法。成功更新

	Hackintool 显示 OpenCore 不正确修正
	https://blog.51cto.com/systemhuiyi/2716675
	如何显示 reset nvram？
	https://macefi.com/course/846.html

2. 后来关机的时候也发现卡住了，无法正常关闭，重启没有问题。后来用下面这个方法进行尝试，可以正常关机。
>  https://blog.csdn.net/fjh1997/article/details/104722014

3. USB 前面板只有一个可以正常使用。
	我使用的机箱前置 USB 接口有两个，但是只有一个可以使用。USB 定制过也依旧没有解决问题。不过之前的 EFI 文件有重新插 U 盘会导致电脑重启的问题，经过这次定制已经得到了解决。