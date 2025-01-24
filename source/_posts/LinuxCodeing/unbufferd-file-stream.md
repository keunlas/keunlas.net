---
title: 无缓冲文件流
comments: true
description: '<center>Linux系统编程, 无缓冲文件流, 跳过用户缓冲区, 直接利用内核缓冲区进行文件读写.</center>'
tags:
  - C语言
  - Linux
  - 无缓冲文件流
  - POSIX
categories: Linux系统编程
abbrlink: 8eafffb1
date: 2025-01-23 14:05:54
---


无缓冲文件流是UNIX和Linux系统中用于进行文件读写操作的一种方式。它提供了底层的文件操作接口，允许直接对文件进行读写操作，而不需要经过标准库的缓冲区。


无缓冲文件流操作需要包含以下头文件：

```c
#include <fcntl.h>  // 包含open函数
#include <unistd.h> // 包含read, write, close, ftruncate函数
#include <sys/types.h> // 包含off_t类型
#include <sys/stat.h> // 包含mode_t类型
```

具体来说：
- `fcntl.h`：定义了`open`函数及其相关标志（如`O_RDONLY`, `O_WRONLY`, `O_RDWR`, `O_CREAT`, `O_TRUNC`等）。
- `unistd.h`：定义了`read`, `write`, `close`, `ftruncate`等函数。
- `sys/types.h`：定义了`off_t`类型，用于表示文件偏移量。
- `sys/stat.h`：定义了`mode_t`类型，用于表示文件权限。

这些头文件提供了进行无缓冲文件操作所需的基本函数和数据类型。

以下是对无缓冲文件流中一些关键函数的总结：

# open
`open`函数用于打开一个文件，并返回一个文件描述符，该描述符用于后续的文件操作。

**函数原型**:
```c
int open(const char *pathname, int flags, mode_t mode);
```

**参数**:
- `pathname`: 文件的路径名。
- `flags`: 文件访问模式（如`O_RDONLY`、`O_WRONLY`、`O_RDWR`）和其他操作标志（如`O_CREAT`、`O_TRUNC`）。
- `mode`: 当使用`O_CREAT`标志时，指定新文件的权限。

**返回值**:
- 成功时返回文件描述符。
- 失败时返回-1，并设置`errno`。

# close
`close`函数用于关闭一个文件描述符。

**函数原型**:
```c
int close(int fd);
```

**参数**:
- `fd`: 要关闭的文件描述符。

**返回值**:
- 成功时返回0。
- 失败时返回-1，并设置`errno`。

# read
`read`函数用于从文件中读取数据。

**函数原型**:
```c
ssize_t read(int fd, void *buf, size_t count);
```

**参数**:
- `fd`: 文件描述符。
- `buf`: 存储读取数据的缓冲区。
- `count`: 要读取的字节数。

**返回值**:
- 成功时返回读取的字节数。
- 失败时返回-1，并设置`errno`。
- 当到达文件末尾时返回0。

# write
`write`函数用于向文件中写入数据。

**函数原型**:
```c
ssize_t write(int fd, const void *buf, size_t count);
```

**参数**:
- `fd`: 文件描述符。
- `buf`: 要写入的数据缓冲区。
- `count`: 要写入的字节数。

**返回值**:
- 成功时返回写入的字节数。
- 失败时返回-1，并设置`errno`。

# ftruncate
`ftruncate`函数用于调整文件的大小。

**函数原型**:
```c
int ftruncate(int fd, off_t length);
```

**参数**:
- `fd`: 文件描述符。
- `length`: 新的文件长度。

**返回值**:
- 成功时返回0。
- 失败时返回-1，并设置`errno`。

# 示例代码
以下是一个使用无缓冲文件流进行文件复制的示例代码：

```c
#define _CRT_SECURE_NO_WARNINGS
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    int src_fd = open("source.txt", O_RDONLY);
    if (src_fd == -1) {
        perror("open source.txt");
        return 1;
    }

    int dest_fd = open("destination.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (dest_fd == -1) {
        perror("open destination.txt");
        close(src_fd);
        return 1;
    }

    char buf[4096];
    ssize_t bytesRead;
    while ((bytesRead = read(src_fd, buf, sizeof(buf))) > 0) {
        ssize_t bytesWritten = write(dest_fd, buf, bytesRead);
        if (bytesWritten != bytesRead) {
            perror("write");
            close(src_fd);
            close(dest_fd);
            return 1;
        }
    }

    if (bytesRead == -1) {
        perror("read");
    }

    close(src_fd);
    close(dest_fd);
    return 0;
}
```

这个示例代码展示了如何使用`open`、`read`、`write`和`close`函数进行文件复制操作。


# lseek

`lseek` 函数是 Unix 和 Linux 系统中的一个系统调用，用于在文件描述符上移动文件偏移量。它允许程序在文件中定位到特定的位置，以便进行读写操作。

函数原型如下：
```c
#include <unistd.h>
#include <sys/types.h>

off_t lseek(int fd, off_t offset, int whence);
```

**参数说明**

- `fd`: 文件描述符，通常是通过 `open` 函数获得的文件描述符。
- `offset`: 偏移量，表示相对于 `whence` 指定的位置移动的字节数。
- `whence`: 基准位置，可以是以下值之一：
  - `SEEK_SET`: 文件开头
  - `SEEK_CUR`: 当前文件偏移量
  - `SEEK_END`: 文件末尾

**返回值**

- 成功时返回新的文件偏移量（从文件开头的字节数）。
- 失败时返回 `-1`，并设置 `errno` 以指示错误类型。
