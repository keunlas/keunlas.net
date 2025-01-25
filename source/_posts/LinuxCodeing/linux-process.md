---
title: Linux进程操作
comments: true
description: <center></center>
date: 2025-01-25 14:36:48
tags:
  - Linux
  - 进程
  - 进程间通信
categories: Linux系统编程
---

# 创建进程



## system

```c
#include <stdlib.h>

int system(const char *command);
```

`system` 函数会调用 `/bin/sh -c` 来执行参数 `command` 指定的命令，并返回命令的退出状态。



## fork

```c
#include <sys/types.h>
#include <unistd.h>

pid_t fork(void);
```

`fork` 函数会创建一个子进程，子进程是父进程的副本，包括进程的地址空间、文件描述符、信号处理程序等。子进程和父进程共享代码段，但是拥有独立的进程空间。

`fork` 函数返回两次，一次在父进程中，一次在子进程中。
- 在父进程中，`fork` 函数返回子进程的进程 ID。
- 在子进程中，`fork` 函数返回 0。
- 如果 `fork` 函数调用失败，则返回 -1。




## exec

```c
#include <unistd.h>
int execl(const char *path, const char *arg, ... /* (char  *) NULL */);
int execv(const char *path, char *const argv[]);
int execle(const char *path, const char *arg, ... /*, (char *) NULL, char *const envp[] */);
int execve(const char *path, char *const argv[], char *const envp[]);
int execlp(const char *file, const char *arg, ... /* (char  *) NULL */);
int execvp(const char *file, char *const argv[]);
int execvpe(const char *file, char *const argv[], char *const envp[]);
```
`exec` 函数族用于在当前进程中加载并执行一个新的程序。这些函数会替换当前进程的地址空间，包括代码段、数据段、堆栈段等。`exec` 函数族不会创建新的进程，而是直接替换当前进程。

`exec` 函数族中的函数有以下几个参数：

- `path`：要执行的程序的路径。
- `arg`：要执行的程序的参数列表，以 `NULL` 结尾。
- `envp`：环境变量列表，以 `NULL` 结尾。

`exec` 函数族中的函数有以下几个区别：

- `execl` 和 `execv` 函数使用 `path` 参数指定要执行的程序的路径，使用 `arg` 参数指定要执行的程序的参数列表。
- `execl` 函数使用可变参数列表，而 `execv` 函数使用 `argv` 数组。
  






# 退出进程

## exit

```c
#include <stdlib.h>
void exit(int status);
#include <stdlib.h>
void _Exit(int status); 
#include <unistd.h>
void _exit(int status;
```

`exit` 函数用于终止当前进程，并将 `status` 作为进程的退出状态。`exit` 函数会关闭所有打开的文件描述符，释放进程的所有资源，并通知父进程进程已经退出。`exit` 函数会等待所有子进程退出，然后终止进程。






