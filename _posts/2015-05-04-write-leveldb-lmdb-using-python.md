---
layout: post
title: Write leveldb/lmdb using python
categories:
- code
comments: true
---
This script is useful to create leveldb/lmdb data in caffe.

Creating leveldb file with python script.

```python
import leveldb
import numpy as np
import cv2
import caffe
from caffe.proto import caffe_pb2

#basic setting
leveldb_file = 'xxxx'
batch_size = 1234

# create the leveldb file
db = leveldb.LevelDB(leveldb_file)
batch = leveldb.WriteBatch()
datum = caffe_pb2.Datum()

item_id = -1
for x in xxx:
	item_id += 1

	#prepare the data and label
	#data = ... #CxHxW array, uint8 or float
	#label = ... #int number

	# save in datum
	datum = caffe.io.array_to_datum(data, label)
	keystr = '{:0>8d}'.format(item_id)
	batch.Put( keystr, datum.SerializeToString() )

	# write batch
	if(item_id + 1) % batch_size == 0:
		db.Write(batch, sync=True)
		batch = leveldb.WriteBatch()
		print (item_id + 1)

# write last batch
if (item_id+1) % batch_size != 0:
	db.Write(batch, sync=True)
	print 'last batch'
	print (item_id + 1)
```

Creating lmdb file with python script.

```python
import lmdb
import numpy as np
import cv2
import caffe
from caffe.proto import caffe_pb2

#basic setting
lmdb_file = 'xxxx'
batch_size = 1234

# create the leveldb file
lmdb_env = lmdb.open(lmdb_file, map_size=int(1e12))
lmdb_txn = lmdb_env.begin(write=True)
datum = caffe_pb2.Datum()

item_id = -1
for x in xxx:
	item_id += 1

	#prepare the data and label
	#data = ... #CxHxW array, uint8 or float
	#label = ... #int number

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