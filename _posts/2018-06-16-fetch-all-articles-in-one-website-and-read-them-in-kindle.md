---
layout: post
title: 抓起某网页所有文章并使用kindle阅读
tags:
- python
- network
categories:
- code
comments: true
mathjax: false
date: 2018-06-16 21:37:15 +0800
---
发现某网站的内容不错，想全部抓下来放在`kindle`上，用代码实现了一下半自动的过程，记录如下。

## 任务分析
+ 首先该网站的文章`url`最后部分按照数字不断递增的，最高一千多一点，这样很容易通过循环下载所有内容。
+ 下载一堆`html`文件后，需要分析提取其中的有用内容，如标题和正文。
+ 重新组织内容以便适宜`kindle`阅读。

## 实现细节
### 内容抓取
该部分比较简单，直接使用`python`调用`wget`实现：
```python
import os
import time
from DotDict import DotDict

class Spider():
	def __init__(self, params):
		self.p = params
		self.url_base = "http://***.***/*/{}"

	def fetch(self):
		for idx in range(self.p.start_id, params.end_id+1):
			this_url  = self.url_base.format(idx)
			os.system('wget ' + this_url)
			time.sleep(0.01)

if __name__ == '__main__':

	params = DotDict()
	params.start_id  = 1
	params.end_id    = 1***

	spider_inst = Spider(params)
	spider_inst.fetch()
```

```python
class DotDict(dict):
	"""dot.notation access to dictionary attributes"""
	# https://stackoverflow.com/a/36968114/3100697
	def __getattr__(self, attr):
		return self.get(attr)
	__setattr__= dict.__setitem__
	__delattr__= dict.__delitem__

	def __getstate__(self):
		return self

	def __setstate__(self, state):
		self.update(state)
		self.__dict__ = self
```

### 提取网页信息
利用`BeautifulSoup`包来实现，第一次使用，核心要点是找需要内容对应的关键词，之后保存为`markdown`格式。
```python
import os
from bs4 import BeautifulSoup

class Html2Text():
	def __init__(self, html_path, markdown_file):
		self.html_path     = html_path
		self.markdown_file = markdown_file

	def run(self):
		html_list = list( os.listdir(self.html_path) )
		html_list = sorted(html_list, key=int)[::-1]
		md_file = open(self.markdown_file ,'w')

		for html_file in html_list:
			full_name = os.path.join(self.html_path, html_file)
			item_title   = BeautifulSoup(open(full_name), 'lxml').find("h1",class_="page-title")
			item_content = BeautifulSoup(open(full_name), 'lxml').find("div",property="content:encoded")

			if "******" not in item_title.text and \
			    			item_content is not None and\
			    			item_content.text is not None:

				title   = item_title.text
				content = item_content.text
				md_file.write('### ' + title + '\n\n')
				md_file.write(content)
				md_file.write('\n\n\n\n')

		md_file.close()
if __name__ == '__main__':
	convert = Html2Text('./html', './******.md')
	convert.run()
```

### 转为`kindle`友好的格式
这里将`markdown`文件粘贴到`wps`里面，然后通过替换功能将`###`替换为标题的样式，然后再通过替换删除`###`，并创建目录。再通过发邮件转换的方式发给`kindle`，由于原来`doc`文件有目录，转换后的文档也有目录，方便在`kindle`中选择阅读。
