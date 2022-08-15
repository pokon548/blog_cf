+++
title = "从零开始的 Android Inline Hook —— 搭建开发环境"
date = "2022-08-14T16:15:15+08:00"
author = ""
authorTwitter = "" #do not include @
cover = "https://s1.ax1x.com/2022/08/14/vU4Y3n.jpg"
tags = ["Android", "Magisk", "Inline Hook"]
keywords = ["Android", "Magisk", "Inline Hook"]
description = "如何从白手起家，到开始使用 Inline Hook？"
showFullContent = false
readingTime = false
hideComments = false
+++

为方便以后开发 Inline Hook 模块，特在这里记录一下我从零开始使用 Inline Hook 技术的整个流程。

这是这个系列的第一部分。后续会随着我的研究进度，分享更多的内容。

## 技术栈
- Magisk，作为注入 Zygote 的底层框架；
- Zygisk，作为劫持 Zygote 的实际作用部分；
- [Dobby](https://github.com/jmpews/Dobby)，用来实现指令精度级的 Inline Hook。

## 开发环境
我使用的是 Android Studio 内含的 Emulator。从 Android 11 开始，x86_64 的镜像内含一个 ARM 指令转译模块 [^1]。在非必要的情况下，不必再使用 ARM 镜像开发应用（转译性能较低）。

如果你是 Windows 用户，也可以考虑使用 [MagiskOnWSALocal](https://github.com/LSPosed/MagiskOnWSALocal) 项目，将 WSA 变成一个完美的 Magisk 开发环境。


## 克隆 Magisk 仓库
> 下文假定你已经在 Emulator 里成功下载了相应的系统镜像，并拥有如何正确使用 Emulator 的经验。

Android Studio 内置的镜像自然是不包含 Magisk 的。我们需要使用 Magisk 项目自带的一个脚本，创建修改后的 ```ramdisk.img```。

使用下面的指令克隆 Magisk 仓库：
```shell
git clone --recurse-submodules -j8 https://github.com/topjohnwu/Magisk.git
```

带上 ``--recurse-submodules`` 的原因是 Magisk 项目本身会引用一些外部模块。``-j8`` 用于加速克隆速度。

## 初始化环境
克隆完成后，先在环境变量里设定 ``ANDROID_SDK_ROOT``，用于让 Magisk 相应脚本找到需要的依赖项目。

或者，如果不想因此而修改环境，你可以用一个取巧的方法：
```shell
ANDROID_SDK_ROOT=/to/sdk/root/directory python3 build.py <your_command>
```

克隆完成后，需要在 Magisk 根目录运行下面的指令，以初始化 Magisk 所需的 NDK 模块：
```shell
./build.py ndk
```

> 不要在 Android SDK Manager 里下载对应的 NDK。可能会导致意外的无法编译错误。

## 植入 Magisk
运行下面的指令：
```shell
./build.py emulator
```

你就能够获得一个一次性的 Magisk 开发环境。重启后环境会消失。

可能你不会想要这个命令，而是下面这个直接 patch ramdisk 的指令：
```shell
./build.py avd_patch /path/to/ramdisk.img
```

> 在成功执行这个命令后，Emulator 将自动重启。命令执行时间较长，请耐心等待。中间不会有太多输出。

之后，任意创建一个新的模拟器，并选中你 patch 后的 ramdisk，即可获得一个带有 Magisk 环境的模拟系统。

[![vUhIlq.png](https://s1.ax1x.com/2022/08/14/vUhIlq.png)](https://imgtu.com/i/vUhIlq)

至此，开发环境已经搭建完毕。

[^1]: https://android-developers.googleblog.com/2020/03/run-arm-apps-on-android-emulator.html