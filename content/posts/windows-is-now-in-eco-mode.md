+++
title = "发挥 Windows 真正的续航能力：EnergyStar"
date = "2022-07-31T22:02:34+08:00"
author = ""
authorTwitter = "" #do not include @
cover = "https://s1.ax1x.com/2022/08/06/vu5Yi4.jpg"
tags = ["Windows", "Eco"]
keywords = ["Windows", "省电"]
description = "原来 Windows 本本可以这么省电啊？"
showFullContent = false
readingTime = false
hideComments = false
+++

> 该应用**不适用**于 Windows 10。要使用这个应用，你还需要 Intel 十代以上或 AMD Ryzen 5000+ 的 CPU。在 Windows 11 22H2 上效果最佳。

众所周知，Windows + X86 CPU 的笔记本续航能力一言难尽。除非是性能低下的超薄本，否则你几乎不可能期待 Windows 能在没有外置电源的情况下，保持出色的续航能力。然而，这款应用可以利用 Windows 在 Insider 版本中引入的 [Windows EcoQoS API](https://devblogs.microsoft.com/performance-diagnostics/introducing-ecoqos/)，让 Windows 大幅限制：
- 后台应用
- 不活跃的前台应用

的活动，以获得令人惊讶的续航能力。

## 到底有多强
简单来说，原本 Windows 本本可能只有不到两小时的续航时间。开了这个之后，你可以获得大概三四个小时的亮屏时间。

几乎可以说是续航翻倍了。而且也因为它限制的不是整体性能，在实际日用时，体验差异几乎不可感知。可以说是很舒服了。

当然，这是我自己笔记本的测试结果。你还可以到 [Twitter](https://twitter.com/imbushuo/status/1553293329403559937) 上观看原作者的续航测试推文。

## 下载地址
[GitHub](https://github.com/imbushuo/EnergyStar)

作者是现任微软的 imbushuo 以及其他一些人。感谢他们的贡献！

## 怎么用？
下载，解压到你喜欢的位置，双击 exe 运行。That's it.