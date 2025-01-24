---
title: 文件映射到内存
comments: true
description: <center>将文件映射到内存, 通过指针像操作内存一样来操作文件.</center>
abbrlink: 45842bc5
date: 2025-01-23 14:44:26
tags:
  - memory
  - Linux
  - 文件映射
  - POSIX
categories: Linux系统编程
---


`mmap` 和 `munmap` 是用于内存管理的两个重要函数，它们在 Unix 和 Linux 系统编程中非常常见。下面是它们的详细使用信息：

# mmap 函数

`mmap` 函数用于将文件或其他设备映射到内存中，从而实现高效的文件 I/O 操作。它的函数原型如下：

```c
#include <sys/mman.h>

void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
```

## 参数说明

- `addr`: 指定映射的起始地址。通常设置为 `NULL`，让系统自动选择地址。
- `length`: 映射区域的长度，以字节为单位。
- `prot`: 映射区域的保护方式，可以是以下几种的组合：
  - `PROT_READ`: 可读
  - `PROT_WRITE`: 可写
  - `PROT_EXEC`: 可执行
  - `PROT_NONE`: 不可访问
- `flags`: 影响映射行为的标志，可以是以下几种的组合：
  - `MAP_SHARED`: 共享映射，文件会被多个进程共享。
  - `MAP_PRIVATE`: 私有映射，文件内容会被复制到匿名内存中。
  - `MAP_FIXED`: 如果指定了 `addr`，则必须映射到该地址，否则返回错误。
  - `MAP_ANONYMOUS` 或 `MAP_ANON`: 不与文件关联，只使用内存。
- `fd`: 文件描述符，如果使用 `MAP_ANONYMOUS`，则该参数被忽略。
- `offset`: 文件映射的偏移量，必须是页大小的倍数。

## 返回值

- 成功时，返回映射区域的起始地址。
- 失败时，返回 `MAP_FAILED`，并设置 `errno`。

## 示例代码

```c
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    int fd = open("example.txt", O_RDONLY);
    if (fd == -1) {
        perror("open");
        return 1;
    }

    struct stat sb;
    if (fstat(fd, &sb) == -1) {
        perror("fstat");
        close(fd);
        return 1;
    }

    char *map = mmap(NULL, sb.st_size, PROT_READ, MAP_SHARED, fd, 0);
    if (map == MAP_FAILED) {
        perror("mmap");
        close(fd);
        return 1;
    }

    // 使用映射区域
    printf("File content: %s\n", map);

    // 解除映射
    if (munmap(map, sb.st_size) == -1) {
        perror("munmap");
        close(fd);
        return 1;
    }

    close(fd);
    return 0;
}
```

# munmap 函数

`munmap` 函数用于解除由 `mmap` 创建的内存映射。它的函数原型如下：

```c
#include <sys/mman.h>

int munmap(void *addr, size_t length);
```

## 参数说明

- `addr`: 要解除映射的区域的起始地址，必须是之前 `mmap` 返回的地址。
- `length`: 要解除映射的区域的长度，必须与 `mmap` 调用时的长度相同。

## 返回值

- 成功时，返回 `0`。
- 失败时，返回 `-1`，并设置 `errno`。

## 示例代码

上面的 `mmap` 示例代码中已经包含了 `munmap` 的使用示例。

# 总结

- `mmap` 用于将文件或其他设备映射到内存中，实现高效的文件 I/O 操作。
- `munmap` 用于解除由 `mmap` 创建的内存映射。
- 使用 `mmap` 和 `munmap` 时需要注意参数的正确设置和错误处理。

