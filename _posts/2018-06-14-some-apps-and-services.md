---
layout: post
title: 自己使用的一些软件和服务
tags:
- software
- network
categories:
- blog
comments: true
mathjax: false
date: 2018-06-14 18:21:55 +0800
---
总结一下最近使用的一些软件和服务，并记录一下搭建或者使用的要点，后续有用到有趣的的再添加。

## Telegram
+ 安全性、跨平台性、速度都很不错，通过手机注册，短信验证码登录。
+ 使用的一些`bot`如下，从名字也能知道其功能
  + `@BotFather`
  + `@GmailBot`
  + `@weatherbot`
  + `@AirPollution_Bot`
  + `@temp_mail_bot`
  + `@scihubot`
+ 有用的链接：[〇](https://telegram.org/)

## Vultr
+ 为了能方便使用`Telegram`，买了`Vultr`的`VPS`，支持支付宝，但是当时为了优惠，注册使用了`paypal`。
+ 服务器最好禁用密码登录，而使用`key`登录，减少网上那些暴力尝试密码的风险。
+ 有用的链接：[〇](https://www.vultr.com), [一](http://zlxdike.github.io/2017/05/28/Vultr-VPS-SSH%E5%AF%86%E9%92%A5%E7%99%BB%E5%BD%95/), [二](http://coolnull.com/3486.html)

## Paypal
+ 接上，为了使用上面的优惠，而注册的，可以绑定中国银联的借记卡。
+ 付款时注意汇率的坑，要修改一下：在`paypal`里——设置——付款——管理预核准付款——设置可用资金来源——兑换选项——在给我的账单中使用卖家列出的币种。
+ 意外的用途，可以很方便的给`Wikimedia`,`Mozilla`等基金会捐赠，以前看到`wikipedia`的捐款倡议都是有心有力无途径。
+ 有用的链接：[〇](https://www.paypal.com/c2/home)

## Shadowsocks
+ 口碑很不错，教程网上很多，在`Centos 7`里面要通过`firewall-cmd`开启相应端口，这点有的教程未提，当时折腾好好久。
+ 新版的`Android`客户端貌似不支持4.4的老系统，尝试发现`shadowsocks-nightly-3.4.2.ap`k可用，可在`github release`中下载使用。
+ `ios`的客户端基本在国内都被下架了，最后发现`SsrConnectPro`国区可用。
+ `Ubuntu`无法全局使用，可在浏览器使用`Proxy SwitchyOmega`，命令行的`.bashrc`中设置`export ALL_PROXY=socks5://127.0.0.1:1080`，之后`git clone`速度飞快。
+ 好多`vps`的的ip被`Google Scholar`判定为异常，可在服务器的`/etc/hosts`中添加如下`ipv6`的地址，信息来自[ipv6-hosts](https://raw.githubusercontent.com/lennylxx/ipv6-hosts/master/hosts)
  ```bash
  2404:6800:4008:c06::be scholar.google.com
  2404:6800:4008:c06::be scholar.google.com.hk
  2404:6800:4008:c06::be scholar.google.com.tw
  2401:3800:4001:10::101f scholar.google.cn
  2404:6800:4008:c06::be scholar.l.google.com
  2404:6800:4008:803::2001 scholar.googleusercontent.com
  ```
+ 有用的链接：[〇](https://github.com/shadowsocks/shadowsocks/wiki), [安装一](https://thief.one/2017/02/22/Shadowsocks%E6%8A%98%E8%85%BE%E8%AE%B0/), [安装二](https://github.com/sirzdy/shadowsocks), [BBR三](https://www.isthnew.com/archives/centos7-bbr.html), [本地四](https://pangsuan.com/p/ubuntu-shadowsocks-client.html)

## Rsshub
+ 用来给没有`rss`输出的网站以`rss`输出功能，支持知乎，微博等。
+ 可使用官方提供的服务，也可以在自己`vps`上搭建，主要是安装nodejs的环境以及依赖，同样要使用`firewall-cmd`开启端口。
+ [feedx](https://feedx.net/)也提供了许多知名网站的全文rss输出。
+ 有用的链接：[〇](https://github.com/DIYgod/RSSHub), [一](https://docs.rsshub.app/install/), [二](https://docs.rsshub.app/)

## Rssbot
+ `Telegram`的一个`bot`，可用来订阅`rss`，有更新时有提醒，相比`feedly`更新频率可以自己设定，结合`Telegram`能更快的获取信息通知。
+ 可使用官方提供的`bot`服务，也可以自己下载`release`提供的`rssbot`直接运行`nohup ./rssbot data.json TELEGRAM_BOT_KEY 300 >/dev/null 2>&1 &`即可，其中第一个参数是保存的数据文件，会自动创建，第二个是`bot`的`key`，第三个是抓起更新的间隔，单位为秒。
+ 该`bot`的基本功能如下:
  ```bash
  /rss       - 显示当前订阅的 RSS 列表，加 raw 参数显示链接
  /sub       - 订阅一个 RSS: /sub http://example.com/feed.xml
  /unsub     - 退订一个 RSS: /unsub http://example.com/feed.xml
  /unsubthis - 使用此命令回复想要退订的 RSS 消息即可退订, 不支持 Channel
  /export    - 导出为 OPML
  ```
+ 有用的链接：[〇](https://github.com/iovxw/rssbot)

## Nginx
+ 搭建一个简单的文件共享服务器，首先安装`nginx`，通过`firewall-cmd`开启`http`服务，修改`/etc/nginx/nginx.conf`，将`user nginx`改为`user root`，并注释掉默认的`server`内容，自己添加一个`/etc/nginx/conf.d/fileshare.conf`，内容参照如下：
  ```
  server {
      listen       80 default_server;
      listen       [::]:80 default_server;
      server_name  _;
      root         /root/sharefile;
  
      location / {
          autoindex on;
          autoindex_exact_size on;
          autoindex_localtime on;
      }
  }
  ```
+ 命令行输入`nginx`即可启用，`nginx -s reload`为重启，`nginx -s stop`为停止，`nginx -t`显示状态。
+ 有用的链接：[〇](https://www.jianshu.com/p/95602720e7c8)

## KindleEar
+ 在`GAE`上搭建自己的Kindle推送，先在`GAE`建立一个`Pyhton`项目，后进入网页控制台里面的`shell`，运行[KindleEar-Uploader](https://github.com/kindlefere/KindleEar-Uploader)的脚本进行安装。
+ 搭好后，通过`app-id.appspot.com`登录管理，默认用户名和密码均是`admin`。
+ 在控制台里面，计算->`App Engine`->设置->自定义网域，可以自定义域名，根据提示先通过TXT记录验证自己的主域名 `******.com`，然后添加`kindle.******.com`二级域名作为该项目的二级域名。在`DNS`提供商处添加值为kindle,指向`ghs.******.com`的二级记录，原理见文中的`Blogger`章节。
+ 有用的链接: [〇](https://github.com/cdhigh/KindleEar), [一](https://sspai.com/post/40509), [二](https://bookfere.com/post/19.html)

## Firefox Quantum
+ 新版的`Firefox`速度很不错，最新版可直接下载可执行文件，放在`/opt`，自己在`~/.local/application`仿造添加`desktop`文件即可。
+ 之前考虑到插件问题，一直没升级，不过试了之后，发现虽然一些插件有缺失，但速度优势明显，目前用的插件如下：
  + Adblock Plus
  + Evernote Web Clipper
  + Proxy SwitchyOmega
  + Push to Kindle
+ 有用的链接: [〇](https://www.mozilla.org/zh-CN/firefox/all/)

## Electronic WeChat
+ `Linux`下的微信客户端，直接下载[`electronic-wechat release`](https://github.com/geeeeeeeeek/electronic-wechat/releases)即可。
+ 有用的链接: [〇](https://github.com/geeeeeeeeek/electronic-wechat)

## Blogger Ghs地址
+ 目前`blogger`的`blogspot`网站可以通过自定域名方式实现墙内访问，需要将博客域名`CNAME`到`ghs.google.com`，但是该域名也被墙。
+ 可在[`ipip ping`](https://www.ipip.net/ping.php)中找`ghs.google.com`的国内能访问的`ip`。
+ 同样测试该`ip`在国内可访问情况，在`DNS`提供商处建立一个ghs的A记录并指向这个好的`ip`。这样自己二级域名`ghs.******.com`相当于`ghs.google.com`，然后将`www`通过`CNAME`指向该`ghs.******.com`即可。
+ 通过建立一个中间`ghs`而不是直接将`www`A记录到`ip`的好处是该`ghs`可多出使用，如文中的`KindleEar`也用了该`ghs`，若以后可用的`ip`有变化，只修改一处即可。
+ 有用的链接: [〇](https://www.jingfengshuo.com/archives/226.html)

## Sci-Hub
+ 目前可用地址[http://sci-hub.tw](http://sci-hub.tw)
+ 最新地址发布[http://sci-hub.love](http://sci-hub.love)
+ 最新地址转跳[http://sci-hub.app](http://sci-hub.app)

## LibGen
+ [地址一](http://gen.lib.rus.ec/)
+ [地址二](http://libgen.io/)

## Website
+ [`torrentz2`](https://torrentz2.eu)搜国外资源的的种子很方便。
+ [同韵查询](http://www.iguci.cn/dictionary/yunzhe.php)，编顺口溜时有用。
+ [谷歌镜像](https://ac.scmor.com/)，无翻墙环境可临时使用。
+ [知网英文版](http://oversea.cnki.net)，学位论文可以下载pdf格式。
+ [`ZygoteBody`](https://www.zygotebody.com)，人体结构。
