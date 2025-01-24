---
title: Linux目录流
comments: true
description: '<center>Linux目录流, POSIX标准.</center>'
tags:
  - C语言
  - Linux
  - 目录流
  - dirent
  - POSIX
categories: Linux系统编程
abbrlink: 2912b58b
date: 2025-01-21 19:53:28
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

```c
#include <dirent.h>     // dirent是directory entry的简写，就是目录项的意思
#include <sys/types.h>
DIR *opendir(const char *name);
```

形式参数：name: 字符串，代表要打开的目录的路径。

返回值：
- 成功时，返回指向 DIR 类型的指针，即目录流的指针
- 失败时，返回 NULL 并设置 errno 以指示错误的原因。常见的失败原因，比如目录没有权限打开、目录不存在等。


## 关闭目录流


```c
#include <sys/types.h>
#include <dirent.h>
int closedir(DIR *dirp);  
```

形式参数：dirp: 由 opendir 返回的目录流指针。

返回值：
- 成功：返回0
- 失败时，返回 -1 并设置 errno 以指示错误的原因。常见的错误原因就是乱传指针，传入的指针不是打开的目录流指针。

## 读目录流

当你使用 opendir 打开一个目录流后，可以使用 readdir 依次读取目录中的每个条目，直到目录中没有更多条目为止。

```c
#include <dirent.h>
struct dirent *readdir(DIR *dirp);
```

形式参数：dirp: 由 opendir 返回的目录流指针。

返回值：
- 返回值: 成功时，返回指向 struct dirent 结构体对象的指针，这个结构体当中包含了目录下文件和子目录的信息（如文件名等）。
- 当目录中没有更多条目时返回 NULL。
- 当函数出错时该函数也会返回NULL。但只要传给函数的指针是一个打开的目录流指针，该函数一般不会出错，所以可以把返回NULL作为读完目录流的标记，而不需要做错误处理。

### dirent结构体

directory entry也就是目录项.

是当前目录下的某个文件/子目录的目录项结构体，这个结构体中会存储当前目录下的某个文件/子目录的信息。

```c
// dirent是directory entry的简写，就是目录项的意思
struct dirent {
 ino_t          d_ino;       // 此目录项的inode编号，目录项中会存储文件的inode编号。一般是一个64位无符号整数（64位平台）
 off_t          d_off;       // 到下一个目录项的偏移量。可以视为指向下一个目录项的指针(近似可以看成链表)，一般是一个64位有符号整数
 unsigned short d_reclen;    // 此目录项的实际大小长度，以字节为单位(注意不是目录项所表示文件的大小，也不是目录项结构体的大小)
 unsigned char  d_type;      // 目录项所表示文件的类型，用不同的整数来表示不同的文件类型
 char           d_name[256]; // 目录项所表示文件的名字，该字段一般决定了目录项的实际大小。也就是说文件名越长，目录项就越大
};
```

其中文件类型d_type的可选值如下(使用宏常量定义的整数)：

```c
DT_BLK      // 块设备文件，对应整数值6
DT_CHR      // 字符设备文件，对应整数值2
DT_DIR      // 目录文件，对应整数值4
DT_FIFO     // 有名管道文件，对应整数值1
DT_LNK      // 符号链接文件，对应整数值10
DT_REG      // 普通文件，对应整数值8
DT_SOCK     // 套接字文件，对应整数值12
DT_UNKNOWN  // 未知类型文件，对应整数值0
```

> 返回的这个结构体指针指向的是静态对象, 所以不需要手动free.



## 移动目录流指针的位置

告知当前的位置.

```c
#include <dirent.h>
long telldir(DIR *dirp);
```

到达记录的位置.

```c
#include <dirent.h>
void seekdir(DIR *dirp, long loc);
```

回到一开始.

```c
#include <dirent.h>
void rewinddir(DIR *dirp);
```


# 总结

目录的相关操作可以获取到很多文件信息

- inode编号
- 文件类型
- 文件大小
- 文件名称

不过想要获得更多的信息, 需要用到另一个系统调用函数`stat`.

