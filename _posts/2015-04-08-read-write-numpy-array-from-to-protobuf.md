---
layout: post
title: Read/write numpy array from/to protobuf
tags:
- python
- caffe
categories:
- code
comments: true
---

Useful code for caffe

```python
import caffe
import leveldb
from caffe.proto import caffe_pb2
import numpy as np

# write 
mean_array = np.zeros((1,2,48,48))
mean_blobproto = caffe.io.array_to_blobproto(mean_array)
f = open('mean.bin', 'wb')
f.write(mean_blobproto.SerializeToString())
f.close()

# read
mean_blobproto_new = caffe_pb2.BlobProto()
f = open('mean.bin', 'rb')
mean_blobproto_new.ParseFromString( f.read() )
mean_array_new = caffe.io.blobproto_to_array( mean_blobproto_new )
f.close()
```
