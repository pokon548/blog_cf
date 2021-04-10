---
title: "Django 新的摩登后台 Jet Admin "
date: 2019-08-10T21:33:50+08:00
---
众所周知<del>才怪</del>，Django 自带的后台并不是那么好用。而且界面于功能实在太过简陋，不是很能迎合后台管理需求。
在不跳槽其它框架的情况下，怎么办呢？pokon548 最近就找到一个特别好用的在线后台管理程序 —— Jet Admin。它能为 Django 提供后台管理服务。
它不止能为在线项目提供服务，就连我们在本地上运行的测试项目（[http://127.0.0.1](http://127.0.0.1)　的这种）也 OK！是不是很神奇（对于 pokon548 这种一点都不精通 Web App 的人来说，当然是拍手称好啦）。

## [](#特点 "特点")特点

pokon548 光说好用当然不够，那就边上图边介绍好了。

### [](#高度可自定义的-Dashboard "高度可自定义的 Dashboard")高度可自定义的 Dashboard

哪些数据很关键要时刻关注？哪些数据要转换成图表？Jet Admin 都能帮你做。
![深度截图20190810213607.png](https://i.loli.net/2019/08/10/dl83kveHSaPfnZV.png)

比如 pokon548 自己正在做一个问卷调查的练手项目（Django 官方的新手教程）。通过选用 List，可以快速了解我有哪些 Questions，和哪些 Questions 有哪些 Votes…
![深度20190810215449.png](https://i.loli.net/2019/08/10/59lr8dycJfEw7t2.png)

当然，Django 后台的常规操作（添加、修改、删除数据）也是一个都不落下。

### [](#漂亮 "漂亮")漂亮

至于怎么个漂亮法，看看官方的介绍图就知道了。
![Preview](https://raw.githubusercontent.com/jet-admin/jet-bridge/master/static/overview.gif)

### [](#测试项目也能用 "测试项目也能用")测试项目也能用

是的，你没有看错。用 runserver 跑起来的 [http://127.0.0.1](http://127.0.0.1) 测试服务器也能用！对于开发者实在是太友好了，不是么。

## [](#隐私 "隐私")隐私

Jet Admin 声称自己的服务器是不能主动碰不到部署端的数据的，必须借助在项目里面加入的 Jet Bridge 才能对数据进行操作。而且 Jet Bridge 在 GitHub 上是开源的。
他们自己的隐私声明如下：

> Jet Admin is built in a way that it hosts only your admin panel interface. We don’t collect or host your private data. Thanks to Jet Admin’s architecture, we are able to display and support your admin without actually accessing your information.
> 
> <footer>**Data privacy & Security - Jet Admin**<cite>[docs.jetadmin.io/user-guide/data-privacy-and-security](https://docs.jetadmin.io/user-guide/data-privacy-and-security)</cite></footer>

所以，数据隐私问题应该不用太过于担心（况且我就是学习 Django，把数据拿走了也没什么意义）。

## [](#把-Jet-Admin-部署到-Django-中 "把 Jet Admin 部署到 Django 中")把 Jet Admin 部署到 Django 中

至于怎么部署，官方已经给了非常详细的步骤：[点我](https://docs.jetadmin.io/getting-started-1/install)，恕不赘述。
