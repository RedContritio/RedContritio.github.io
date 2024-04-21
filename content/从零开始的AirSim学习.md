---
title: "从零开始的AirSim学习"
date: 2024-04-21T19:14:27+08:00
draft: false
tags: []
layout: post
math: false
toc: false
---

因为科研需求，需要用 AirSim 生成数据集，所以试图配置了一下，踩了一连串的坑。

<!--more-->

<!-- toc -->

# UE5？不适配！

起手打开 Epic Game Launcher，点开引擎下载，默认推荐的版本是 **UE5.3.2**。

嗯，乍一看都推荐了，下载吧，下载大几十 G 之后，发现和 AirSim 不适配，无法顺利编译（前面流程正常，但是使用 VS 打开 sln 后，解析和编译都会出错）。

这时候搜了一下编译错误的问题，并未得到解决方案，但注意 MS/AirSim 仓库可以发现，最后的 release 大约在两年前，这时候搜索 "UE5 AirSim"，可以看到一个解决方案是 [Colosseum](https://github.com/CodexLabsLLC/Colosseum)。

因为 AirSim 停止更新，因此好事者接着做了 Colosseum 以支持 UE5，尝试了一下，仍然错误，这时候看 Colosseum 的 README.md，可以看到只支持到 UE5.2，好吧，重新安装 **UE5.2.1**，并使用 Colosseum，尝试构建，失败。

（注意，这里的构建描述的都是 `Unreal/Environments/Blocks` 构建）

到这里已经过去一天半的有效工作时间了。

不如安心用回 **UE4**？很自然的想法，绕开问题就没有问题。

安装了 **UE4.27.2**，并重新使用了 `AirSim`，编译执行，问题解决。

（如果使用现成框架，去适应它吧，除非有 **上司或者老板** 愿意为这份时间成本买单）
