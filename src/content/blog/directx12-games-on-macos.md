---
author: Liya Du
pubDatetime: 2023-06-11T17:42:00Z
title: Play DirectX 12 games on macOS
postSlug: directx12-games-on-macos
featured: true
draft: false
tags:
  - macos
  - game
  - directx12
ogImage: "/assets/20230611/diablo4-gpt.jpg"
description: A tutorial in Chinese about how to play DirectX 12 games on macOS with Game Porting Toolkit beta released by Apple on WWDC2023.
---

今年 Apple 在 WWDC2023 上除了发布了最吸引眼球的 Apple Vision Pro。另一个对于广大苦于 macOS 孱弱的游戏生态的玩家可能比 Vision Pro 更重要的新闻是 Apple 低调发布了 Game Porting Toolkit，一个可以让开发者将 DirectX 12 游戏移植到 macOS 上的工具库。这个工具包含了一个 DX12 到 Metal 的转换器，以及一个 Metal 的运行时库。这个工具目前还在 beta 阶段，但是就目前测试下来的结果非常激动人心，在 Apple Silicon 上的 macOS 终于基本可以玩上最新基于 DirectX 12 的各种 3A 大作了！

网上目前已经有不少详细的英文教程，但还没有个详细的中文教程出来。所以大致总结了下步骤，以 Diable 4 为例，希望用 macOS 的玩家都能尽快玩上。

## Table of contents

## 准备环境

我使用的是 8 核 M1 Pro 的 MacBook Pro，macOS 版本是 Sonoma 14.0 Developer Beta。根据 Reddit 上的讨论，Ventura 13.4 也可以跑，但我还没有测试过。声音与外设直插 PS5 手柄都没有任何问题。

目前 Game Porting Toolkit 只支持 Apple Silicon 的硬件，Intel 的硬件暂时（也可能是永远）无法使用。

