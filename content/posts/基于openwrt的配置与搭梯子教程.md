---
layout: post
tags: ["openWrt","clash", "openclash", "网络配置"]
title: "基于openwrt的配置与搭梯子教程"
date: 2024-03-08T14:08:16+08:00
math: false
draft: false
toc: true
---

因为某些技术性故障，导致路由器崩掉了，重置且配置梯子好麻烦啊，所以写个备忘录。

<!--more-->

<!-- toc -->

刚初始化的时候，密码默认是 "" 或者 "password"，直接输入进入即可。

## 拨号

可以说是最简单的步骤了，首先进入 openwrt，在顶部菜单栏 Network -> Interfaces 进入后，在 Interfaces 选项卡中，找到 WAN 这一行。

在最右侧的 "Edit" 按钮打开编辑页面，"Protocol" 选择 "PPPoe"，随即下面会出现一行 "Really switch protocol?"。

选择按钮 "Switch Protocol"，并填入 "PAP username" 和 "PAP password"，点击右下角 "Save" 保存回到 Interfaces 选项卡。

**最后记得点选页面右下角 "Save & Apply" 按钮，应用改动。**

回合结束，现在可以联网了！

## opkg 换源

对没错，虽然我们要搭梯子，但是前期还是要先换源。

打开顶部栏 System -> Software。在顶部栏最右侧选择 "Configure opkg"。

最下面是软件源 "/etc/opkg/distfeeds.conf"，其中内容如下：

```conf
src/gz openwrt_core https://downloads.openwrt.org/releases/22.03.0/targets/mediatek/mt7622/packages
src/gz openwrt_base https://downloads.openwrt.org/releases/22.03.0/packages/aarch64_cortex-a53/base
src/gz openwrt_luci https://downloads.openwrt.org/releases/22.03.0/packages/aarch64_cortex-a53/luci
src/gz openwrt_packages https://downloads.openwrt.org/releases/22.03.0/packages/aarch64_cortex-a53/packages
src/gz openwrt_routing https://downloads.openwrt.org/releases/22.03.0/packages/aarch64_cortex-a53/routing
src/gz openwrt_telephony https://downloads.openwrt.org/releases/22.03.0/packages/aarch64_cortex-a53/telephony
```

我们不妨换成清华源，将其中的 "downloads.openwrt.org" 替换为 "mirrors.tuna.tsinghua.edu.cn/openwrt" 即可。

需要注意，这里与 openWrt 的版本相关，因此不要直接复制最终结果，而是确认版本是什么再替换这个字符串。

替换后内容如下：

```conf
src/gz openwrt_core https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.0/targets/mediatek/mt7622/packages
src/gz openwrt_base https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.0/packages/aarch64_cortex-a53/base
src/gz openwrt_luci https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.0/packages/aarch64_cortex-a53/luci
src/gz openwrt_packages https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.0/packages/aarch64_cortex-a53/packages
src/gz openwrt_routing https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.0/packages/aarch64_cortex-a53/routing
src/gz openwrt_telephony https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.0/packages/aarch64_cortex-a53/telephony
```

随后右下角 "Save"，在 Software 面板的顶部栏右侧选择 "Update lists" 按钮，更新软件源信息。

如果正常，输出如下：

```
Downloading https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.0/targets/mediatek/mt7622/packages/Packages.gz
Updated list of available packages in /var/opkg-lists/openwrt_core
Downloading https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.0/targets/mediatek/mt7622/packages/Packages.sig
Signature check passed.
```

关键在于有若干条 "Signature check passed."，即说明更新软件源成功。

## openclash 依赖

openclash 依赖大部分都很简单，但是特别的是，需要卸载 dnsmasq，改为安装 dnsmasq-full。

如果在命令行，直接执行

```sh
opkg remove dnsmasq && opkg install dnsmasq-full
```

如果在图形界面，这里比较简单，略过不表。

后续安装使用过程参考 [openclash 安装教程](https://blog.hellowood.dev/posts/openwrt-安装使用-openclash/) 即可。

特别的，openclash 的包建议下载最新版，且先下载到本地后，上传到 openWrt 使用。

（因为尚未配置梯子时，github 网速当前日益变差）

## 关闭 ipv6 dhcp

因为 openclash 和 ipv6 dhcp 不适配，所以只能先关闭 ipv6 dhcp。

参考教程 [Openwrt关闭IPV6的方法](https://www.kancloud.cn/bigdongdong%20/armgear/3071459)
