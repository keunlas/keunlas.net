---
title: 在Archlinux中启动spectacle报错lapack的解决方法
comments: true
description: <center>今天运行使用spectacle这款截图工具的时候突然报错了，经过排查是lapack库的锅。解决过程记录在此。</center>
tags:
  - Archlinux
  - spectacle
  - lapack
  - blas-openblas
categories: 折腾
abbrlink: 97a1dd2e
date: 2025-01-11 16:36:00
---

# 发现问题

当我像往常一样按下快捷键进行截图的时候，突然KDE报了一个错误通知。

```error
Could not activate remote peer 'org.kde.spectacle': startup job failed
```

从这个通知，看不出来到底问题出在哪，只知道spectacle无法启动。于是我决定用命令行运行一遍spectacle.

命令行运行后错误消息给的很精准，问题立马暴露出来了。

```error
spectacle: symbol lookup error: /usr/lib/liblapack.so.3: undefined symbol: taa_
```

很明显问题在于lapack这个看名字像是和线性代数有关的库。

# 解决问题

竟然知道了问题出在lapack这个库上，那么就要想办法解决这个库了。

## 解法一：降级lapack

在archlinux论坛上，我看到有人说，这由lapack 3.12.1这个版本而导致的问题，只要降级到3.12.0就没有问题了。

于是我着手安装downgrade这个工具，准备降级lapack。

```shell
sudo pacman -S downgrade
```

安装好downgrade之后，使用下面这个命令进行降级。

```shell
sudo downgrade lapack
```

在downgrade界面中，选择lapack 3.12.0-5版本，进行降级。

降级完成之后，再次运行spectacle已经没有任何问题了，可以正常使用了。

## 解法二：使用其他的线性代数库替代lapack

正所谓，降级只是缓兵之计。

降级之后，如果不排除这个lapack包在更新之外的话，只要lapack还没解决这个问题，每次更新都要手动降级lapack一遍，实在是麻烦。

如果把lapack排除在升级之外，之后就需要我们自己去注意lapack的版本升级了，万一忘记了，岂不是就要一直用旧版的吗😰，直到旧版再次出现问题，才能够想起来升到更新的版本。简言之，就是对于我这种喜欢用最新的包的archlinux用户来说，强迫症犯了。

所以我们可以用其他的库来代替出问题的库。

这里我在archlinuxcn交流群中，发现了blas-openblas这个库，这个库就不会让spectacle出现问题，所以直接安装。

```shell
sudo pacman -S blas-openblas
```

在安装的过程中，把冲突的包全部移除（lapack包含于其中），安装完成之后。spectacle就可以正常运行了。

