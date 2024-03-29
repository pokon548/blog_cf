+++
title = "绕过限制，为 Supernote 安装第三方程序"
date = "2022-08-06T13:12:14+08:00"
cover = "https://s1.ax1x.com/2022/08/06/vuF5sP.jpg"
tags = ["supernote", "sideload", "hack"]
keywords = ["supernote", "sideload", "hack"]
description = "把你的 Supernote 变成一个又能写字，又能轻办公的墨水篇屏平板，岂不是非常爽？"
+++

> 事先提示：操作有风险。这么做可能会破坏 Supernote 的保修策略，也可能会导致你的 Supernote 无法正常使用。操作前请三思，确认自己是否有足够的动手能力。

不过，能够点进这篇文章的人，应该都是拥有动手能力，而且知道自己在做什么的超级用户吧？那么，废话不多说，直接开始吧。

# 原理
- Supernote A5X / A6X 的固件基于安卓系统实现；
- 可以在没有任何限制的情况下，直接使用已经解锁的 fastboot；
- 内置的 Recovery 允许 adb shell 直接拿到 root 权限。借此可远程提取并修改 Supernote 里的任意文件。

## 操作步骤
以下步骤并不复杂，但是需要你有一定的 Android 刷机经验。

### 开启 ADB 侧载应用权限
即使是已经解锁，你也暂时无法使用 ```adb install xxx.apk``` 安装你喜欢的东西。如果现在就尝试的话，会得到 ```command not supported``` 的提示。
这时，我们就需要修改对应的配置文件，以开放侧载权限。

输入 ```adb reboot recovery```，把 Supernote 置入到恢复模式当中。保持连接，继续输入以下指令：
```
adb shell
```
你就成功的在电脑终端里拿到了一个带 root 的 shell。神奇吧！

如果你熟悉 ```vi``` 的话，其实就不用继续看下面的步骤了。只需要[^2]：
1. 打开系统分区里的 ```prop.default```；
2. 将 ```ro.secure=1``` 修改为 ```ro.secure=0```；
3. 将 ```ro.debuggable=0``` 修改为 ```ro.debuggable=1```。

之后，重启到系统内，并愉快的使用 ```adb``` 侧载任意你想要的东西吧。

当然，如果你不会使用 ```vi```，请继续看下面的简单教程。

如果你之前输入了 ```adb shell``` 并拿到了远程终端，请输入一次 ```exit``` 以退出。我们会把这个配置文件拿到电脑上来编辑，再放回去。

要这么做，需要输入这个指令：
```
adb pull /system/etc/prop.default .
```
你就会在当前目录下拿到一份配置文件的副本。用你任意喜欢的文本编辑器打开它，找到并替换以下几项：
- ```ro.secure=1``` 修改为 ```ro.secure=0```；
- ```ro.debuggable=0``` 修改为 ```ro.debuggable=1```。

改完别忘了保存。

之后，我们用 ```adb push``` 指令把这个修改了的文件推回去：
```
adb push prop.default /system/etc
```

完成后，输入 ```adb reboot``` 回到系统。此时，你就可以安装任意想要的程序了！

## 后续步骤
到这里为止，你就可以在 Supernote 上安装任意想要的应用程序了！不过，或许你还会在意一些特定的的使用问题。我结合自己的使用体验，写了这些建议，随意参考 :)。

