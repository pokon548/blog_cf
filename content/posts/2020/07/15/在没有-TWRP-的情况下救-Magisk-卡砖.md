---
title: "在只有 ADB 能用的 Android 10+ TWRP 下救 Magisk 卡开机"
description: "Android 10 + TWRP = 救砖无路？不存在的 2333"
date: 2020-07-15T17:51:21+08:00
---
> 原本我以为[一加](https://www.oneplus.com)是一个适合刷机的牌子。现在看来我错了，~~一个不恰当的 Magisk 模块就能要了我的命~~（

不过有一说一，这也不能全怪一加。由于自 Android 10 开始，Google 向 AOSP 引入了[动态分区](https://source.android.google.cn/devices/tech/ota/dynamic_partitions/implement)机制，导致所有依赖于 [TWRP](https://twrp.me) 刷机的机油们都不好过。至于具体原因，可以看 TWRP 官方的说明：[点这儿](https://twrp.me/site/update/2019/10/23/twrp-and-android-10.html)。

简而言之，就是 TWRP 还没有找到能在 Recovery 阶段解读动态分区机制的办法。现有的 TWRP 原版解读不了动态分区上的所有逻辑分区。分区都动不了，刷机也就无从谈起。

不过还好，我们还有倒数第二重利器：Fastboot。虽然 Google 搞事把分区整个重写了一遍，但总不能把自家车门也焊死了吧。而 Fastboot 也是这篇文章，以及现在给使用了动态分区机制的机子刷机的主要思路。官方对改写的 Fastboot 协议（fastbootd）解释在此：[点我](https://source.android.google.cn/devices/tech/ota/dynamic_partitions/implement#fastbootd)

## 拯救砖头，刻不容缓（不是
在尝试了一阵子之后，我逐渐发现了一件事：TWRP 下虽然几乎所有功能残废，但是！有一个功能还是能用 — ADB。

你也许会说：唉几乎所有分区都废掉了，系统都炸穿了，要它何用？但别着急嘛。我只是说几乎所有分区都废掉了，**可没说所有分区都废掉了**。

比如说，我们打开半残 TWRP 的终端，输入这个指令：``` ls /dev/block/by-name/* ```。你会惊奇的发现：块设备居然还能列得出来。哎！原来 **dev** 分区还是好的。那，这又有什么作用呢？如果你精通 Unix 系的其中一哲学，你很快就会知道我为啥要想到这个。不过不知道也没关系，在这里现学现用也不见得是件坏事：

> In UNIX, everything is a file. -- [Wikipedia](https://en.wikipedia.org/wiki/Everything_is_a_file)

在远古时期的 Unix 和流淌着新鲜血液的 Linux 身上，都拥有着这个哲学（或者说，设计理念）：一切都是文件。

既然一切都能是文件，那么我们不妨大胆猜测，在 Linux 下：
- 文件夹是一种文件
- 我的鼠标也被视作一种文件
- 我的硬盘分区也能被视作一种文件
- ...

那么，以上的推测是否正确呢？完全正确！不过由于本篇博客的著述是针对 Android 救砖的，就不展开说明它们了。对具体原理感兴趣的同学可以 Google 一下。关键字：块文件。

## 略见曙光
也就是说，我们现在可以读取到手机的部分分区。而它们作为一种文件，也就理所应当的具备了能复制的特性。

通过复制 -> 得到了分区文件 -> 不就等于得到了分区镜像嘛！但问题来了。要怎么才能把这好不容易得来的功夫付诸于电脑，让美好发生呢？这就要让 ADB 隆重登场的时间了。

ADB 在这里虽然残废，但有一个指令不仅不残废，还很好用：
``` adb pull /path/to/file ```

pull 指令的意思是：从手机里**复制指定目录的文件到电脑上**。恍然大悟？知道为啥要介绍 ADB 了吧。人家可还是有点东西的呢——用着指令，你就有能在电脑上打补丁的材料了啊。

既然如此，那还不赶紧用上这给力的工具。输入这串指令：

``` adb pull /dev/block/by-name/boot-* . ```

就可以把 boot 镜像拿到电脑上了。

> ## 双分区手机请注意你当前的槽位设定
> 由于双分区设计的手机每次刷机时，使用的分区可能不一样（A 或者 B）。因此，在拷贝镜像前，请先在 TWRP 的重启界面确认当前槽位是哪一个。这直接决定了你拿到的启动镜像**是否正确**。这样，你就不会为备份 boot_a 还是 boot_b 所困扰了（也就是上述指令上打 * 的地方）。

## 制作 Magisk 核心模式的启动镜像
得到了 Boot img，自然是成功了大半。喜悦之余，我们还需要考虑一个问题：Magisk patch 咋整？

毕竟平时，我们的 patch 工作是交给 Magisk manager 来完成的。不过手机开不了机，我们就得自己想办法在电脑上解决。还好，patch 所需的一切都在 Magisk 这个小小的安装 zip 包里有。

不过原版 zip 打包出来的还是会卡开机。毕竟有问题的模块在默认情况下还是会启动。这时，我们就需要求助于改装过的，打包出来的镜像是 Magisk 核心模式启动的 zip 包。[到这里下载它](https://github.com/ajayyy/magisk-recovery/releases/download/20.299999/magisk-debug.zip)，解包备用。

> ## GitHub 连接困难？
> 你也可以试试我镜像的 Magisk 核心模式修改版刷机包：[点我](https://cloud.pokon548.ink/s/TyikrfjK7bsaPxZ)。速度保证满意 2333

## 分析安装包结构
在一通查找后，有一个位于 ```common``` 文件夹下的文件引起了我的注意：

[![Uwjc34.png](https://s1.ax1x.com/2020/07/15/Uwjc34.png)](https://imgchr.com/i/Uwjc34)

直觉告诉我，这就是 Magisk Manager 给 boot 打 patch 的核心程序。点开一看，果然如此。

[![UwvP2Q.png](https://s1.ax1x.com/2020/07/15/UwvP2Q.png)](https://imgchr.com/i/UwvP2Q)

而且，还在脚本里把使用方法介绍的十分清楚：```./boot_patch.sh 待补丁的启动镜像.img```。这就好办了。有能用的工具，就开搞吧。

> ## 请注意修改 #! 解释器的路径
> 默认的脚本是为 Android 所设计，故默认的解释器路径为 ```#!/system/bin/sh```。但在电脑上这路径是错误的。请自行修改为 ```#!/bin/sh``` 或其它合理替代。

## Patch！
由于脚本里面把可执行文件路径写的很死，故不能通过指定可执行文件路径的方式相对执行 boot_patch.sh。推荐的方法是：把复制出来的镜像，以及 x86 和 common 目录里所有的可执行文件放在一个文件夹里。

就像这样：

![Uwx9dx.png](https://s1.ax1x.com/2020/07/15/Uwx9dx.png)

然后，执行 ```./boot_patch.sh boot.img```。开搞！

打好补丁的镜像叫做 ```new-boot.img```。就用这个镜像，Fastboot 刷进去。重启时 Magisk 就不会加载任何模块，顺利开机。

## Fastboot YES!
将你的手机置入 Fastboot 模式并连上电脑，输入以下指令：```fastboot flash boot new-boot.img```。

刷好重启。快开机，把有问题的模块干掉吧！至于你问要恢复正常模式怎么办... 原始镜像交给正常的 Magisk Manager 打一次补丁，刷进手机里不就好了（

好了，正片结束。以下为无伤大雅的小唠嗑。

## 一些尝试过，但似乎无效的办法
因为变砖了，且正巧我也是从 MIUI V2.3 刷过来的老（~~一开口就是老某某了~~）发烧友，所以遇到问题总是会试着自己解决。

结果在得出正文的解决办法前，愣是踩了一堆坑，欲哭无泪。果然是经验赶不上变化了么（（

### TWRP 残废
这也是本文一开始提及的大坑。这不比[手机强制加密](https://source.android.google.cn/security/overview/kernel-security#filesystem-encryption)，好歹这玩意只针对 Data。这下倒好，动态分区搞得 TWRP 没啥分区能看，更别提刷机了。

然后，```adb sideload``` 也是废的。而且，进入这个模式之后会导致我的 TWRP 逻辑卡死（即类比你程序里写了个无限循环，终止不了的情况）。要不是后面找了一圈，知道一加强制关机的逻辑是**要同时按下音量 - 和电源键十秒以上**，我可能得因此直接送修 7T Pro 了🤣

### 官方 Recovery + ADB Enabled = 同样残废
原本，我认为这种方法是我的救星。毕竟一个能解读动态分区的 Recovery + 完整的 ADB 权限，足以让我把阻止开机的模块删掉。

![救星啊（结果不是）](https://s1.ax1x.com/2020/07/15/UwT7a4.png)

结果，这个 Recovery 解密不了 data 分区...

不知道我动了什么，但是无论我如何折腾（取消解锁、确认 Recovery 完整性...），adb 就是拿不到 data 分区的数据。故只好作罢。

## 解包官方固件... payload.bin 是什么鬼
> 其实这个官方固件里的文件是可以作为可执行文件运行的（有点类似于 Linux 上的 [AppImage](https://appimage.org)）。+x 之后执行一段时间，自带的解包代码会最终把所有分区的镜像放出来，其中就包含 boot。奈何当初我不知道这一点，就有了本文正片里蹩脚的救砖方法。

本以为，我有官方固件包，还愁解决不了开机问题？结果打开一看... 只有一个大大的 payload.bin。

啊？我熟悉的 boot.img 去哪了？啊不过没有关系，只要有解包工具我还是一条好汉不是。结果一搜，貌似没有 Linux 上解包 payload.bin 的工具...