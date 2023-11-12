---
title: 修复「Your device is corrupt」
author: 初時雨
email: Hatsushigure_c@163.com
readmore: true
hideTime: false
date: 2023-10-27 14:29:53
tags: [小米, 刷机]
description: 胆大心细得成功
categories: 搞机
---
## 问题出现

书接[上文](/2023/07/22/i-must-be-more-careful/)(你这上文有点久远了啊)，经过我的~~不懈努力~~，我的手机又一次出现了这个问题
![Your device is corrupt](https://i2.100024.xyz/2023/07/22/li45wi.webp)
_(素材复用.jpg)_
但是如今的我，早已不是当初那个从没刷过机的小白了，经过多方求证，我发现这个问题和手机的 AVB 验证有关

## 相关资料
> 启动时验证会尽力确保所有已执行代码均来自可信来源（通常是设备的原始设备制造商 [OEM]），以防受到攻击或损坏。它可建立一条从受硬件保护的信任根到引导加载程序，再到 boot 分区和其他已验证分区（包括 system、vendor 和可选的 oem 分区）的完整信任链。在设备启动过程中，无论是在哪个阶段，都会在进入下一个阶段之前先验证下一个阶段的完整性和真实性。
> 除了确保设备运行的是安全的 Android 版本外，启动时验证还会检查 Android 版本是否为具有回滚保护的正确版本。回滚保护可确保设备只会更新到更高的 Android 版本，从而帮助避免可能的漏洞持续存在。

(摘自 [android.com](https://source.android.com/docs/security/features/verifiedboot?hl=zh-cn))

## 轻松秒杀

了解了这些，我们就可以着手解决问题了。在出现这个问题之前，我只使用 `DSU Sideloader` 加载了多次 [`GSI`](https://developer.android.com/topic/generic-system-image?hl=zh-cn), 尝试体验类原生，虽然最后都无法进入系统界面，只能无线软重启。而我们知道, `GSI` 一般是不会替换上述的 `boot`, `oem` 分区的，那么真相就只有一个了，那就是 `system` 分区损坏了。
备份数据，进入 `fastbootd` 模式之后，瞄准 `system` 直接开刷。重启之后，很明显，问题已经被轻松秒杀，那个烦人的提示已经完全消失了。而且更妙的是，我的数据完全没有丢失。从曾经的小白到如今能够自己解决问题，我想这也是一种成长吧。

## 更新预告
[刷机到底刷哪里--Android 分区详解](about:blank) ~~有生之年系列~~