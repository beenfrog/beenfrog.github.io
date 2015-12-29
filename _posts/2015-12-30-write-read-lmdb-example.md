---
layout: post
title: Write/Read lmdb file for caffe with python
categories:
- code
comments: true
---
This is an example to show how to wirte and read lmdb file for caffe with python. You can use cv2 to read your own image data.

Write
```python
import lmdb
import numpy as np
import cv2
import caffe
from caffe.proto import caffe_pb2

#basic setting
lmdb_file = 'lmdb_data'
batch_size = 256

# create the leveldb file
lmdb_env = lmdb.open(lmdb_file, map_size=int(1e12))
lmdb_txn = lmdb_env.begin(write=True)
datum = caffe_pb2.Datum()

item_id = -1
for x in range(1000):
    item_id += 1

    #prepare the data and label
    data = np.ones((3,64,64), np.uint8) * (item_id%128 + 64) #CxHxW array, uint8 or float
    label = item_id%128 + 64

    # save in datum
    datum = caffe.io.array_to_datum(data, label)
    keystr = '{:0>8d}'.format(item_id)
    lmdb_txn.put( keystr, datum.SerializeToString() )

    # write batch
    if(item_id + 1) % batch_size == 0:
        lmdb_txn.commit()
        lmdb_txn = lmdb_env.begin(write=True)
        print (item_id + 1)

# write last batch
if (item_id+1) % batch_size != 0:
    lmdb_txn.commit()
    print 'last batch'
    print (item_id + 1)

```

Read
```python
import caffe
import lmdb
import numpy as np
import cv2
from caffe.proto import caffe_pb2

lmdb_env = lmdb.open('lmdb_data')
lmdb_txn = lmdb_env.begin()
lmdb_cursor = lmdb_txn.cursor()
datum = caffe_pb2.Datum()

for key, value in lmdb_cursor:
    datum.ParseFromString(value)

    label = datum.label
    data = caffe.io.datum_to_array(datum)

    #CxHxW to HxWxC in cv2
    image = np.transpose(data, (1,2,0))
    cv2.imshow('cv2', image)
    cv2.waitKey(1)
    print('{},{}'.format(key, label))

```