---
title: Windows 预览体验过期无法更新的解决方案
date: 2022-09-18 13:00:28
tags: Windows
categories: 杂谈
description: "奇怪的经验增加了"
---
## 起因
由于傻逼学校太久不放假，一直没有时间碰电脑(差不多50天吧)。开机后，直接触发时间炸弹(忘了截图了，~~随便放一张凑合下吧~~)。
![评估副本已过期](https://i2.100024.xyz/2022/09/18/u3nz9u.webp)
作为一名快圈勇士，我怎么会被区区时间炸弹吓到呢，于是我果断选择更新，这才有了更多奇怪的发现。
## 影响
#### 开幕雷击: 积累更新无法安装
![](https://i2.100024.xyz/2022/09/18/u4f720.webp)
这种情况肯定是求助万能的~~百度~~微软支持啦，但是很不幸，没有一种方法可以解决我的问题。
#### 梅开二度: 功能更新异常
就一般理性而言，只要替换[`AppraiserRes.dll`](https://cn.bing.com/search?q=AppraiserRes.dll "AppraiserRes.dll")，功能更新就不会出问题。然而，经过漫长的等待后...
![](https://i2.100024.xyz/2022/09/18/u51jyt.webp)
?!~~你TM故意找茬是吧~~
我用了超过5年 Windows 10 及以上版本，这种情况还是第一次见到。什么时候功能更新还没重启就能完成了？很显然，网上甚至没有人出现过这种问题，更不用谈解决方案了。
#### 三羊开泰: 奇怪的权限索取
众所周知，打开任务管理器是不需要授予管理员权限的，但是...
![](https://i2.100024.xyz/2022/09/18/u5gmg3.webp)
???
还有设置
![](https://i2.100024.xyz/2022/09/18/u5mpma.webp)
## 真相揭晓
问题越积越多，但是真相也越来越清晰了。结合之前看过的某个视频([BV1xg411k7Qw](https://bilibili.com/BV1xg411k7Qw "BV1xg411k7Qw"))，不难发现，这就是证书过期导致的！
顺着这条线索，我查看了 Windows 的证书
![](https://i2.100024.xyz/2022/09/18/u62vvs.webp)
不出所料，证书在前一天过期了(大冤种就是我了)。既然知道原因了，那么接下来解决就很简单了吧(确  信)。
## 问题解决
很好，经过亿个小时的奋斗，我放弃了。但就在这时，我突然脑洞大开。既然签名是16日过期的，那我把日期改到15日会怎么样呢？没想到，这个方法奏效了。
![](https://i2.100024.xyz/2022/09/18/u68lce.webp)
成！功！了！
## 总结
能够遇到这种奇葩情况的大冤种估计全网找不出第二个来，这篇文章就当是一个预警吧。看来既然选择了快圈，还是得按时更新啊！不说了，更新去了。