---
layout: post
title: How to load data in pytorch
tags:
- pytorch
- python
categories:
- code
comments: true
mathjax: true
date: 2018-08-30 18:29:05 +0800
---
How to load data in pytorch?

## Custom data

```python
import torch
import torch.utils.data as data
import torchvision

class CustomData(data.Dataset):
	def __init__(self, num=10):
		self.num = num
		self.data = torch.zeros((self.num, 2), dtype=torch.int32)
		for idx in range(self.num):
			self.data[idx, 0] = idx
			self.data[idx, 1] = -idx

	def __len__(self):
		return self.num

	def __getitem__(self, index):
		return self.data[index, :]


if __name__ == "__main__":
	
	#============ torch.utils.data.Dataset
	dataset  = CustomData()
	dataloader = data.DataLoader(dataset=dataset, batch_size=3,
		         shuffle=True, drop_last=True, num_workers=4)
	for it, batch_data in enumerate(dataloader):
		print(it)
		print(batch_data)


	#============  torch.utils.data.TensorDataset
	tensor = torch.zeros((10, 2), dtype=torch.int32)
	for idx in range(tensor.shape[0]):
		tensor[idx, 0] = idx
		tensor[idx, 1] = -idx

	dataset = data.TensorDataset(tensor)
	dataloader = data.DataLoader(dataset=dataset, batch_size=3,
		         shuffle=True, drop_last=True, num_workers=4)
	for it, batch_data in enumerate(dataloader):
		print(it)
		print(batch_data)


	#==============  torch.utils.data.ConcatDataset
	tensor1 = torch.zeros((10, 2), dtype=torch.int32)
	for idx in range(tensor1.shape[0]):
		tensor1[idx, 0] = idx
		tensor1[idx, 1] = -idx
	tensor2 = torch.zeros((10, 2), dtype=torch.int32)
	for idx in range(tensor2.shape[0]):
		tensor2[idx, 0] = idx+100
		tensor2[idx, 1] = -(idx + 100) 

	dataset1 = data.TensorDataset(tensor1)
	dataset2 = data.TensorDataset(tensor2)
	dataset = data.ConcatDataset([dataset1, dataset2])

	dataloader = data.DataLoader(dataset=dataset, batch_size=3,
		         shuffle=False, drop_last=True, num_workers=4)
	for it, batch_data in enumerate(dataloader):
		print(it)
		print(batch_data)
```

## Image data

```python
import torch
import torch.utils.data as data
import torchvision.datasets as datasets
import torchvision.transforms as transforms
import torchvision.utils as utils

rootpath = "/home/xxx/data/"
transform = transforms.Compose([
				transforms.CenterCrop((100,100)),
				transforms.Resize((64,64)),
				transforms.RandomHorizontalFlip(),
				transforms.ToTensor(),
				transforms.Normalize((0.5,0.5,0.5), (0.5,0.5,0.5))])
dataset = datasets.ImageFolder(root=rootpath, transform=transform)

dataloader = data.DataLoader(dataset=dataset, batch_size=32,
		         shuffle=True, drop_last=True, num_workers=4)

for it, batch in enumerate(dataloader):
	
	if it == 0:
		# print(it)
		# print(batch[0], batch[1] )
		img_grid = utils.make_grid((batch[0]+1)*0.5, nrow=8, normalize = False)
		print(img_grid.shape)
		print(img_grid.dtype)
		utils.save_image((batch[0]+1)*0.5, 'grid.png',  nrow=8, normalize = False)

		break;

```
