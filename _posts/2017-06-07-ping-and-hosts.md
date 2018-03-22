---
layout: post
title: Ping和hosts
tags:
- network
categories:
- code
comments: true
mathjax: true
date: 2017-06-07 16:50:00 +0800
---
修改hosts访问谷歌是一种简单常用的方法，不过一般都是从网上找别人提供的ip地址，如何自己找呢？

## 使用ping找ip地址
直接自己ping一般都是不行的，如果能ping通那就不用改hosts了。可使用[ipip](https://www.ipip.net/ping.php) 这个网站提供的工具，查询所需访问网址在世界各地ping到的ip，然后对这些ip地址自己在本地用ping测试一下，看哪些地址在国内是能用的。

## 修改hosts
将得到的ip地址和想访问的url加入hosts即可。如下两个目前自己测试可以用，访问时使用https即可。

```bash
216.58.199.238 encrypted.google.com
172.217.26.4 scholar.google.com
```