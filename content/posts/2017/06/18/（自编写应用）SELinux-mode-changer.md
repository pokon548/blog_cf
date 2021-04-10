---
title: "（自编写应用）SELinux Mode Changer"
date: 2017-06-08T21:27:50+08:00
---
这是我第一次涉足Android开发领域的作品，由于开发经验不足，可能会存在诸多问题，我会在后期更新。望诸位理解

介绍：这是一个可以自动切换SELinux模式的程序，它会在每一次开机自启动并自动设置SELinux为permissive模式。（需要没有优化软件阻止获取开机完成广播）
特殊要求：该软件需要root
大小：（仅APK）不足60K
用法：第一次开启是唯一需要你手动设置SELinux模式的时候，（打开获取root权限后即设置好）之后你就可以把这个APP放到你喜欢的文件夹里了，下次开机程序便会自动更改模式到permissive
Tips：由于先后顺序关系，该程序自启动可能较迟。如果一直没有看见Toast提示，请检查你的优化软件是否放行开机自启动权限

![图](https://attach.bbs.miui.com/forum/201706/18/102531ydyd7smm46w9g6sy.png.thumb.jpg)

下载链接：[https://pan.baidu.com/s/1c2aQPPI](https://pan.baidu.com/s/1c2aQPPI)
