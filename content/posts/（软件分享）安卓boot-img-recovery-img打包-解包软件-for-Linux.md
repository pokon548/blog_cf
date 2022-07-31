---
title: "（软件分享）安卓boot Img Recovery Img打包 解包软件 for Linux"
description: 只适用于 Linux 桌面环境...
date: 2016-02-20T21:10:01+08:00
---
只适用于Linux桌面环境，不适用于Windows（可以Windows安装cygwin执行这个软件）

链接在底部（这里以最新的twrp3.0汉化作示例，末尾有成品）
使用方法：
解包boot/recovery文件
1、给所有可执行文件加上执行权限（否则会打包错误或漏打包）
![图](https://attach.bbs.miui.com/forum/201602/20/112536be74x8d4xf7rcg6f.png.thumb.jpg)

2、执行命令：（请保证终端所执行的文件夹在这个程序的文件夹内）

<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">sudo sh mkboot boot.img output</span>
</pre></td></tr></table></figure>

![图](https://attach.bbs.miui.com/forum/201602/20/112956hznn5mf5qqn7ga9e.png.thumb.jpg)
（为什么要加上sh呢？因为部分终端不加sh会去内部寻找此命令，而一般情况下是没有的，会报错）
boot.img请根据自己的实际路径修改，建议与可执行文件同一个目录里。

3、随意修改
![图](https://attach.bbs.miui.com/forum/201602/20/113109chxhqy1qxp0h0qkh.png.thumb.jpg)

打包boot/recovery：
1、执行下面命令即可：

<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">sudo sh mkboot output new_boot.img</span>
</pre></td></tr></table></figure>

（同样，new_boot.img也要根据实际的路径来）
新的boot/recovery就出来了哦：
![图](https://attach.bbs.miui.com/forum/201602/20/113709vvgwzlvclbu5ac19.png.thumb.jpg)
下载地址：（为保证最新，贴上github链接）[https://github.com/xiaolu/mkbootimg_tools](https://github.com/xiaolu/mkbootimg_tools)

可能有些人看不懂，这很正常，因为这是为大触准备的:)
末尾资源：自修改版twrp3.0 for红米1s3g联通电信（补全简体中文和繁体中文，其它未动，未作精简和改变，语言和字体均为官方）：
(另外说一下，twrp3.0本来是自带中文的，但是不知什么原因变成了编译者可选的扩展语言，所以，如果你想亲自汉化的话，可以去twrp的github下载相应的文件替换）

![图](https://attach.bbs.miui.com/forum/201602/20/115739qsoo55o5phk9bmu5.png.thumb.jpg)
![图](https://attach.bbs.miui.com/forum/201602/20/115740z5xzd3bx50cg0sem.png.thumb.jpg)
[http://pan.baidu.com/s/1hrjqkuC](http://pan.baidu.com/s/1hrjqkuC)
