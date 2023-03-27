---
title: "国内git加速"
tags: []
date: 2023-02-11T20:59:17+08:00
layout: 'post'
draft: false
---

一步式操作：

```bash
git config --global url."https://ghproxy.com/https://github.com/".insteadOf "https://github.com/"
git config --global protocol.https.allow always
```

<!--more-->

国内访问 ``git`` 各种意义上非常痛苦，不考虑部分优质环境，大多数人都选择使用代理来解决 ``github`` 的问题。

比如此前常用的方法是 ``git clone https://github.com/example.git`` 的时候，手动替换掉对应的 url，比如 ``git clone https://ghproxy.com/https://github.com/example.git``。

但是这样毕竟只能处理单个仓库，甚至在处理其子模块的时候也会产生新问题，此前我的解决方案是替换 `.gitmodules` 中的 `url`。当然 work，但是过于浅显与侵入式，因此该方法并不好用。

前些天了解到 [fastgit](https://doc.fastgit.org/zh-cn/guide.html#web-%E7%9A%84%E4%BD%BF%E7%94%A8)，很好用啊，我当场就爱了。


| 站源 | 地址 | 缓存 |
| :---: | :---: | :---: |
| github.com | hub.fastgit.xyz | 无 |
| raw.githubusercontent.com | raw.fastgit.org | 无 |
| github.githubassets.com | assets.fastgit.org | 无 |
| customer-stories-feed.github.com | customer-stories-feed.fastgit.org | 480 分钟 |
| Github Download | download.fastgit.org | 480 分钟 |
| GitHub Archive | archive.fastgit.org | 无 |

就顺手看到他们仓库里的 `git` 换源教程，很方便，嗯，所以和之前的 `ghproxy` 一结合，成了。

附带，如果此后出现其他问题，也可以先用 `fastgit` 凑活着。
