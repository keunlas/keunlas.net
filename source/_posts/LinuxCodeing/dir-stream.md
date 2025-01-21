---
title: Linux目录流
comments: true
description: '<center>Linux目录流, POSIX标准.</center>'
tags:
  - C语言
  - Linux
  - 目录流
categories: Linux系统编程
abbrlink: 2912b58b
date: 2025-01-21 20:53:28
---

# 基础目录操作

## chmod

```c
#include <sys/stat.h> // 使用该函数需要包含的头文件
int chmod(const char *pathname, mode_t mode);
```
- 形参
  - `pathname`：文件或目录的路径字符串。
  - `mode`：要设置的新权限，这里要使用权限数字表示法，即八进制数（C语言中八进制整数需要以"0"开头）。
- 返回值
  - 成功时，chmod 返回 0。
  - 失败时，返回 -1，并设置 errno 以指示错误原因。

> 需要注意的是：设置权限时需要传参权限的"数字表示法"，  
> 而且设定的权限就是文件的最终权限，  
> 掩码只会影响新建文件，不会影响chmod函数。

## getcwd

```c
#include <unistd.h>     // 需要包含此头文件调用函数
char *getcwd(char *buf, size_t size);
```

- 形式参数：
  - buf: 指向存放当前工作目录字符串的字符数组
  - size: 这个数组的大小
- 返回值：
  - 成功时，getcwd 返回一个指向 buf 的指针，buf 中包含了当前工作目录的绝对- 路径。
  - 失败时，返回 NULL，并设置 errno 以指示错误的原因。出错的原因普遍是buf- 数组过小，无法容纳整个工作目录绝对路径字符串。


当buf为NULL, 并且size为0时, 由getcwd负责申请空间, 但是需要开发者去释放, 所以不推荐.

## chdir

```c
#include <unistd.h>
int chdir(const char *path);
```

- 形式参数：
  - path: 是一个指向字符数组的指针，这个数组包含了新工作目录的路径。
  - 路径可以是绝对的（从根目录开始，例如 "/usr/local/bin"）或相对的（相对于当前工作目录，例如 "../docs"）。
- 返回值：
  - 成功时，chdir 返回 0。
  - 失败时，返回 -1，并设置全局变量 errno 以指示错误的原因。比较常见的错误，比如目标目录不存在、权限问题、目标不是目录等。


当前工作目录是进程的属性，也就是说每一个进程都有自己的当前工作目录。而且父进程创建子进程的时候，子进程会继承父进程的当前工作目录。


## mkdir


```c
#include <sys/stat.h>
#include <sys/types.h>
int mkdir(const char *pathname, mode_t mode);
```

- 形式参数：
  - pathname: 要创建的目录的路径名。可以是绝对路径也可以是相对路径名。
  - mode: 设置新目录的权限。这是一个八进制数，类似于 chmod 的权限设置（例如，0775）。注意，最终的权限还会受到进程的 umask 设置的影响。
- 返回值：
  - 成功时，mkdir 返回 0。
  - 失败时，返回 -1，并设置 errno 以指示错误的原因。常见错误比如目录已存在等。

新建一个目录设定权限为777，你可能会发现最终生成的目录权限为775, 这是因为文件掩码 umask 的影响
- 文件掩码 umask 是一个三位的八进制数，用于在创建新文件或者目录时移除一些权限。
- 比如umask若是0002，指定权限为777，最终结果就会是775，相当于"新权限 - 掩码"才得到新权限
- 可以通过指令umask来查看当前掩码值，也可以用umask n来设定一个新的掩码值。
- 掩码的存在是为了系统安全，避免新文件目录拥有过多的权限。较低的 umask 值会导致更开放的权限，而较高的 umask 值会导致更严格的权限。

## rmdir

rmdir（remove directory）系统调用函数用于删除一个空目录，注意只能删除空目录。

```c
#include <unistd.h>
int rmdir(const char *pathname);
```

- 返回值：
  - 成功时，rmdir 返回 0。
  - 失败时，返回 -1，并设置 errno 以指示错误的原因。常见的错误原因有目录非空，不存在或无权限等。


# 目录流操作


| 文件流 | 目录流    |
| ------ | --------- |
| fopen  | opendir   |
| fclose | closedir  |
| fread  | readdir   |
| fwrite | ×         |
| ftell  | telldir   |
| fseek  | seekdir   |
| rewind | rewinddir |


## 打开目录流




## 关闭目录流





## 读目录流






