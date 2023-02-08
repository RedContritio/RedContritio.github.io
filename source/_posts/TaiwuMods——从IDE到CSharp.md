---
title: TaiwuMods——从IDE到CSharp
date: 2020-03-26 23:45:16
tags: VS CSharp Mods Environment
# cover: https://tva2.sinaimg.cn/large/9bd9b167ly1g2qjwnq07hj21hc0u07ct.jpg
---

## 前情提要

因为一直怀疑VS装的有问题，所以打算卸载重装一下试试。

然后，以前的项目就打不开了。（指 Taiwu_Mods）

因此，大致记录一下过程，方便以后自己查。
<!--more-->

## 安装VS

如题，不做描述。

需要特别注意的一点是，msbuild要构建C#项目，可能会出现找不到CMAKE_CSharp_COMPILER的情况。

参考 [StackOverflow上的某篇教程](https://stackoverflow.com/questions/51668676/cmake-visual-studio-15-2017-could-not-find-any-instance-of-visual-studio)，可以得知，需要安装 Desktop development with C++ 组件。

也就是说，为了用cmake构建CSharp项目，需要VS中的.Net组件和桌面C++组件。（大概10G）

