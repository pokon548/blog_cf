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
zhihu.com#%#originalEvent = document.addEventListener
zhihu.com#%#document.addEventListener = (event,listener,useCapture)=>{event!="DOMContentLoaded"?originalEvent(event,listener,useCapture):listener.toString().length<1000?originalEvent(event,listener,useCapture):originalEvent(event,function e(){originalScroll=window.addEventListener;window.addEventListener=(event,listener,useCapture)=>{event!="scroll"?originalScroll(event,listener,useCapture):listener.toString().length==177?null:originalScroll(event,listener,useCapture)};listener()},useCapture)}
```

## 探究过程
知乎最新的模态提示框不再是以前单独的 ``signflow.js`` 请求，可以简单的屏蔽对应 JS 的请求解决。通过反查刘看山模态提示框的图片资源请求，发现这部分登录逻辑与主网站的 js 绑定在一起，简单的屏蔽资源会导致网站无法正常使用。

而且，该模态框使用动态 css 名称，简单的按类型屏蔽也起不到实质作用。

## 但是...
谁说广告屏蔽器只能按 CSS 类名屏蔽的？

~~虽然你的 CSS 类名是随机的，但是你的 ``style`` 是固定不变的。那我直接按照你类的 ``style`` 特征屏蔽不就好了吗？~~

~~花费五分钟，写好了弹窗屏蔽规则，实验成功。~~

为了实现完美的效果，逆向了一下知乎的前端 JS，并针对性写了一段脚本来处理这个麻烦的弹出框[^1]。简单来说，我通过：
1. 劫持浏览器的 ``window.addEventListener`` 函数，并替换为自定义的版本；
2. 在自定义的处理函数中，判断 ``listener`` 中的函数是否用于加载登录弹窗。如果是，则拒绝将其添加到浏览器事件中。

来从源代码层面上解决弹窗问题[^2]。
## 不足
- 右上角的登录模态框依然会弹出。但是，如果你在页面完全加载完毕前向下滚动，它就不会弹出。可以近似忽略它的存在。

[^1]: AdGuard 浏览器插件允许使用 JavaScript 作为高级过滤规则。不清楚 uBlock Origin 等是否可以使用。请自行测试
[^2]: 直接过滤元素会带来闪动问题。知乎目前的登录模态框并非直接镶嵌在 HTML 内，而是由 JavaScript 动态生成 DOM 组件。因此，简单的元素过滤会使得登录模态框依然有短暂弹出的机会，带来糟糕的浏览体验