+++
title = "移除知乎的登录弹窗"
date = "2022-09-06T21:25:43+08:00"
author = ""
authorTwitter = "" #do not include @
cover = "https://s1.ax1x.com/2022/09/06/vHVmC9.jpg"
tags = ["zhihu", "adblock"]
keywords = ["zhihu", "adblock"]
description = "没人在意你家的登录权益，我只想安安静静的找点资料"
showFullContent = false
readingTime = false
hideComments = false
+++

## TL;DR
将以下过滤规则加入到你的广告过滤器里：
```html
! 屏蔽滚动页面时右下角弹出的刘看山登录提示框
zhihu.com##div[style^="margin-top: 16px; position: fixed; opacity: 1; transform: translateY"]

! 下面两个规则会一并把登录入口干掉。介意请勿添加。每次点一个页面都弹出登录 model 真是太烦人了...
zhihu.com##div[class="ColumnPageHeader-profile"]
zhihu.com##div:last-child > div > div[style^="transform-origin: center bottom; margin-top: -8px; opacity: 1; transform: none;"]
```

## 探究过程
知乎最新的模态提示框不再是以前单独的 ``signflow.js`` 请求，可以简单的屏蔽对应 JS 的请求解决。通过反查刘看山模态提示框的图片资源请求，发现这部分登录逻辑与主网站的 js 绑定在一起，简单的屏蔽资源会导致网站无法正常使用。

而且，该模态框使用动态 css 名称，简单的按类型屏蔽也起不到太好的作用。

## 但是...
谁说广告屏蔽器只能按 CSS 类名屏蔽的？

虽然你的 CSS 类名是随机的，但是你的 ``style`` 是固定不变的。那我直接按照你类的 ``style`` 特征屏蔽不就好了吗？

花费五分钟，写好了弹窗屏蔽规则，实验成功。

## 不足
- 由于这两个模态框是 JS 动态加入的（而非镶嵌在原本的 HTML 里），因此弹窗会在非常短暂的时间里弹出一次（闪动），随后便被移除；
- 登录窗口变得彻底不可用（被删除）。后面可能会想点别的办法嫁接一下。