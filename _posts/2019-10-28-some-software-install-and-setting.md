---
layout: post
title: 常用软件的安装与配置
tags:
- software
- python
- linux
- network
categories:
- code
comments: true
mathjax: true
date: 2019-10-28 18:37:00 +0800
---
记录一下常用软件的安装与配置

## Ubuntu
系统的常用设置

### 将home下的文件夹名改为英文
+ 终端中输入如下，并选择同意改名
```bash
export LANG=en_US
xdg-user-dirs-gtk-update
```
+ 之后再在终端输入
```bash
export LANG=zh_CN
```
+ 重启电脑后，会提示时候改回成中文，选择不改即可

### 可视化硬件信息
```bash
sudo apt-get install hardinfo
```

### LXDE的双屏幕设置
```bash
sudo apt-get install arandr
```

## Firefox
常用插件如：
+ Adblock Plus
+ GitLab Markdown Viewer
+ Proxy SwitchyOmega

## Sublime Text 3
常用的扩展包以及相关的配置如下

### 自身设置
**key bindingd** 中设置
```json
[
	{ "keys": ["ctrl+alt+z"], "command": "cancel_build" }
]
```

### PackageResourceViewer
可以用来更改默认的的包内设置，尤其是修改默认的build系统

### Terminus
可在Sublime中打开终端，使用方式`crtl+shift+p`中搜索`Terminus: `根据提示选择. 设置默认终端可参考：
```json
{
    "shell_configs": [
        {
            "name": "PowerShell",
            "cmd": "powershell.exe",
            "env": {},
            "enable": true,
            "default": true,
            "platforms": ["windows"]
        }
    ]
}
```

### ConvertToUTF8
处理中文乱码问题

### Anaconda
这个是插件Anaconda，结合系统安装的Anaconda可将Sublime打造成优良的Python开发环境.新建一个`build system`,填写如下
```json
{
    "cmd": ["C:\\Users\\Administrator\\Anaconda3\\python.exe", "-u", "$file"],
    "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
    "selector": "source.python",
    "encoding": "utf-8" ,
    "env": {"PYTHONIOENCODING": "utf8"}
}
```
Anaconda自身的设置如下：
```json
{
	"python_interpreter": "C:\\Users\\Administrator\\Anaconda3\\python.exe",
	"anaconda_linting": false
}
```

### 错误处理
若开启后出现`unable to download XXXX. please view the console for more details`,这里`XXXX`是已安装的包，这时到`Preferences\setting-user`中将`ignored_packages`中被禁的包移除，同时到`Preferences\Package Settings\Package Control\Settings-User`添加如下的内容。主要的原因是连接网络的小工具有问题。
```json
"debug": true,
"downloader_precedence":
{
	"linux": ["curl","urllib","wget"],
	"osx": ["curl","urllib"],
	"windows": ["wininet"]
},
```

#### 待续
`to be continued`

## Anaconda

安装后添加系统的path如下：
```bash
C:\Users\Administrator\Anaconda3
C:\Users\Administrator\Anaconda3\Scripts
C:\Users\Administrator\Anaconda3\Library\bin
```

## Synergy
用于多台机器之间共享鼠标、键盘、剪贴板。只要在一台机器上安装了鼠标键盘，就可以在其他机器上使用，对于桌面有多台机器来说非常的方便。例如一台桌面Linux，一台Windows，一台嵌入式的Nano系统，这样只要一套键鼠就可以操作所有机器了，减少了各种切换以及对桌面的空间需求。
安装过程，首先对于Nano，使用命令安装如下：
```bash
sudo apt-get install synergy
```
当前安装的版本为1.8.8，之后在[sourceforge](https://sourceforge.net/projects/synergy-stable-builds/files/v1.8.8-stable/) 上下载对应的Win下面的安装包，同样针对桌面Linux也下载对应的deb安装包。三个均安装好，设置一台机器为服务器，也即共享键鼠的机器，记住该机器ip。之后进入其他的机器，设置为客户端，填写服务器的ip，记住该机器的屏幕名，然后再进入有键鼠的那台机器，点击`设置服务器`,添加各个客户端的屏幕，且命名为各机器的屏幕名，并拖动摆放好各个屏幕的相对关系，这样就完成了设置。除了能共享键鼠外，剪贴板里面的文字内容也可以共享的。而文件拖拽复制功能Linux下不支持还是有点遗憾。

## Win 10双网卡设置
将两个网线接好后，可在`控制面板\所有控制面板项\网络连接`里面将一个网卡只启用ipv4另一个只启用ipv6,这样就可以各取所需了。

## Rufus 制作U盘启动镜像
[Rufus](https://rufus.ie/)简洁好用。



