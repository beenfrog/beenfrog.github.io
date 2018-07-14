---
layout: post
title: A simple pytorch example
tags:
- pytorch
categories:
- code
comments: true
mathjax: false
date: 2018-07-13 16:47:25 +0800
---

A simple pytorch(version: 0.5.0a0) example inludes `load data`, `define model`, `train`, `test`, `view log(tensorboard)`, `save model`.

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.utils.data as data
import torch.optim as optim
from tensorboardX import SummaryWriter
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--cuda_id", type=int, default=0, help="gpu id")
param = parser.parse_args()

class PointDataset(data.Dataset):
	def __init__(self, data_num=1000):
		self.data_num = data_num
		self.data_tensor = torch.zeros(self.data_num, 2).uniform_(-1,1)
		self.label_tensor = (self.data_tensor[:,0] * self.data_tensor[:,1]).gt(0).to(torch.long)

	def __getitem__(self, index):
		return self.data_tensor[index,:], self.label_tensor[index]

	def __len__(self):
		return self.data_num

class Classifier(nn.Module):
	def __init__(self):
		super(Classifier, self).__init__()
		self.fc1 = nn.Linear(2,30)
		self.fc2 = nn.Linear(30,2)

	def forward(self, x):
		x = self.fc1(x)
		x = F.relu(x)
		x = self.fc2(x)
		return x

class GridTest():
	def __init__(self, device, num=101):
		self.device = device
		self.num    = num
		self.data2d = torch.zeros(self.num*self.num, 2)
		half = (self.num -1 ) / 2
		idx = 0
		for y in range(self.num):
			for x in range(self.num):
				self.data2d[idx, 0] = (x - half) / half
				self.data2d[idx, 1] = (y - half) / half
				idx = idx + 1

	def run(self, net):
		net.eval()
		with torch.no_grad():
			output = F.softmax( net(self.data2d.to(self.device)), 1 )
			result = output.data[:,0].clone()
			img = result.view(self.num, self.num).to('cpu')
		net.train()
		return img

writer = SummaryWriter()
device = torch.device('cuda:%d'%(param.cuda_id)) if torch.cuda.is_available() else torch.device('cpu')

dataset    = PointDataset(1000)
dataloader = data.DataLoader(dataset, batch_size=100, shuffle=True, num_workers=4)
net = Classifier()
net.to(device)
cri = nn.CrossEntropyLoss()
optimizer = optim.Adam(net.parameters(), lr=0.01)

tester = GridTest(device)

for epoch in range(1000):
	
	loss_total = 0
	for it, (batch_data, batch_label) in enumerate(dataloader):
		optimizer.zero_grad()
		loss = cri(net(batch_data.to(device)), batch_label.to(device))
		loss.backward()
		loss_total += loss.item()
		optimizer.step()

	print('epoch:%d, loss=%6.5f'%(epoch, loss_total))
	writer.add_scalar('loss', loss_total, epoch) 

	if epoch % 10 == 0:
		img = tester.run(net)
		writer.add_image('grid', img, epoch)
	if epoch % 100 == 0:
		torch.save(net.state_dict(), './runs/modelsnet_%04d.model'%(epoch))
```
