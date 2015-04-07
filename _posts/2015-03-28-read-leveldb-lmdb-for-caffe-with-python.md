---
layout: post
title: Read leveldb/lmdb for caffe with python
categories:
- code
comments: true
---

The code piece for read leveldb or lmdb with python.

##leveldb

```python
import caffe
import leveldb
import numpy as np
from caffe.proto import caffe_pb2

db = leveldb.LevelDB('leveldb_file')
datum = caffe_pb2.Datum()

for key, value in db.RangeIter():
	datum.ParseFromString(value)

	label = datum.label
	data = caffe.io.datum_to_array(datum)
```

##lmdb

```python
import caffe
import lmdb
import numpy as np
from caffe.proto import caffe_pb2

lmdb_env = lmdb.open('lmdb_file')
lmdb_txn = lmdb_env.begin()
lmdb_cursor = lmdb_txn.cursor()
datum = caffe_pb2.Datum()

for key, value in lmdb_cursor:
	datum.ParseFromString(value)

	label = datum.label
	data = caffe.io.datum_to_array(datum)
```
