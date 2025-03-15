---
title: ArchlinuxCN 源 clash-verge-rev 2.1.2-2 无法启动服务
comments: true
tags:
  - clash
  - archlinux
categories: 折腾
abbrlink: b859c90d
date: 2025-03-13 11:33:25
---

昨天晚上睡觉前, pacman -Syu 更新了一下. 结果今天早上一看, 我的clash-verge不起作用了!

查看日志发现是clash-verge-service无法连接, 手动启用获得报错信息"33211端口被占用".

但是通过 'ss -utln | grep 33211' 发现, 占用33211端口的就是我手动启动的这个clash-verge-service呀?

对比了一下, 我发现了CN源的clash-verge-rev比AUR的要高了一小个版本, 也就是'pkgrel'高了一个版本, 但是clash-verge本身没有更新, 查看了commit信息, 发现是clash-verge的上游有一个更新导致的.

所以我猜测, 就是这个的原因, 于是换到了AUR的clash-verge-rev 2.1.2-1, 最终问题解决.