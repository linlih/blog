---
title: "Tmux 使用"
description: "tmux"
keywords: "tmux"

date: 2024-03-02T21:32:12+08:00
lastmod: 2024-03-02T21:32:12+08:00

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
#url: "tmux.html"
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
# Support Math Formulas render, options: mathjax, katex
#math: mathjax
# Enable chart render, such as: flow, sequence, classes etc
#mermaid: true
---


软件安装：`apt install tmux` 就可以了

然后安装一下 oh my tmux，已经配置好了很多内容，免得自行配置，可以直接使用：
```bash
$ cd
$ git clone https://github.com/gpakosz/.tmux.git
$ ln -s -f .tmux/.tmux.conf
$ cp .tmux/.tmux.conf.local .

```

鼠标模式：`<prefix> m` 这样就可以使用鼠标来选择 pane，以及选中复制的能力

tmux 常用的命令，有一张快速的参考表：https://tmuxcheatsheet.com/

这里总结下常用的：

| Session 管理 | 命令                                | 示例                        |
| ---------- | --------------------------------- | ------------------------- |
| 创建会话       | tmux new -s session-name          | tmux new -s demo          |
| 进入会话       | tmux a -t session-name            | tmux a -t demo            |
| 列出会话       | tmux ls                           |                           |
| 删除会话       | tmux kill-session -t session-name | tmux kill-session -t demo |
| 删除所有会话     | tmux kill-server                  |                           |
| 列出会话       | \<prefix\> s                      |                           |
| 退出会话       | \<prefix\> d                      |                           |
| 重命名会话      | \<prefix\> $                      |                           |

| Window 管理 | 命令                | 示例           |
| --------- | ----------------- | ------------ |
| 新建窗口      | \<prefix\> c      |              |
| 重命名窗口     | \<prefix\> ,      |              |
| 切换窗口      | \<prefix\> number | \<prefix\> 1 |
| 列出窗口      | \<prefix\> w      |              |
| 关闭窗口      | \<prefix\> $      |              |

| Pane 管理 | 命令                      | oh my tmux   |
| ------- | ----------------------- | ------------ |
| 垂直分割    | \<prefix\> %            | \<prefix\> _ |
| 水平分割    | \<prefix\> "            | \<prefix\> - |
| 关闭      | \<prefix\> x  或者输入 exit |              |
| 选择pane  | \<prefix\> 方向键          |              |
| 自动布局    | \<prefix\> space        |              |

关于 oh my tmux 有很多的配置，用到了到时候再补充吧

