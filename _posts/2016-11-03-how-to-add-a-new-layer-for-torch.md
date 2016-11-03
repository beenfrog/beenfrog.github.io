---
layout: post
title: 如何在torch中添加新的层？
tags:
- deep learning
- torch
- lua
- c/c++
- chinese
categories:
- research
comments: true
mathjax: true
date: 2016-11-03 21:01:46 +0800
---
Torch本身提供了许多常用的层(模块)，但有时需要自己写一些系统没有提供的层，怎么写呢，官方提供了一些[说明](http://torch.ch/docs/developer-docs.html)，一般分为两种情况：

1. 所有运算均能通过Tensor自带的操作来完成，这样只要写一个lua文件就行，然后直接require就可以使用了，非常方便。
2. 使用Tensor操作无法完成或者效率太低时，就需要使用C和Cuda来实现核心算法，然后用lua来调用，这个稍微复杂一些。


## 基本结构
无论哪一种情况，一个关键的文件是新模块的lua文件(NewLayer.lua)，其一般模板如下:

```lua
local NewLayer, Parent = torch.class('nn.NewLayer', 'nn.Module')

function NewLayer:__init()
   Parent.__init(self)
end

function NewLayer:updateOutput(input)
end

function NewLayer:updateGradInput(input, gradOutput)
end

function NewLayer:accGradParameters(input, gradOutput)
end
```

第一行定义了新的层类，并继承自nn.Module（顺带提一下，一般的层继承自nn.Module，损失层继承自nn.Criterion。lua语言本身没有直接提供面向对象编程功能，但可以通过闭包、元表等方式实现，而torch本身提供了面向对象的功能，通过torch.class来实现的）。定义的新层一般有四个主要的成员函数：

1. \_\_init()，初始化操作，如我们常用nn.Linear(3, 4)，表示输入3个元素，输出4个元素，其初始化函数形式为 function Linear:\_\_init(inputSize, outputSize, bias)，其中bias表示有无偏置项，默认为无。
2. updateOutput(input)，前向forward调用这个函数，计算$Y=F(X)$，$X$为input，输出为$Y$。
3. updateGradInput(input, gradOutput)，反向backward调用这个和下面的函数，该函数已知input $X$，gradOutput $\frac{\partial E}{ \partial Y}$，输出损失对输入的偏导 gradInput $\frac{\partial E}{ \partial X}$，其中$\frac{\partial E}{ \partial X} = \frac{\partial E}{ \partial Y} \times \frac{\partial Y}{ \partial X}$。
4. accGradParameters(input, gradOutput)，该函数已知input $X$，gradOutput $\frac{\partial E}{ \partial Y}$，输出损失对参数的偏导 gradWeight和gradBias，其中 $\frac{\partial E}{ \partial W} = \frac{\partial E}{ \partial Y} \times \frac{\partial Y}{ \partial W}$，$\frac{\partial E}{ \partial B} = \frac{\partial E}{ \partial Y} \times \frac{\partial Y}{ \partial B}$。对于有些没有需要学习参数的层（如nn.ReLU，nn.Dropout等），不需要写这个函数。 

## C和Cuda实现
以上函数的具体实现，若能通过Tensor自带的计算完成，则直接一个lua文件即可，因为Tensor的运算自动支持cpu和gpu，故该层也能直接支持cpu和gpu了。

但若无法直接用Tensor的计算来实现时，就需要自己写C和Cuda的代码了，并且写完后需要编译安装。需要添加和修改的文件如下：

1.新建 torch/extra/nn/NewLayer.lua，这个就是上面提到的通用lua模板了，不过既然要编译安装，放在nn目录下会省去很多麻烦。代码里主要调用后面C和Cuda的实现函数，其调用形式大致如下，省略号表示其他参数，根据层的不同而不同。其中lua的数据通过cdata()转为C/Cuda的指针的形式，然后调用C/Cuda的实现来完成计算。

```lua
input.THNN.NewLayer_updateOutput(
      input:cdata(),
      self.output:cdata(),
      ...
   )
input.THNN.NewLayer_updateGradInput(
      input:cdata(),
      gradOutput:cdata(),
      self.gradInput:cdata(),
      ...
   )
input.THNN.NewLayer_accGradParameters(
      input:cdata(),
      gradOutput:cdata(),
      self.gradInput:cdata(),
      ...
   )
```

2.新建 torch/extra/nn/lib/THNN/generic/NewLayer.c，添加C语言实现代码，需实现的函数定义在THNN.h中。
3.新建 torch/extra/cunn/lib/THCUNN/NewLayer.cu，添加Cuda实现代码，需实现的函数定义在THCUNN.h中。
4.修改 torch/extra/nn/init.lua，添加 require('nn.NewLayer')，将新模块添加到nn的初始化文件中。
5.修改 torch/extra/nn/lib/THNN/init.c，添加以下两行。THGenerateFloatTypes.h 文件细看会发现很有意思的东西，由于C语言没有C++中的模板(Template)，而这里既要支持 float 类型又要支持 double 类型，怎么办呢，torch中大量使用了宏来实现了类似Template的功能。这个[帖子](https://apaszke.github.io/torch-internals.html)就分析了torch的一些细节，下面的评论中还有关于为何不用C++的一些讨论。

```c
#include "generic/NewLayer.c"
#include "THGenerateFloatTypes.h"
```

6.修改 torch/extra/nn/lib/THNN/generic/THNN.h，添加 NewLayer.c 中的函数声明，包括下面两个或者三个函数：

```c
TH_API void THNN_(NewLayer_updateOutput)(
     THNNState *state,
     THTensor *input,
     THTensor *output,
     ...);

TH_API void THNN_(NewLayer_updateGradInput)(
     THNNState *state,
     THTensor *input,
     THTensor *gradOutput,
     THTensor *gradInput,
     ...);

TH_API void THNN_(NewLayer_accGradParameters)(
     THNNState *state,
     THTensor *input,
     THTensor *gradOutput,
     THTensor *gradInput,
     ...);
```

7.修改 torch/extra/cunn/lib/THCUNN/THCUNN.h，添加 NewLayer.cu 中的函数声明，包括下面两个或者三个函数：

```c
 TH_API void THNN_CudaNewLayer_updateOutput(
     THCState *state,
     THCudaTensor *input,
     THCudaTensor *output,
     ...);

TH_API void THNN_CudaNewLayer_updateGradInput(
     THCState *state,
     THCudaTensor *input,
     THCudaTensor *gradOutput,
     THCudaTensor *gradInput,
     ...);

TH_API void THNN_CudaNewLayer_accGradParameters(
     THCState *state,
     THCudaTensor *input,
     THCudaTensor *gradOutput,
     THCudaTensor *gradInput,
     ...);
```

8.修改 torch/extra/nn/test.lua，添加CPU实现的测试代码。
9.修改 torch/extra/cunn/test.lua，添加GPU实现的测试代码。

以上就是所需要添加和修改的部分，写的比较简要，具体实现中可参考torch中原有的模块，特别是C/Cuda代码实现中需要模仿和学习现有的代码中的技巧。

## 编译安装
代码写完后，就可以编译安装了，因为我们将代码添加在了nn和cunn目录下，直接重新编译nn和cunn即可，命令如下：

``` bash
cd torch/extra/nn/
luarocks make rocks/nn-scm-1.rockspec

cd torch/extra/cunn/
luarocks make rocks/cunn-scm-1.rockspec
```

## 其他
本文几月前首发于知乎，现在在这个博客里面再发一次。