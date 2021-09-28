---
layout: post
title: 在VPS上搭建类似KindleEar的服务
tags:
- software
- python
- network
- kindle
- linux
categories:
- code
comments: true
mathjax: false
date: 2021-09-29 02:52:13 +0800
---
前几年一直使用[ `KindleEar` ](https://github.com/cdhigh/KindleEar)将rss订阅定时发送到 `Kindle` 上阅读，使用的体验非常的好。`KindleEar` 本身需要搭建在 `GAE` 上，而 `Google` 从去年开始就时不时提醒 `GAE` 如果不添加付款信息那么将停止提供服务。结果还是一直能用，直到今年8月果然是不能用了。由于用 `KindleEar` 好几年了，一时不能用了还挺不习惯的。

经过搜索发现原来知名的电子书管理软件 `calibre` 也能实现将 `rss` 订阅转换为 `mobi` 格式并邮件发送，而 `KindleEar` 也是基于此实现的。这样一来自己在 `VPS` 实现类似的功能就没什么难度了，只是做一些集成就好了。

下面按照 软件安装、`rss`转换、`mobi`发送、系统运行介绍整个实现。

整体实现包括 `feed.csv`、`rss.recipe`、`run_convert`、`kindlefeed.py` 四个文件。

## 软件安装
+ 安装python3
+ 安装[calibre](https://calibre-ebook.com/download_linux), 经测试，最新版无法安装在`CentOS 7`中，该系统推荐用[3.x的旧版](https://download.calibre-ebook.com/3.html)
+ 注册[mailjet](https://www.mailjet.com/)的邮件发送服务，获取`api_key, api_secret`，验证设置自己的发送邮箱。
+ 安装用于发送邮件的`mailjet`服务接口： `pip install mailjet-rest`

## rss转换
首先定义一个需要订阅的 `rss` 列表文件：`**feed.csv**`，标题与地址用逗号隔开，类似如下
```csv
Solidot,https://www.solidot.org/index.rss
Slashdot,http://rss.slashdot.org/Slashdot/slashdotMain?format=usm
```

然后编写 `rss` 转换的设置文件，虽然后缀是 `recipe`，其实是正宗的 `python` 程序，这里示例文件: `**rss.recipe**` 如下
```python
#!/usr/bin/env python3
import csv
import os
from calibre.web.feeds.news import BasicNewsRecipe


def from_file():
    feeds_file = '/var/www/kindlefeed/feed.csv'
    with open(feeds_file) as csvfeeds:
        return [(author.decode("utf-8"), url) for author, url in csv.reader(csvfeeds)]


class Rss2KindleRecipe(BasicNewsRecipe):
    title = "RSS News"
    oldest_article = 1
    max_articles_per_feed = 20
    auto_cleanup = True
    remove_empty_feeds = True
    language = 'zh'
    remove_tags=[dict(name='img')] # 去除图片
    publication_type = 'magazine'

    feeds = from_file()
```

`recipe` 的更多参数设置见 [API documentation for recipes](https://manual.calibre-ebook.com/news_recipe.html)

## mobi发送
定义好 `recipe` 就可以将 `rss` 转为 `mobi` 了，定义转换脚本: `**run_convert**`
```bash
#!/usr/bin/env bash

ebook-convert /var/www/kindlefeed/rss.recipe /var/www/kindlefeed/kindlefeed.mobi \
              --language zh_CN \
              --output-profile kindle \
              --title "RSS News"
```

转换好后就可以通过邮箱发送了，`calibre` 自带 `calibre-smtp` 的命令，需要自己提供发送邮箱的信息，使用起来也不复杂。但是自己以前用过 `mailjet` ，专业的邮件发送服务，更灵活。发送的代码: `**kindlefeed.py**`
```python
import os
import base64
from mailjet_rest import Client

run_convert_str = 'bash /var/www/kindlefeed/run_convert'
os.system(run_convert_str)

api_key = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
api_secret = 'yyyyyyyyyyyyyyyyyyyyyyyyyyyyyy'
mailjet = Client(auth=(api_key, api_secret), version='v3.1')

from_email = "xxx@xxx.com"
to_email   = "xxx@kindle.com"
subject    = ""
message    = 'hello, kindle'

file_location = "/var/www/kindlefeed/kindlefeed.mobi"
filename = os.path.basename(file_location)
b64content = None
with open(file_location, "rb") as fid:
	b64content = base64.b64encode(fid.read()).decode("utf-8")

data = {
  'Messages': [
    {
      "From": {
        "Email": from_email,
        "Name": "Me"
      },
      "To": [
        {
          "Email": to_email,
          "Name": "kindle"
        }
      ],
      "Subject": subject,
      "TextPart": message,
      "HTMLPart": message,
      "Attachments": [
        {
       	  "ContentType": "application/octet-stream",
       	  "Filename": filename,
       	  "Base64Content": b64content
       	}
      ]
    }
  ]
}

result = mailjet.send.create(data=data)
print(result.json())
```

## 系统运行
定时运行 `kindlefeed.py` 即可实现预想的功能，要实现定时运行，只要在 `/etc/crontab` 添加如下一行即可实现每天18点发送，注意`python` 也要用全路径。
```bash
0 18 * * * root /opt/anaconda3/bin/python /var/www/kindlefeed/kindlefeed.py >>/var/log/kindlefeed.log 2>&1 &
```

显然，自己实际运行时要修改上面代码中的文件路径，以及 `mailjet` 的 `key` 和邮箱地址。
