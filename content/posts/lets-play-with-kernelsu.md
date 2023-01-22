+++
title = "和 KernelSU 一起愉快的玩耍"
date = "2023-01-22T19:30:04+08:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["kernelsu", "lime"]
keywords = ["kernelsu", "lime"]
description = "经过一周多的努力，终于将 KernelSU 移植到红米 Note 9 4G 上面了！"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

[KernelSU](https://kernelsu.org) 是一个全新的 Root 实现。它将授权部分的代码集成到了内核里。相比起已有的用户态 Root 授权方案，它有更好的隐蔽性和安全性。

在该项目正式推出后一天，我便开始着手为 Redmi Note 9 4G 适配一款可以使用的 KernelSU 内核。

由于该项目初期并未向 4.14 内核进行特殊适配，因此在移植的过程中遇到了一些困难。随着项目本身的完善，我在今天终于成功编译出了一款可以使用的内核。

该内核基于 ShelbyHell 制作的 [Sunrise Kernel](https://github.com/ShelbyHell/sunrise_kernel_juice)。我在其基础上加入了针对 KernelSU 的内核代码。

你可以在 [GitHub](https://github.com/pokon548/kernelsu-xiaomi-juice) 找到 Actions 源代码，以及可以使用的成品。

## 兼容性
只在 AOSPA topaz（Android 13）上测试通过。其它 ROM 请自行测试。