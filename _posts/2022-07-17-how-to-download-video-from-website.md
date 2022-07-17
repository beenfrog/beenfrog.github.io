---
layout: post
title: 如何从视频网站上下载视频
tags:
- network
- software
categories:
- code
comments: true
mathjax: false
date: 2022-07-17 19:51:13 +0800
---
虽说现在在线观看视频很方便，但是网络上资源的稳定性是靠不住的，可能不久就因为版权、审查等原因就不见了，所以遇到喜欢的视频必须及时下载保存。

## 常用的下载工具
+ [yt-dlp](https://github.com/yt-dlp/yt-dlp),是`youtube-dl`的fork版本，速度更快，支持的网站不限于`YouTube`，尤其境外的视频网站支持的非常好。
+ [lux](https://github.com/iawia002/lux)，国内视频网站支持的非常好。
+ [you-get](https://github.com/soimort/you-get)，国内视频网站支持的较好。
+ [ykdl](https://github.com/SeaHOH/ykdl)，国内视频网站支持的较好。

需要下载的时候，可以依次试试，这几款各有所长，可能有一款会有惊喜。

## 手动下载
如果上述四款工具都不可用，那么就需要自己分析了，方法就是打开F12调试，刷新，在log里面搜索类似`m3u8`的字段，如果找到了可用`yt-dlp`下载该`url`即可。或者自己分析`m3u8`文件，下载每个片段，最后用`ffmpeg`合并即可，一开始自己就是写代码这么处理的，后来发现`yt-dlp`就能直接解决这个问题。

有些视频网站为了防止用户分析下载，在用户进入调试模式后视频网页就不工作或者直接跳转回主页。遇到这样的情况可以开启`Firefox`的全局日志，这样不用开启调试就可以获取所有的网页请求。方法如下：
```bash
# 在命令行中通过下面三行启动firefox
export MOZ_LOG=timestamp,nsHttp,nsHostResolver
export MOZ_LOG_FILE=firefox_http.log
/usr/bin/firefox
# 打开所需的视频网站，开始播放，之后在命令行中用如下脚本分析日志文件
grep uri firefox_http.log.moz_log  |awk -F 'uri=' '{print $2}' |grep http
# 根据日志结果，做出处理
```