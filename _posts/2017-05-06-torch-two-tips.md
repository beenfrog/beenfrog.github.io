---
layout: post
title: Torch小细节两则
tags:
- deep learning
- torch
categories:
- code
comments: true
mathjax: true
date: 2017-05-06 16:00:00 +0800
---
## permute引起的错误
在torch中读取图像自己一般使用[torch-opencv](https://github.com/VisionLabs/torch-opencv),由于opencv读取的图像是$H \times W \times 3$，而网络中一般是$batch \times 3 \times H \times W$的顺序，故而经常使用permute进行转换。而这个操作并未进行数据的复制，最好接着使用clone，以防出现未知的错误，这也是长久以来的经验教训。今天又遇到这样错误，只因clone位置不对，又找了好久才发现。

```lua
-- wrong
local input = cv.resize{frame, {w,h}}:permute(3,1,2):resize(1,3,h,w):cuda():clone()
-- right
local input = cv.resize{frame, {w,h}}:permute(3,1,2):clone():resize(1,3,h,w):cuda()

```

## CrossEntropyCriterion的折腾
自己的系统Ubuntu 12.04比较老了，只安装了cudnn R2，好多最新的cudnn功能都无法使用。最近使用全卷积网络做分割小实验，发现nn.CrossEntropyCriterion不支持pixel-level的损失计算，cudnn中有SpatialCrossEntropyCriterion，而自己的老cudnn不支持。一时还不想折腾系统升级，故而折腾了一下nn.CrossEntropyCriterion，也能完成自己的目的。其实也简单，多加一个变形的网络，也就是transpose和reshape的组合，代码如下，将rnet拆开有利于训练好后net的部署使用。

```lua
net = define_segnet()
cri = nn.CrossEntropyCriterion()

rnet = nn.Sequential()
rnet:add(nn.Transpose({2,3},{3,4}))
rnet:add(nn.Reshape(4*240*320,5, false))

...
local output   = net:forward(batch_data)
local output_r = rnet:forward(output)
local err      = cri:forward(output_r, batch_label)
local df_do   = cri:backward(output_r, batch_label)
local df_dor  = rnet:backward(output, df_do)
net:backward(batch_data, df_dor)

```