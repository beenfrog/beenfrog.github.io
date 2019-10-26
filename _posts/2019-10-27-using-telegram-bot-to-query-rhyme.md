
---
layout: post
title: 利用telegram bot查询同韵汉字
tags:
- python
- telegram
- software
categories:
- code
comments: true
mathjax: false
date: 2019-10-27 01:47:05 +0800
---
在日常编顺口溜和打油诗的时候，查询同韵的汉字是不可或缺的任务，熟练的人可以直接想出相关的同韵字，而不熟练的人通过查询同韵字典可从中获取许多灵感，相关的查询网站很多，这里自己利用telegram的bot来实现这一功能，主要用于bot编程的实践学习。

## 代码与演示
+ 演示[TongYunBot](https://t.me/TongYunBot)
+ 代码[Github](https://github.com/beenfrog/TongYunBot)

## 实现细节
+ telegram bot的框架部分比较简单，安装了[python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot/)之后其中自带的[echobot2.py](https://github.com/python-telegram-bot/python-telegram-bot/blob/master/examples/echobot2.py)就有接受用户输入文字然后将原文返回的功能，在此基础上只要实现一个能完成从输入到自己想要的输出的模块即可。
+ 核心的模块定义了一个类`Yun`，初始化时读取韵表存储在dict中，并对同韵的利用汉字常用字的词频进行排序。 然后利用输入查询其同韵的词并返回即可。
+ 实现上原同韵字典u和ü没有分开，这俩个很难同韵，故而手动将其分开。 完成这一任务过程中才发觉我们一般常用的汉字太少，不认识的汉字太多。
+ 为了更顺，只查询同声调的字，没有考虑平仄。
+ 为了支持能在python2/3中运行，且支持汉字的读取和输出，接触和学习了之前没碰到的东西，面向谷歌编程。
+ 处理python2/3兼容以及汉字相关的片段，摘自[yun.py](https://github.com/beenfrog/TongYunBot/blob/master/yun.py)
```python
import sys
if sys.version_info[0] < 3:
	reload(sys)
	sys.setdefaultencoding('utf8')
else:
	from functools import cmp_to_key


with io.open(self.yunbiao_filename, 'r', encoding='UTF-8') as fid:
	for line in fid:


if sys.version_info[0] < 3:
	newlist = sorted(oldlist, cmp = cmp_hanzi)
else:
	newlist = sorted(oldlist, key = cmp_to_key(cmp_hanzi))
```
## 效果图
![bot demo](/assets/img/bot.jpg )
