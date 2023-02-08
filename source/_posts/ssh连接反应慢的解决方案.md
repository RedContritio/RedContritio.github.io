---
title: ssh连接反应慢的解决方案
date: 2021-11-05 15:53:31
tags: ssh sshd 服务器 网络
# cover: https://tva2.sinaimg.cn/large/9bd9b167gy1g2qk3q3oboj21hc0u0k4b.jpg
---

## 发现问题

最近几次走 ssh 连接树莓派的时候，经常会发生连接速度较慢的问题，表现为每次按键后需要几秒时间才能反应过来，甚至直接断开连接，例如[局域网ssh延迟](https://serverfault.com/questions/961576/ssh-lag-in-lan-on-some-machines-mixed-distros)这篇中所述的情况。

<!--more-->

在发生延迟后，对树莓派ip进行一个的ping，然后，嘿嘿，ping炸了。

简单来说，ping 的时候，丢包率达到了***喜人的 87%*** 以上。

哇，那怎么会这么高呢？

通过直接在树莓派上操作，网络什么也都正常。

唔，所以大致是 ping 不稳定，导致的 ssh 连接不稳？

## 定位问题

首先，在[网上](https://www.annhe.net/article-4504.html)看到了一个有趣的关键词：

**高效能以太网**

> 高能效以太网（英语：Energy-Efficient Ethernet，简称EEE）是一套对双绞线与计算机网络标准之以太网家族的背板的增强，使其在低数据活动期间消耗较少的功率。
> 
> 其目标是将功耗降低50%以上，同时保持与现有设备的完全兼容。
> 
> 它的功率降低以几种方式实现。在100 Mbit/s、1吉比特和10 Gbit/s速度的数据链路中，物理层发送器会始终使用能量。
> 
> 在没有数据发送时，它们可以进入“睡眠”模式以节约能源。
> 当控制器软件或固件确定不需要发送数据时，它可以发出一条“低功耗闲置”（LPI）请求到以太网控制器的物理层PHY。PHY然后将LPI信号在特定时间发送到链路上，以及禁用发送器。
> 
> 信号刷新将周期性地发送以维持链路信令的完整性。当需要发送数据时，将在预定时间段发送IDLE信号。
> 数据链路可以被视为始终在运行，因为即使发送路径处在睡眠模式，接收信号的电路仍保持活跃。

综上所述，可以看出，**EEE会采用降低发送频率的方式节能**。

执行该命令，查看是否启用了 EEE。

```bash
ethtool --show-eee eth0
```

通过观察可以看到，输出中，有 

> EEE status: enabled - active

因此判断出，问题出现在 EEE 上。

## 解决问题

```bash
sudo ethtool --set-eee eth0 eee off
```

执行后再次检查 EEE 状态，可以看到其已关闭（disabled）。

为了保险，进行

```bash
sudo service sshd restart
```

从而解决问题，丝滑 ssh。

## 长期配置

可以将其加入开机启动项中，例如在 Raspberry 4B 上，可以将命令加入到 `/etc/rc.local` 中。

加入的命令如下：

```bash
sudo ethtool --set-eee eth0 eee off
```