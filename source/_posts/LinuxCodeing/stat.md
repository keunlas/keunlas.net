---
title: Linux系统调用函数stat
comments: true
description: <center>stat是一个系统调用函数，用于获取指定文件的相关信息。</center>
date: 2025-01-21 20:45:31
tags:
  - C语言
  - Linux
  - stat
categories: Linux系统编程
---


# stat系统调用函数

stat用于获取指定文件的相关信息, dirent虽然也可以获得文件的信息, 但是太少了.

```c
#include <sys/stat.h>
int stat(const char *path, struct stat *buf);
```

- 形参
  - path：这是一个指向字符数组的指针，代表要获取信息的文件的路径名。
  - buf：这是一个指向 struct stat 结构的指针，用于存储获取到的path文件的信息。

- 返回值：
  - 成功：返回 0。
  - 失败：返回 -1，并且 errno 被设置以指示错误的原因。常见的出错原因有文件不存在、没有权限等。


## 基于dirent目录项结构体, 获取路径path

这里主要有两个方法:

- `chdir`到当前工作目录, 此时文件名就是相对路径
- 拼接dirent的目录名和文件名, 拼成绝对路径.

## 管理传入的stat结构体的buf指针

如果是局部变量的话, 那么由系统管理.

如果是动态分配的内存, 有开发者管理.

# stat结构体解析

这里列出一些常用的内部成员:

```c
struct stat {
    mode_t   st_mode;    // 包含文件的类型以及权限信息
    nlink_t  st_nlink;   // 文件的硬链接数量 
    uid_t    st_uid;     // 文件所有者的用户ID
    gid_t    st_gid;     // 文件所有者组的组ID
    off_t    st_size;    // 文件的实际大小, 以字节为单位
    // ...more
    struct timespec st_mtim;  /* 包含文件最后修改时间的结构体对象 */
};
#define st_mtime st_mtim.tv_sec
```

这些字段的类型都是使用别名来定义的，在64位Linux操作系统上，这些别名的类型一般是：

- mode_t：一般是一个32位无符号整数。
- nlink_t：一般是一个64位无符号整数。
- uid_t和gid_t：一般是一个32位无符号整数。
- off_t：一般是一个64位无符号整数。

关于timespec结构体, 代码如下:

```c
struct timespec {
    __time_t tv_sec; // 时间戳，秒为单位。此类型别名一般就是long类型
    __syscall_slong_t   tv_nsec; // 纳秒 - 存储时间戳当中不足秒的部分，用于精准表示时间。此类型别名一般就是long类型
};
```


















