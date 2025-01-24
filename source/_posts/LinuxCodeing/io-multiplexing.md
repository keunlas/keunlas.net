---
title: I/O多路复用
comments: true
description: <center>主要内容有管道文件, 以及select结构体.</center>
tags:
  - Linux
  - I/O
  - 多路复用
  - 管道
  - select
  - POSIX
categories: Linux系统编程
abbrlink: 7ce262fd
date: 2025-01-24 11:51:14
---

# 创建管道文件

管道文件是一种特殊的文件, 它允许一个进程向另一个进程发送数据. 管道文件有两种类型: 命名管道和匿名管道.

## 匿名管道

匿名管道是一种只能在具有亲缘关系的进程之间使用的管道. 它通常用于父子进程之间的通信. 创建匿名管道的函数是 `pipe()`, 它的函数原型如下:

```c
#include <unistd.h>

int pipe(int pipefd[2]);
```

`pipefd` 是一个数组, 用于存储管道文件的文件描述符. `pipefd[0]` 是管道的读端, `pipefd[1]` 是管道的写端. 如果 `pipe()` 函数调用成功, 它将返回 0, 否则返回 -1 并设置 `errno` 以指示错误原因.

匿名管道的读写规则如下:
- 写端关闭: 如果写端关闭, 则读端将返回 0, 表示管道已关闭.
- 读端关闭: 如果读端关闭, 则写端将收到 SIGPIPE 信号, 表示管道已关闭.
- 管道已满: 如果管道已满, 则写操作将阻塞, 直到有足够的空间可用.

## 命名管道

命名管道是一种可以在没有亲缘关系的进程之间使用的管道. 它通常用于不同进程之间的通信. 创建命名管道的函数是 `mkfifo()`, 它的函数原型如下:

```c
#include <sys/types.h>
#include <sys/stat.h>

int mkfifo(const char *pathname, mode_t mode);
```

`pathname` 是命名管道的路径名, `mode` 是命名管道的权限. 如果 `mkfifo()` 函数调用成功, 它将返回 0, 否则返回 -1 并设置 `errno` 以指示错误原因.

命名管道的读写规则与匿名管道相同.

也可以在Linux命令行中使用 `mkfifo` 命令创建命名管道.

```sh
mkfifo named.pipe
```

这个命令将会创建名为`named.pipe`的管道, 管道文件不允许用`vim`这样的编译器打开.

# 使用管道

通过`echo hello > 1.pipe`向管道缓冲区写入数据, 我们会发现这个写入操作会直接写阻塞; 这是因为使用命名管道 , 是从一端读取数据, 从另一端写入数据, 必须读端写端同时打开, 无法单独去打开读端或者写端。

当我们另开一个窗口(新的进程)使用`cat 1.pipe`读取管道数据, 相当于开启了管道的读端,  这可以使之前打开的写端正常写入,  从阻塞态变为非阻塞态写入;  等写端写入完成之后, 读端读出写入数据。


**像文件一样使用管道**

管道文件可以像普通文件一样使用, 可以使用 `read()` 和 `write()` 函数来读写管道文件. 例如:

```c
#include <all_in_one.h>

int main(int argc, char const *argv[]) {
  ARGS_CHECK(argc, 2);
  // 只读打开管道
  int pipe_fd = open(argv[1], O_RDONLY);

  // 记录读取字节大小
  ssize_t read_size;
  // 记录读取到的字符串
  char buf[BUFSIZ] = {0};
  // 没有出错就循环读取
  while ((read_size = read(pipe_fd, buf, BUFSIZ - 1)) != -1) {
    // 确保读取到的内容长度不为0
    if (read_size != 0) {
      buf[read_size] = '\0';
      printf("received [%s] from %s.\n", buf, argv[1]);
    }
    // 休息一会儿, 太快看不清.
    sleep(1);
  }

  printf("read done.");
  return 0;
}
```










