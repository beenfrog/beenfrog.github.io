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

#### 待续
`to be continued`

## Anaconda

安装后添加系统的path如下：
```bash
C:\Users\Administrator\Anaconda3
C:\Users\Administrator\Anaconda3\Scripts
C:\Users\Administrator\Anaconda3\Library\bin
```
