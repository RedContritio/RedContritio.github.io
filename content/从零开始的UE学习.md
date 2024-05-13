---
title: "从零开始的UE学习"
date: 2024-04-21T19:14:27+08:00
draft: false
tags: ["AirSim", "UE"]
layout: post
math: false
toc: false
---

因为科研需求，需要用 UE 生成数据集，所以试图配置了一下，踩了一连串的坑。

<!--more-->

<!-- toc -->

# AirSim 不适配 UE5？

起手打开 Epic Game Launcher，点开引擎下载，默认推荐的版本是 **UE5.3.2**。

嗯，乍一看都推荐了，下载吧，下载大几十 G 之后，发现和 AirSim 不适配，无法顺利编译（前面流程正常，但是使用 VS 打开 sln 后，解析和编译都会出错）。

这时候搜了一下编译错误的问题，并未得到解决方案，但注意 MS/AirSim 仓库可以发现，最后的 release 大约在两年前，这时候搜索 "UE5 AirSim"，可以看到一个解决方案是 [Colosseum](https://github.com/CodexLabsLLC/Colosseum)。

因为 AirSim 停止更新，因此好事者接着做了 Colosseum 以支持 UE5，尝试了一下，仍然错误，这时候看 Colosseum 的 README.md，可以看到只支持到 UE5.2，好吧，重新安装 **UE5.2.1**，并使用 Colosseum，尝试构建，失败。

（注意，这里的构建描述的都是 `Unreal/Environments/Blocks` 构建）

到这里已经过去一天半的有效工作时间了。

不如安心用回 **UE4**？很自然的想法，绕开问题就没有问题。

安装了 **UE4.27.2**，并重新使用了 `AirSim`，编译执行，问题解决。

（如果使用现成框架，去适应它吧，除非有 **上司或者老板** 愿意为这份时间成本买单）

# 虚拟场景搭建

要仿真，先得有环境，很简单的思路是先用官方 `Blocks`，嗯，还不错，但是完全是白模，所以试试下载 MS/AirSim 里面的 `AirSimNH` 场景？小城镇街区，看起来还不错。

试玩之后发现，这个并不支持在编辑器中打开。

**这里我并不确定**，但是似乎没有源码会使模拟变得麻烦。

所以找了几个免费的环境（`Environments`）场景来测试看看各自的场景体验。

这里最终选择了四个室内场景：

- Cigar Room Environment
- Edith Finch : Barbara 的房间
- Edith Finch : Edie 的房间
- Edith Finch : Sam 的房间

我的做法是将 Edie 房间与 Sam 房间拼接，得到最终的多房间室内场景。

以及因为需求略微改动，现在不需要 AirSim 了，跳过这部分

# 重新构建光照出错？源码编译

这次学聪明了，先新建项目，然后把 Edie 的房间复制到我自己的主场景里，提示需要重新构建光照，

构建光照失败 Lighting build failed. Swarm failed to kick off

提示还表示，需要重建UnrealLightmass ？？？ FXXK YOU OFF，我从 epic launcher 里面下载的 UE4.27，怎么重建哦。

放弃了，看了网上的攻略，最终思路是老老实实源码编译。

首先去 Github 下载了 4.26.2 的 release 包，随后先执行 Setup.bat 脚本来获取依赖。

但是，UE4 的 Setup.bat 网络环境太差了！！！（也可能是因为我这边的环境差）

虽然这是自动带断点了，但是不断重新执行 Setup.bat 也很败坏心情，能不能自动化呢？可以。

首先打开 Setup.bat，把里面最后一部分，从

```bat
rem Done!
goto :end

rem Error happened. Wait for a keypress before quitting.
:error
pause      # 修改这行哦

:end
popd
```

改成

```bat
rem Done!
goto :end

rem Error happened. Wait for a keypress before quitting.
:error
exit /b -1

:end
popd
```

这样执行失败时，不再需要手动按键，并且返回值不为 0，用来判断是否正常退出的。

接着，我们使用 powershell 打开，执行这段用来不断重试：

```powershell
do {
    # 执行 Setup.bat 文件
    $result = Start-Process -FilePath "Setup.bat" -Wait -PassThru

    # 检查返回值
    if ($result.ExitCode -eq 0) {
        Write-Host "Setup.bat 执行成功!"
    } else {
        Write-Host "Setup.bat 执行失败，返回值: $($result.ExitCode)"
    }

} while ($result.ExitCode -ne 0)

```

这样就会自动不断重试执行 Setup.bat，直到正常返回（进入 `:end` 分支）
