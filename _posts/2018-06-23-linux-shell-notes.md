---
layout: post
title: Linux shell notes
tags:
- linux
categories:
- code
comments: true
mathjax: false
date: 2018-06-23 16:15:42 +0800
---
一些Linux命令以及示例。

## wget
```bash
wget www.xxx.xxx/file.txt
wget www.xxx.xxx/file{000..100}.txt
wget -i urllist.txt
```

## curl
```bash
curl -O www.xxx.xxx/file.txt
curl -O www.xxx.xxx/file[000-100].txt
```
## qpdf
```bash
qpdf --decrypt infile.pdf outfile.pdf
```
## pdf compress
```bash
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/ebook -dNOPAUSE  -dBATCH -sOutputFile=compress.pdf input.pdf
```
