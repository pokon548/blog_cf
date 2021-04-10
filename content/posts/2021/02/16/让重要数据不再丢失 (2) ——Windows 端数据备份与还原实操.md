---
title: "让重要数据不再丢失 (2) ——Windows 端数据备份与还原实操"
date: 2021-02-16T19:13:17+08:00
draft: false

description: "终于不用担心系统坏掉之后从零开始的 Windows 生活了！"
---
在上一篇文章中，我们稍微的了解了备份的理论指引。那么，要如何落到实处呢？这就是本篇博文要讲述的东西了。
篇幅略长，请~~自备爆米花~~做好长时间阅读的准备。

## 备份 Windows 本体
首先，Windows 自 7 时代就自带了一个相当好用的系统备份工具：备份与还原。~~比 Ghost 什么的好用一万倍~~ 我将其用于系统本体的备份。
虽然这个工具看上去有点老，但你依然可以在 Windows 10 里找到它。方法如下：

1. ```Win + R``` 启动“运行”，里面输入```control```并回车，打开控制面板。
[![ycRlPH.png](https://s3.ax1x.com/2021/02/16/ycRlPH.png)](https://imgchr.com/i/ycRlPH)

2. 点击“系统与安全”下面的“备份与还原(Windows 7)”。
[![ycRtqf.png](https://s3.ax1x.com/2021/02/16/ycRtqf.png)](https://imgchr.com/i/ycRtqf)

3. 点击“设置备份”。
[![ycR0iQ.png](https://s3.ax1x.com/2021/02/16/ycR0iQ.png)](https://imgchr.com/i/ycR0iQ)

之后，Windows 应该会弹出一个对话框，让你选择备份内容存储的位置。
[![ycR2ZT.png](https://s3.ax1x.com/2021/02/16/ycR2ZT.png)](https://imgchr.com/i/ycR2ZT)

我这里为演示起见，选择了同一硬盘上的不同分区作为存放位置，**但实际备份时，不建议这么操作！**
> 我会说我只是因为移动硬盘忘了拿回家了吗（x

之后，Windows 会让你选择需要备份的内容。由于我自己重定向了个人文件夹的位置到 D 盘，而且所有重要文件都在里面，因此没有自定义其它备份路径的需求。我就直接选择了默认选项。你可以按照自己的实际需求“让我选择”。
[![ycRTQ1.png](https://s3.ax1x.com/2021/02/16/ycRTQ1.png)](https://imgchr.com/i/ycRTQ1)

这里是最后一步。当确认备份设定无误后，你就可以大胆的按下“保存设置并运行备份”！
[![ycRjFe.png](https://s3.ax1x.com/2021/02/16/ycRjFe.png)](https://imgchr.com/i/ycRjFe)

备份就开始了哦~
[![ycWHpj.png](https://s3.ax1x.com/2021/02/16/ycWHpj.png)](https://imgchr.com/i/ycWHpj)

备份完成后，你应该就可以在相应磁盘里找到相应的系统镜像备份了。

## 制作应急启动盘
由于镜像的特殊性，在还原镜像时，会格式化**包括 Recovery** 在内的系统磁盘分区。这也意味着，如果需要还原，你将无法借助“高级重启菜单”启动还原工具，顺利的完成还原操作。
![有可能会变成这样（哭）](https://filestore.community.support.microsoft.com/api/images/1e4c0d4c-5308-45ac-b3ec-7d0537256430?upload=true)

那么，怎么制作安装盘呢？老毛桃？微 PE？No。
> ### 为什么不选择上面所说的制作工具
>
> 最主要的原因还是它们加料太多。它们都有篡改还原后系统浏览器主页的丑闻。同时，你很难分辨出你还原的系统是否会被这类启动盘制作工具篡改。次要原因嘛，就是下面有一个更好用的制作工具！也是国人开发的哦~

我们选择 Ventoy。一个**支持直接启动 ISO、wim 等**的多系统启动盘制作工具。
![截图](https://www.ventoy.net/static/img/screen/screen_bios2.png)

之所以选择这个，是因为：
- 开源软件。放心透明
- 随意放启动镜像。换言之，只要你的 U 盘撑得起你的野心，你可以内装无数个维护和安装镜像！而不需要每次更换维护镜像时重置 U 盘
- 软体小巧，功能强大。10 MB 的 exe 就可以做到这一切~
- 升级不格式化 U 盘！

你可以直接在[这里](https://www.ventoy.net/cn/download.html)下载到 Ventoy 的最新版本。下载的同时，请同时准备好一个 U 盘~

下载完毕，解压 zip，双击打开...
[![ychvYF.png](https://s3.ax1x.com/2021/02/16/ychvYF.png)](https://imgchr.com/i/ychvYF)

嗯。简洁的界面。
[![yc4U6s.png](https://s3.ax1x.com/2021/02/16/yc4U6s.png)](https://imgchr.com/i/yc4U6s)

> ### 请备份 U 盘内重要数据！
>
> 首次制作会**清除 U 盘内所有数据**！如果里面有重要数据，请提前备份。
> 后期升级不会影响 U 盘内数据。且除开 Ventoy 本体，剩下的可用空间非常大，你可以直接把它当成一枚正常的 U 盘日用。但应急的时候，它还能派上普通 U 盘到不了的用场~

当软件识别到你的 U 盘后（没有识别得到的话，戳几下右上角绿色的刷新按钮试试？），你就可以直接点“安装”将 Ventoy 安装到你的 U 盘里了。

> ### 警告！备份数据！
>
> 软件在此时会弹出两次黄色警告框，告诉你接下来的步骤会**丢失 U 盘内的所有数据**！
> [![yc4otO.png](https://s3.ax1x.com/2021/02/16/yc4otO.png)](https://imgchr.com/i/yc4otO)
> 
> 请再次确认你没有重要数据还未从 U 盘里备份出来。
> 备份完毕并确认无误后，两个对话框都按“是”即可继续安装。

安装完成后，你的 U 盘会多出一个叫"Ventoy"的空分区。

> 由于我自己已经用了很长时间，因此截图里不是空的（笑）

[![yc5SN8.png](https://s3.ax1x.com/2021/02/16/yc5SN8.png)](https://imgchr.com/i/yc5SN8)

接下里，你就可以把 ISO，wim 等安装镜像文件直接拷到这个分区里了。位置不重要—— Ventoy 在开机时会自动遍历该分区目录下的所有子文件，并将其自动添加到启动菜单里（方便吧？）。

> ### Wim 启动支持需额外插件
>
> 请在[这里](https://www.ventoy.net/cn/plugin_wimboot.html)下载该插件，并放到分区下的指定位置，方可在启动菜单里支持 wim 开机选单。

至于你说放些什么好？对本文而言，[Windows 10 安装镜像](https://www.microsoft.com/zh-cn/software-download/windows10)足以！因为里面本身就自带一个 WinRE，可以用来还原系统~（不过很多人可能不知道就是了）

## 如何还原
首先，从 U 盘启动，选择 Windows 10 安装镜像。启动。

> 为了方便截图，以下步骤均在 VirtualBox 里演示！实机操作也是一样的~

进入到了熟悉的紫色安装界面~ 我们在这里先选择“下一步”。
[![ycoSSg.png](https://s3.ax1x.com/2021/02/16/ycoSSg.png)](https://imgchr.com/i/ycoSSg)

然后，~~不是点击“现在安装”！~~ 点击左下角的“修复计算机(R)”
[![ycoimn.png](https://s3.ax1x.com/2021/02/16/ycoimn.png)](https://imgchr.com/i/ycoimn)

疑难解答，系统映像恢复...
[![ycoJfO.png](https://s3.ax1x.com/2021/02/16/ycoJfO.png)](https://imgchr.com/i/ycoJfO)

[![ycotpD.png](https://s3.ax1x.com/2021/02/16/ycotpD.png)](https://imgchr.com/i/ycotpD)

接下来就是一路傻瓜式操作，就不继续演示了。
[![yco47q.png](https://s3.ax1x.com/2021/02/16/yco47q.png)](https://imgchr.com/i/yco47q)

> 其实是我没有足够大的外接硬盘放演示用系统镜像，导致演示不下去了（Em

当操作正确无误后，WinRE 会要求重启。重启后，系统也就恢复到先前备份时的状态了~
