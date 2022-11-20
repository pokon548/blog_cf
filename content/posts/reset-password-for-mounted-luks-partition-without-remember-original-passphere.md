+++
title = "无需原密钥，重置已经挂载的 LUKS 加密分区密码"
date = "2022-11-06T11:07:09+08:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["luks", "encrypt", "recovery"]
keywords = ["luks", "encrypt", "recovery"]
description = "刚刚加密了一个分区，挂载完分区，拷贝完重要文件，却唯独忘了记下加密分区的密钥？首先，千万不要卸载已经挂上的加密分区，或是重启计算机！深吸一口气，并继续阅读这篇文章（恢复很容易！）"
readingTime = false
hideComments = false
+++

> 🚨 注意！该文章 **只适用** 于恢复这样的分区：
>
> 1. 该分区以 LUKS 方式加密；
> 2. 该分区 **依然** 在系统里处于挂载状态，**没有被卸载出系统**。换言之，该分区处于解锁状态，你还能（暂时）自由的访问上面的数据。
>
> **如果你已经失去了该分区的访问能力（分区已被卸载），且没有可靠的数据备份，那么本文章无力回天。请另寻高就或是就地重开，非常抱歉。**
>
> 但如果你的加密分区满足上面两个条件，请继续阅读！你可以（甚至是轻松的）重置加密分区密钥，而且无需输入你忘记的原密钥！

## TL;DR
以 root 身份执行下列指令：
```bash
cryptsetup luksAddKey /dev/sda1 --master-key-file <(dmsetup table --showkey /dev/mapper/luks-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | awk '{print$5}' | xxd -r -p)
```

其中，```sda1``` 替换为你实际加密的分区名，```luks-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx``` 替换为该分区对应的 luks mapper。

稍等片刻，程序便会弹出如下提示，让你输入需要增加的新密钥。

```bash
输入密钥槽的新口令: 
确认密码：
```

注意，此时 ```cryptsetup``` 不再要求你输入原密钥。在可靠的位置（如密码管理器）生成并记录一个新的密钥，并把它作为参数喂给程序吧！

喂完了记得回车确认！

> ❗等一等！先确认是否真的执行完成！
>
> 在你兴高采烈的执行完上面的指令后，别忘了测试你的新密钥是否有效！输入以下指令（别忘了把 ```sda1``` 替换为你实际的分区名！）：
> ```bash
> cryptsetup --test-passphrase -v open /dev/sda1
> ```
> 看看你的新密钥是否可以解锁分区。
>
> 如果你可以通过上面的指令，顺利得到类似于如下的结果：
> ```bash
> Key slot 1 unlocked.
> Command successful.
> ```
> 则证明你的新密钥可被用于解锁分区！松一口气，喝杯茶，重置完成了！:)
>
> 如果你不是很关心接下来的原理部分，便可以直接关掉这篇文章，继续做其它的事情了。

## 原理
或许你会好奇，以上操作的原理是什么？简而言之，以上的复合命令做四件事：
- 从已经挂载的 luks mapper 中提取密钥信息；
- 通过 ```awk``` 指令过滤出实际的密钥；
- 由于 ```cryptsetup``` 只接受二进制密钥，因此需要用 ```xxd``` 指令将提取出的密钥转换为 binary 格式；
- 将转换完成的密钥，作为参数喂给 ```cryptsetup luksAddKey```。由于已获得合法的密钥信息，该程序不再向用户询问密码，转而直接进入增加密钥的步骤，也就是变相跳过了输入原密码的环节。

## 背景
至于为什么会写这么一篇文章，起因其实是我今天想要迁移系统。我有一个备份 Linux 系统用的移动磁盘，为避免丢失硬盘后造成信息安全问题，便对所有的磁盘进行了 LUKS 加密。

在备份的时候，我用密码管理器生成了一个高强度密码，并直接用于该备份磁盘的加密。然而，我只是让 KDE 的 Dolphin 记住了我加密分区的密码，但密码管理器没有记住！

当时，我还没意识到这是个十分危险的举措。原因在于，备份的密钥仅*单点留存*于原系统里 [^2]，如果系统损毁无法进入，备份也同样无法解锁，相当于没有备份。

直到今天，我在已经将整个家目录迁移到备份磁盘，准备迁移系统的时候，突然想起来一件事：

> 我的备份是否可以正常访问？

毕竟，我是从一个系统迁移到另一个系统，原有的系统信息等价于全部丢失。于是，我便习惯性的打开密码管理器，查找先前记录的备份分区密钥。

然而！并没有找到！所有密码管理器里记录的密钥都无法解锁该备份分区！心情复杂 :(

于是，遂开始在网上寻找解决方案。在花费了约两个小时之后，顺利找到了如上的解决方案，问题解决。

可以愉快的迁移系统了！

## 特别感谢
[RedHat 的这篇文章](https://access.redhat.com/solutions/1543373)

[^2]: 我的密码管理器是自建实例，并严格每周执行 3-2-1 原则备份。可以认为存放在密码管理器里的信息不会灭失