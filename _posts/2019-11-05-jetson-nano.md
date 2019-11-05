---
layout: post
title: Jetson Nano 折腾记
tags:
- software
- python
- linux
- pytorch
- hardware
- network
categories:
- research
comments: true
mathjax: true
date: 2019-11-05 09:14:00 +0800
---
最近弄到一个Jetson Nano, 好久没碰硬件了编程了，玩一玩，再次记录一下折腾的过程。

## 硬件安装
+ 按照官方的教程来比较容易，见[Getting Started With Jetson Nano Developer Kit](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit), 期间自己遇到的问题有两：一、最初下载的镜像有问题，导致安装出错，重下后解决；二、利用转接头显示器无法显示，后来换了显示器解决。

## 基础软件

+ 该系统是`ubuntu 18.04`，和折腾一般系统一样，安装一些常用软件。在之前，配置好`SecureCRT`和`FileZilla`以方便数据传输和管理，之后安装代理软件。
+ 系统里面自带chrome，安装插件就是个难题，只能首先在终端里面`chromium-browser --proxy-server="socks5://127.0.0.1:10808"`开启代理，然后就可以打开[Proxy SwitchyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif)并安装插件了。
+ 安装`proxychains4`，使终端也能用到代理，方法见[proxychains](https://research.beenfrog.com/blog/2018/06/14/some-apps-and-services.html#proxychains).
+ 测试了一下，排除网速的影响，nano看1080P不卡，还是很厉害的。
+ `Sublime Text` 用不了，真是可惜
## 开发软件