下面绝大部分步骤都需要在命令行下运行，在你的 macOS 上打开 [Terminal.app](https://support.apple.com/guide/terminal/open-or-quit-terminal-apd5265185d-f365-44cb-8b09-71a064a42125/)，或者 [iTerm2.app](https://iterm2.com/)。

### 安装 Command Line Tools for Xcode 15 Beta

从 [https://developer.apple.com/download/all](https://developer.apple.com/download/all/?q=xcode%20command%20line%20tools%2015) 上下载并安装 `Command Line_Tools_for_Xcode_15_beta.dmg`。

### 安装 Rosetta 2

[Rosetta 2](https://support.apple.com/en-us/HT211861)可以让你在 Apple Silicon 上运行 Intel 架构的应用。考虑到大部分 Windows 游戏都是基于 Intel x86-64 架构的，所以你需要安装 Rosetta 2。

打开 Terminal.app 或者 iTerm2.app，输入下面的命令：

```bash
softwareupdate --install-rosetta
```

安装非常快几秒钟就完事。中间需要你按`A`同意协议。而且会有些报错信息，但不影响成功安装。

```
I have read and agree to the terms of the software license agreement. A list of Apple SLAs may be found here: https://www.apple.com/legal/sla/
Type A and press return to agree: a
2023-06-09 15:35:19.198 softwareupdate[5164:89899] Package Authoring Error: 032-84877: Package reference com.apple.pkg.RosettaUpdateAuto is missing installKBytes attribute
Install of Rosetta 2 finished successfully
```

## 安装 Game Porting Toolkit

由于 Game Porting Toolkit 是在 Rosetta 2 下运行，下面基本所有步骤都需要在 Intel 兼容层的`bash`或者`zsh`环境下运行，后面我们都用 “Rosetta 2“ 环境代指。

### 切换 Terminal 到 Intel 架构

打开 Terminal.app 或者 iTerm2.app，输入下面的命令：

```bash
arch -x86_64 zsh
```

这样就可以在 Intel 架构下运行后续所有的命令了。如果你之前用`brew`安装过`zsh`的话，这里可能会报错：

```
admin@MacBook-Pro ~ % arch -x86_64 zsh
arch: posix_spawnp: zsh: Bad CPU type in executable
```

这时候你需要先卸载`brew`安装的`zsh`，然后再运行一遍命令即可。错误原因是`brew`只会安装默认架构（arm64）的包，而不会安装 x86-64 的。

```bash
brew uninstall zsh
```

### 安装 Intel 架构的 Homebrew

即使你之前安装过 Homebrew，大概率是 for arm64 架构的。所以你还需要为 x86-64 再安装一把。两个 Homebrew 会安装在不同目录，所以并不会冲突。

[Homebrew](https://brew.sh/) 应该不用多介绍了，已经算是 macOS 下默认意义的命令行包管理工具了。

首先确保你运行了上一趴命令切换到了 Rosetta 2 环境。运行如下命令安装 Homebrew：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

然后运行下面的命令，确保你接下去使用的是 Rosetta 2 下的 Homebrew：

```bash
which brew
```

```
admin@MacBook-Pro ~ % which brew
/usr/local/bin/brew
```

要是路径是`/opt/homebrew/bin/brew`，那就是 arm64 下的 Homebrew。

### 安装 Game Porting Toolkit Beta

先运行下面命令加上 Apple 的官方源：

```bash
brew tap apple/apple http://github.com/apple/homebrew-apple
```

然后最花时间的来了，下载和编译 game-porting-toolkit：

```bash
brew update && brew -v install apple/apple/game-porting-toolkit
```

整个过程在我的机器上花了大概 45 分钟，确保你的电池足够...

## 配置 Wine 和 Game Porting Toolkit

Apple Game Porting Toolkit 是基于 [Wine](https://www.winehq.org/) 和 [Crossover](https://www.codeweavers.com/crossover) 做的二次开发进行 DirectX 12 到 Metal 3 的直接转译。

### 配置 Wine

先在本地建个目录用于以后所有的 Windows 游戏：

```bash
mkdir -p ~/Games
```

然后我们准备把 Diablo 4 和 Battle.Net 都装在`~/Games/battle-net`下面，设定下 Wine 需要的环境变量：

```bash
export WINEPREFIX=~/Games/battle-net
```

然后打开 Wine 的配置窗口，我们要把模拟的 Windows 版本改成 Windows 10：

```bash
`brew --prefix game-porting-toolkit`/bin/wine64 winecfg
```

<Image src="/assets/20230611/wine-configurator-win10.png" format="png" width="480" alt="Wine Configurator" />

设置完成后点`Apply`然后`OK`退出。

### 在 Wine 中配置 Game Porting Toolkit

接下来我们要在 Wine 中安装 Game Porting Toolkit。先从[https://developer.apple.com/download/all/?q=game](https://developer.apple.com/download/all/?q=game)下载 `Game porting toolkit beta.dmg`，然后双击打开。

继续我们的命令行，把 Game Porting Toolkit 安装到 Wine 里面：

```bash
ditto /Volumes/Game\ Porting\ Toolkit-1.0/lib/ `brew --prefix game-porting-toolkit`/lib/
```

然后把 `Game porting toolkit beta.dmg` 里面的所有可执行文件都拷贝到 `/usr/local/bin` 下面，这样以后就不用每次都挂载 Game Porting Toolkit 的 dmg 文件了：

```bash
cp /Volumes/Game\ Porting\ Toolkit-1.0/gameportingtoolkit* /usr/local/bin
```

最后对于 Battle.Net，我们需要改下 Windows 注册表，让它正确识别这是个 Windows 10。分别以此运行下面三个命令，用来设置 Windows 10 的版本号（19042）：

```bash
`brew --prefix game-porting-toolkit`/bin/wine64 reg add 'HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion' /v CurrentBuild /t REG_SZ /d 19042 /f
```

```bash
`brew --prefix game-porting-toolkit`/bin/wine64 reg add 'HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion' /v CurrentBuildNumber /t REG_SZ /d 19042 /f
```

```bash
`brew --prefix game-porting-toolkit`/bin/wineserver -k
```

## 安装 Battle.net 和大菠萝 4

终于到了安装游戏环节了。我们以 Diablo 4 为例。其他如 Steam 步骤大同小异，但可能各自有一些设置，可以和参考下面参考链接章节。

从 [https://www.blizzard.com/download/](https://www.blizzard.com/download/) 下载 Battle.net 安装包。这边我们默认下载到了 `~/Downloads` 目录。

确认 Wine 下面可以知道下载下来的 `.exe`：

```bash
ls ~/Games/battle-net/drive_c/users/crossover/Downloads/
```

然后运行下面命令安装：

```bash
gameportingtoolkit ~/Games/battle-net ~/Games/battle-net/drive_c/users/crossover/Downloads/Battle.net-Setup.exe
```

这时候我遇到了一个问题，显示无法找到 `gameportingtoolkit`。可以把 `Terminal.app` 直接关闭，然后重新打开，
先进入 Rosetta 2 然后再运行一遍上面的命令。

然后就是一路 Next，安装完成后，打开 Battle.net，登录你的账号，然后安装 Diablo 4。

<Image src="/assets/20230611/diablo4-playing.jpg" format="jpeg" alt="Playing Diablo 4" />

下面的 [Playing Diablo IV on macOS](https://www.outcoldman.com/en/archive/2023/06/07/playing-diablo-4-on-macos/) 最后一个章节给出了一些 Diablo 4 里面的设置，可以参考下。

当需要运行游戏的时候可以直接用这个命令行：

```bash
PATH="/usr/local/bin:${PATH}" arch -x86_64 /usr/local/bin/gameportingtoolkit-no-hud ~/Games/battle-net  ~/Games/battle-net/drive_c/Program\ Files\ \(x86\)/Diablo\ IV/Diablo\ IV\ Launcher.exe
```

其中 `gameportingtoolkit-no-hud` 是不显示 HUD 的版本，如果你想看到 HUD 的话，可以用直接 `gameportingtoolkit`。

后面还有创建游戏快捷方式。我个人比较喜欢直接命令行，所以这步就略过了。另一个原因是目前 GitHub 上有不少人在做前端用来更方便的使用 Game Porting Toolkit，譬如 [Whihsky](https://github.com/IsaacMarovitz/Whisky/)，相信不用多久，各种易用的前端甚至于苹果官方的版本都会慢慢出现。

## 参考链接

- [Playing Diablo IV on macOS](https://www.outcoldman.com/en/archive/2023/06/07/playing-diablo-4-on-macos/)，主要步骤基本都是参考这篇文章，写的基本很详细了。
- [AppleGamingWiki - Game Porting Toolkit](https://www.applegamingwiki.com/wiki/Game_Porting_Toolkit)，各种额外 Q&A 信息，譬如如何解决 Steam 启动黑屏的问题。还有现有的一些游戏的兼容性列表。
- [Reddit /r/macgaming](https://www.reddit.com/r/macgaming/)，最新的讨论都可以在这个 Reddit 频道找到。