### 安装应用商店
为了方便后续使用，你可以现在就在 Supernote 上安装一个应用商店，以随时随地安装（和卸载？）众多应用程序。我选择的是 ```Aurora Store```，一个开源的 ```Google Play``` 商店替代。你可以在[这里](https://auroraoss.com/) 下载到它。

下载完毕后，使用下面的指令将 APK 安装到 Supernote 里：
```
adb install <你 APK 的名字>.apk
```

安装完成后，可以在 Supernote 的侧边栏里找到它。

### 安装悬浮球软件
截止到 ```Chauvet 2.5.17```，Supernote 并未对第三方程序进行任何适配。也就意味着，你在使用第三方应用时，会遇到一个尴尬的问题：```没有返回键```。换言之，没法好好使用很多程序。

或许你会说，我直接把 Android 自带的底部导航栏开出来不就好了[^3]？遗憾的是，Supernote 似乎阉割了这个选项。即使你在 ```build.prop``` 里指定了开启它的键值对，也无法使用导航栏。

怎么办呢？也许可以考虑安装一个可以模拟返回键的悬浮球软件。

我选择的是```简悬浮```，你可以在[这里](https://www.coolapk.com/apk/com.bs.smarttouch) 下载它[^4]。安装完成后，给予它```无障碍```和```显示在上层```的权限即可开始使用。

### 安装文件管理器
Supernote 自带的文件管理器只显示特定的文件夹，不会显示整个根目录下的内容。

如果你和我一样，希望手动管理一切，可以考虑安装一个第三方的文件管理器，便于使用。

### 安装 Magisk
想要在 Supernote 上体验操纵一切的快感[^5]？你或许会想要给它 root 一下。怎么操作呢？

首先，还是重启到 Recovery。之后，用这个指令把 ```boot.img``` 捞出来：
```
adb pull /dev/block/by-name/boot .
```

你就得到了一份完整的 boot 镜像。至于怎么 ```patch ramdisk```，我假定读到这里的读者都拥有这项技能。哈哈。

完成后，将 Supernote 重启到 ```fastboot``` 模式，输入以下指令刷写新的启动镜像：
```
fastboot flash boot magisk_patched_*******.img
```

完成，重启，收工。

### 侧边栏应用设置
你或许会安装许多 ```Set it and forget it``` 类应用。换言之，它们在安装并设定完成后，就不再多加配置了。此时，你或许会想把这些程序从侧边栏里划出去，避免占用宝贵的列表空间[^6]。

导航到 ```设置``` -> ```应用``` -> ```侧滑栏标签配置```，点击对应应用的⛔图标，即可将其移出侧边栏。

### 不兼容的应用
有一些程序是没法在 Supernote 定制系统上跑的。这是我自己尝试的程序，不一定全面：
- 基于 Chromium 内核的主流浏览器。我测试了 ```Bromite```、```Chrome```，都会在完成初始化设置后闪退。连接电脑看日志，发现似乎是 Supernote 删除了某个底层的绘制 API，使得浏览器无法正常渲染内容。解决方法：安装 ```Firefox```，或者是 ```Via``` 这种没有自带内核的浏览器[^7]；
- VPN。确认连接的对话框被删除了，因此你无法授权任何 VPN 连接[^8]；
- 需要 Google 框架的应用。因为许可原因，Supernote 没有内置 GApps，自然也就无法使用依赖于谷歌框架的应用。

### 拓展阅读
- [Hacking and rooting the Ratta Supernote A5 X (and probably A6 X)](https://github.com/TA1312/supernote-a5x)[^9]


[^1]: 神奇的是，Supernote 在执行完这个命令后，不会依照 Google 的标准执行恢复出厂设置的操作。也算是个好事，避免了备份和还原数据的麻烦
[^2]: 如果直接在终端里输入 ```vi``` 并按下回车的话，你会惊奇的发现没有这个指令。试试看在指令的前面加一个 ```toybox```，你会回来感谢我的 :)
[^3]: Chauvet 基于 Android 8.1 定制而来
[^4]: 当然，你可以随便选择其它的应用程序。我选择这个程序，充其量只是因为它简洁无广告罢了
[^5]: 中二症晚期症状
[^6]: Supernote 侧边栏的分页功能做的似乎不怎么好用。大量的应用会产生导航困难
[^7]: 神奇的是，内置的 Android System Webview 又是可以正常使用的（基于 Chromium）。当然，或许你会尝试升级它。我的建议是：放弃，因为一样会闪退，而且会把 Supernote 里面的一些程序给弄崩
[^8]: 但是 VPN 本身似乎是可以用的，只是无法创建新连接而已。不知道有没有什么可以绕过的手段
[^9]: 虽然看起来本文章是从这里借鉴过来的，但这只是个巧合 :)