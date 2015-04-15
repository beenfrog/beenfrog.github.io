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
    layer_data_raw = net.blobs[layer_name].data
    layer_data = layer_data_raw.copy()
    
    layer_data -= layer_data_raw.min()
    if layer_data_raw.max() - layer_data_raw.min() != 0:
        layer_data /= (layer_data_raw.max() - layer_data_raw.min())

    _,ch,h,w = layer_data.shape

    rows = np.floor( np.sqrt(ch) ).astype(int)
    cols = np.ceil( ch * 1.0 / rows ).astype(int)
    img_blobs = np.ones( (rows*(h+pad) , cols*(w+pad) ) ) * pad_color

    for idx in range(ch):
        r = idx / cols
        c = idx % cols

        img_blobs[r*(h+pad):r*(h+pad)+h, c*(w+pad):c*(w+pad)+w] = layer_data[0,idx,:,:]

    cv2.imshow(layer_name, img_blobs)

def view_local(net, layer_name, channel = 0, pad = 2, pad_color = 0.0):
    layer_data_raw = net.params[layer_name][0].data[channel,:,:,:]
    layer_data = layer_data_raw.copy()
    
    layer_data -= layer_data_raw.min()
    if layer_data_raw.max() - layer_data_raw.min() != 0:
        layer_data /= (layer_data_raw.max() - layer_data_raw.min())

    _,filter_all,pixel_all = layer_data.shape
    w = h = np.sqrt(filter_all).astype(int)
    rows = cols = np.sqrt(pixel_all).astype(int)
    img_filters = np.ones( (rows*(h+pad) , cols*(w+pad) ) ) * pad_color

    for r in range(rows):
        for c in range(cols):
            img_filters[r*(h+pad):r*(h+pad)+h, c*(w+pad):c*(w+pad)+w] = \
            np.transpose( layer_data[0,:,r*rows + c].reshape(h, w) , (0,1))

    cv2.imshow(layer_name, img_filters)
```
