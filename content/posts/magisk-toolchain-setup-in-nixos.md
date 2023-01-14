+++
title = "在 NixOS 里设置 Magisk 编译环境"
date = "2023-01-14T09:18:56+08:00"
author = ""
authorTwitter = "" #do not include @
cover = "https://s2.loli.net/2023/01/14/2uN5LKgRwtVDHIT.png"
tags = ["magisk", "nixos"]
keywords = ["magisk", "nixos"]
description = "当然不由虚拟机或是容器技术强力驱动！"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

在迁移到 NixOS 后，我面对的第一个开发问题是：Magisk 编译不动了 [^1]。

众所周知，NixOS 默认并没有安装 ```linker```。这会导致你在执行 Magisk 自带的编译工具链时，出现以下报错信息：

```shell
...
Error on executing binary file: No such file or directory
...
```

实际上，这个报错很具有迷惑性。它其实并不是：

> 系统找不到*你请求执行*的可执行文件

而是：

> 系统找不到*动态链接库程序 linker* 的可执行文件

详情可参考这个 [issue](https://github.com/NixOS/nixpkgs/issues/107375)。

解决方案也很简单，只要创建一个合适的环境就好了。[buildFHSUserEnv](https://ryantm.github.io/nixpkgs/builders/special/fhs-environments/) 可以胜任这个工作。

你可以在[我的这个 gist](https://gist.github.com/pokon548/334a92f21cbdb74705524ea012b20035) 里找到可以直接使用的成品。将它下载下来，并执行以下指令：

```shell
nix-shell magisk.nix
```

你就可以得到一个完备的 Magisk 开发环境。

[^1]: 事实上，早期我还未熟悉 NixOS 环境的时候，使用的是基于 LXC 技术的 Debian 容器，来完成编译 Magisk 的任务。然而，容器技术实在是过于重量级。在熟络了 NixOS 后，我便抛弃了这种做法，并顺便写下了这篇文章 :)