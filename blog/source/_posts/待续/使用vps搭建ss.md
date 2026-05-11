---
layout: 待续
title: 使用vps搭建ss
date: 2017-11-04 15:59:15
tags: 
- 待续
categories: 
- 待续
---
#### 使用 VPS 搭建 SS
由于之前的 VPN 到期了，刚好想自己搞一个 VPS，初步搭建记录一下过程。
<!-- more -->
#### 购买 VPS 服务器
登录 [Vultr](https://my.vultr.com/) 官网选择需要的服务器。我选择日本东京的服务器。服务器类型选择 CentOS 7 x64 ![CentOS 7 x64](https://raw.githubusercontent.com/hGhostD/MarkDownPhotos/master/VPS/ServerType.png)等服务器启动后可以获取相关信息，查看 IP 地址和密码![](https://raw.githubusercontent.com/hGhostD/MarkDownPhotos/master/VPS/Account.jpeg)
#### 配置服务器
服务搭建完成后需要连接服务器，mac 用户可以直接在终端 使用 `ssh root@XX.XX.XX.XXX` 指令进行连接。windows 可以下载 [XShell](https://nofile.io/f/eb5dUzYMQK4/Xshell_setup_wm.exe) 提取密码: 666。
连接之后输入密码，遍进入服务器终端。

使用 root 用户登录，运行一下命令:
```
wget --no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
chmod +x shadowsocks.sh
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```
按照提示命令，输入密码、端口号、加密方式(aes-256-cfb)等信息
```
Congratulations, Shadowsocks-python server install completed!
Your Server IP        : // 服务器 IP
Your Server Port      : // 刚才设置的端口号
Your Password         : // 刚才设置的密码
Your Encryption Method:your_encryption_method

Welcome to visit:https://teddysun.com/342.html
Enjoy it!
```
到此初步配置已经结束，最后需要重启一下服务器。
以后可能还需要用 VPS 做一些更有难度的工作，不过第一步通过 VPS 翻墙总算是大功告成了！！！

<br>
>参考资料
>[vultr](https://my.vultr.com/)
>[自建ss服务器教程](https://github.com/Alvin9999/new-pac/wiki/%E8%87%AA%E5%BB%BAss%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%95%99%E7%A8%8B)
>[Shadowsocks Python版一键安装脚本](https://teddysun.com/342.html)