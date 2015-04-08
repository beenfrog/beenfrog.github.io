---
layout: post
title: View layer blobs in Caffe
categories:
- code
comments: true
---

Useful code for caffe

```python
import caffe
import numpy as np
import cv2

def view_layer(net, layer_name, pad = 8, pad_color = 0.2):
    layer_data = net.blobs[layer_name].data
    
    layer_data -= layer_data.min()
    if layer_data.max() - layer_data.min() != 0:
        layer_data /= (layer_data.max() - layer_data.min())

    _,ch,h,w = layer_data.shape

    rows = np.floor( np.sqrt(ch) ).astype(int)
    cols = np.ceil( ch * 1.0 / rows ).astype(int)
    img_blobs = np.ones( (rows*(h+pad) , cols*(w+pad) ) ) * pad_color

    for idx in range(ch):
        r = idx / cols
        c = idx % cols

        img_blobs[r*(h+pad):r*(h+pad)+h, c*(w+pad):c*(w+pad)+w] = layer_data[0,idx,:,:]

    cv2.imshow(layer_name, img_blobs)
```
