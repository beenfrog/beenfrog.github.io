---
layout: post
title: 从github网页上直接发布一篇博客
tags:
- blog
- github
categories:
- blog
comments: true
mathjax: true
date: 2018-06-12 23:45:16 +0800
---
这个github上的文章一般是在本地编写好，通过git提交到github上，这篇博客试试直接从网页上编写发布一篇文章。操作也挺简单的，先到定位到仓库中的_posts文件夹下，新建一个新文件就可以直接在上面编写了。实际体验还不错，注意要将右上角**Line wrap model**选为**Soft warp**

## 贴一段代码
在`.bashrc`中添加如下环境变量，可使终端里面的命令也用上ss代理，`git clone`体验上有百倍加速。

```bash
export ALL_PROXY=socks5://127.0.0.1:1080
```

## 贴一张图片
![Wall](/assets/img/wall.jpg)

## 贴一个链接
[Google](https://www.google.com)

## 贴一个公式
$$ x_{1,2} = \frac{-b \pm \sqrt{b^2-4ac}}{2a}$$

## 贴一个列表
+ **强调**
+ **指出**
+ **提出**
