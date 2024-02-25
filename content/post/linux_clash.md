---
title: "Linux代理配置"
description: "linux_clash"
keywords: "linux_clash"

date: 2024-02-25T16:01:53+08:00
lastmod: 2024-02-25T16:01:53+08:00

categories:
 -
tags:
  -
  -

# Post's origin author name
#author:
# Post's origin link URL
#link:
# Image source link that will use in open graph and twitter card
#imgs:
# Expand content on the home page
#expand: true
# It's means that will redirecting to external links
#extlink:
# Disabled comment plugins in this post
#comment:
# enable: false
# Disable table of content int this post
# Notice: By default will automatic build table of content 
# with h2-h4 title in post and without other settings
#toc: false
# Absolute link for visit
#url: "linux_clash.html"
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
# Support Math Formulas render, options: mathjax, katex
#math: mathjax
# Enable chart render, such as: flow, sequence, classes etc
#mermaid: true
---

国内云服务器经常遇到的一个问题就是 git clone 非常慢，在没有代理的情况下，能不能 clone 下来都是看运气，重试几次可能就可以。

另外在安装一些软件需要访问国外网站的时候，如果国内没有提供代理源，那也是非常慢，平时想要尝试一些新东西的时候，第一步内容下载就卡住了。

对于 windows 平台，有很多图形化的工具的代理工具，在配置上也相对直观方便，但是在云服务器上，只有终端，缺乏一个方便快捷的手动来配置代理，需要很多的精力去探索，没有傻瓜化的操作方法。

# Clash 安装
由于近期 clash 仓库都被删除了，可以使用 github 上备份的[下载地址](https://github.com/Kuingsmile/clash-core/releases/tag/1.18)。（这个时候 linux 机器还没有对应的代理能力，如果直接在 linux 机器上下载因为网络问题下载不了的话，可以在本地电脑先下载，再传到 linux 机器上）

示例下载的版本是：[clash-linux-amd64-v1.18.0.gz](https://github.com/Kuingsmile/clash-core/releases/download/1.18/clash-linux-amd64-v1.18.0.gz)，有更新的话可以下载最新的。解压后得到一个 clash 的可执行文件，为该文件增加可执行权限后执行：`./clash`。

第一次执行会在 ~/.config/clash 中生成一些配置文件（下载 Country.mmdb 的时候可能有网络问题，多尝试几次）：
```bash
cache.db  config.yaml Country.mmdb
```

# 订阅配置
由于下载得到的 clash 并不提供更新订阅配置的能力，所以需要通过额外的手段来更新配置。

在云服务器使用场景下，在 git clone 或者安装软件的时候才需要进行代理，所以对配置订阅的自动化管理并不是强需求，所以这里选择的方式是将本地使用的 clash 配置直接拷贝上去。

这里我使用的是 clash verge 客户端，配置文件可以从这里打开：
![image.png](https://images.lightpixels.top/img/20240225150743.png)

复制这个文件的内容，覆盖掉 `~/.config/clash` 路径下的 config.yaml

# 选择节点
在 Linux 上选择节点是要给比较麻烦的事情，这里有一个比较讨巧的方法：使用第三方的平台接入，在 web 端进行选择。

这两个网址提供了接入的能力，但是由于这两个网站是第三方的，所以接入的时候需要通过公网才行。
> https://clash.razord.top
> 
> https://yacd.haishan.me/ 

在配置中提供了外部控制的接口，默认是 9090，这样就可以通过该端口进行管理了，前提是要把云服务器的 9090 端口也开放下。
```yaml
external-controller: ':9090'
```

但由于直接在公网开放一个端口还是比较危险的，特别是对于相对公开的端口，所以建议修改下默认端口。

如果不想使用上面的第三方页面，也可以自动搭建，yacd 提供了 docker 安装方法，启动下就可以了。参考：https://github.com/haishanh/yacd

# 代理

前台启动 clash
```bash
./clash -d /root/.config/clash/
```

在想要代理的终端执行：
```bash
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890
```

然后就可以愉快的 clone 和安装软件了。

取消代理可以执行以下命令：
```bash
unset  http_proxy  https_proxy 
```

# ShellCrash
针对命令行的 clash 代理能力，找到了另外一个项目，虽然这个项目提供了比较方便的脚本，但是使用起来还是有一定门槛，放在这里作为参考。

https://github.com/juewuy/ShellCrash ，对应的[教程](https://github.com/DustinWin/clash_singbox-tutorials/blob/main/%E6%95%99%E7%A8%8B%E5%90%88%E9%9B%86/Clash/%E5%85%A8%E7%BD%91%E6%9C%80%E8%AF%A6%E7%BB%86%E7%9A%84%E8%A7%A3%E9%94%81%20SSH%20ShellCrash%20%E6%90%AD%E9%85%8D%20AdGuardHome%20%E5%AE%89%E8%A3%85%E5%92%8C%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B-Clash%20%E6%96%B9%E6%A1%88.md)

# 参考链接
https://blog.iswiftai.com/posts/clash-linux/
