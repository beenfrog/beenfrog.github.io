---
layout: post
title: LaTeX插入高版本pdf图片时的问题一则
tags:
- latex
categories:
- code
comments: true
mathjax: false
date: 2017-07-06 17:28:32 +0800
---
在用LaTeX写东西时，需要插入流程图，就直接使用Linux下的wps制作，然后转换为pdf。图弄好后使用XeLaTeX编译发现图片无法显示，试了一下转为png是没问题的，难道subfigure不支持pdf，然后尝试了minipage方式插图，还是不行。之后找了其他的pdf图片试了试，可以，原来wps保存的pdf版本是1.7，而之前用Office保存的版本是1.4，估计是版本太高了，怎么转换呢。在wps中没找到解决方案，搜了搜，用如下的gs命令即可。好大的坑，花了好长的时间才解决这个问题。

```bash
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -o output.pdf  input.pdf
```
