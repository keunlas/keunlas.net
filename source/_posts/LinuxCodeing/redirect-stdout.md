---
title: 重定向标准流
comments: true
description: <center>主要是dup函数.</center>
tags:
  - C语言
  - Linux
  - 文件流
categories: Linux系统编程
abbrlink: 27162a1a
date: 2025-01-24 11:35:28
---


# 标准流

在 Unix 和 Linux 系统中，标准流（Standard Streams）是进程与外部环境之间的三个基本接口。它们分别是标准输入（stdin）、标准输出（stdout）和标准错误（stderr）。每个进程默认都有这三个标准流，它们都是文件描述符，可以通过文件描述符进行操作。

- 标准输入流stdin：宏STDIN_FILENO，对应整数值0
- 标准输出流stdout：宏STDOUT_FILENO，对应整数值1
- 标准错误流stderr：宏STDERR_FILENO，对应整数值2

# 重定向标准流

在 Unix 和 Linux 系统中，可以使用 `dup` 和 `dup2` 函数来复制文件描述符，从而实现标准流的重定向。这些函数可以将一个文件描述符复制到另一个文件描述符，使得两个文件描述符指向同一个文件表项。这样，对其中一个文件描述符的操作就会影响到另一个文件描述符指向的文件。

> 当标准流被关闭时, 就可以让其他的流占用标准流的文件标识符, 从而实现重定向.

# dup 函数

`dup` 和 `dup2` 是 Unix 和 Linux 系统中用于文件描述符操作的函数。它们用于复制文件描述符，使得多个描述符可以引用同一个打开的文件。下面是它们的详细信息：

## 原型
```c
#include <unistd.h>

int dup(int oldfd);
```

## 功能
`dup` 函数用于复制文件描述符 `oldfd`，返回一个新的文件描述符，该描述符与 `oldfd` 共享同一个文件表项。新描述符保证是最小的未使用的描述符。

## 参数
- `oldfd`：要复制的文件描述符。

## 返回值
- 成功时，返回新的文件描述符。
- 失败时，返回 -1，并设置 `errno` 来指示错误。

# dup2 函数

## 原型
```c
#include <unistd.h>

int dup2(int oldfd, int newfd);
```

## 功能
`dup2` 函数用于将文件描述符 `oldfd` 复制到 `newfd`。如果 `newfd` 已经打开，则首先关闭它。`dup2` 保证返回的文件描述符是 `newfd`。

## 参数
- `oldfd`：要复制的文件描述符。
- `newfd`：目标文件描述符。

## 返回值
- 成功时，返回 `newfd`。
- 失败时，返回 -1，并设置 `errno` 来指示错误。

# 注意事项
1. `dup` 和 `dup2` 都会复制文件描述符的打开模式和文件偏移量。
2. 使用 `dup2` 时，如果 `oldfd` 和 `newfd` 相同，则函数不执行任何操作，直接返回 `newfd`。
3. `dup2` 在复制前会自动关闭 `newfd`，即使它当前正在使用。

通过这些函数，可以方便地管理文件描述符，实现文件描述符的复制和重定向。