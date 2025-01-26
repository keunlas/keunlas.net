---
title: Linux进程操作
comments: true
description: <center>主要包括创建进程, 退出进程, 进程间通信以及信号.</center>
tags:
  - Linux
  - 进程
  - 进程间通信
  - 信号
categories: Linux系统编程
abbrlink: 9d03ce17
date: 2025-01-25 14:36:48
---


# Linux命令

## 探查进程(ps, top)



```sh
# -e所有进程
# -f详细信息
ps -ef
# -l长列表
ps -l
# l长列表, BSD风格
ps l
#
ps aux
# --forest显示进程间父子关系, GNU风格
ps --forest
# 实时查看进程
top
```


## 如何看进程信息

**UNIX SYSTEM V 风格**

```
F (标志): 显示进程的标志，这些标志是内核用于处理进程的不同属性的。
S (状态): 显示进程的状态。常见状态包括 R/运行中, S/睡眠中, D/不可中断的睡眠状态, Z/僵尸进程, T/停止或被跟踪, I/空闲进程(不活跃进程: I约等于Z).
UID (用户ID): 显示启动进程的用户的用户ID。
PID (进程ID): 显示进程的ID。
PPID (父进程ID): 显示创建当前进程的父进程的ID。
C (CPU使用率): 显示进程占用的CPU使用率的近似值。
PRI (优先级): 显示进程的优先级。
NI (nice值): 显示进程的nice值，这是一个影响进程调度优先级的值。
ADDR (地址): 显示进程的内存地址。
SZ (大小): 显示进程使用的内存大小。
WCHAN (等待通道): 如果进程是睡眠的，这里显示进程等待的资源。
STIME (开始时间): 显示进程开始的时间。
TTY (终端): 显示进程绑定的终端设备。
TIME (CPU时间): 显示进程使用的CPU时间总计。
CMD (命令): 显示启动进程的命令名或命令行。
```

**UNIX BSD 风格**

```
USER: 进程的所有者
PID: 进程ID
%CPU: 进程占用的CPU百分比
%MEM: 进程占用的内存百分比
VSZ: 进程使用的虚拟内存量（KB）
RSS: 进程占用的固定内存量（KB）（常驻集大小）
TTY: 进程的终端类型
STAT: 进程的状态 (附加值Eg: <高优先级进程 s进程是会话领导  l多线程  +位于后台的进程组...)
START: 进程的启动时间
TIME: 进程占用CPU的时间
COMMAND: 启动进程的命令
```




## 结束进程(kill, killall)

```sh
# 显示所有信号
kill -l
# 以异常方式终止进程
kill -9 pid
```

```sh
kill <PID>
kill -s HUP <PID>

killall <name(可使用通配符)>
```


# 库函数

## 创建进程



### system

```c
#include <stdlib.h>

int system(const char *command);
```

`system` 函数会调用 `/bin/sh -c` 来执行参数 `command` 指定的命令，并返回命令的退出状态。



### fork

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




### exec

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
  






## 退出进程

### exit

```c
#include <stdlib.h>
void exit(int status);
#include <stdlib.h>
void _Exit(int status); 
#include <unistd.h>
void _exit(int status;
```

`exit` 函数用于终止当前进程，并将 `status` 作为进程的退出状态。`exit` 函数会关闭所有打开的文件描述符，释放进程的所有资源，并通知父进程进程已经退出。`exit` 函数会等待所有子进程退出，然后终止进程。




## 进程控制


### 孤儿进程

如果父进程先于子进程退出，则子进程成为孤儿进程，此时将自动被PID为1的进程收养,  PID为1的进程就成为了这个进程的父进程。当一个孤儿进程退出以后，它的资源清理会交给它的父进程来处理。

### 僵尸进程

如果一个进程已经终止，但是其父进程还没有调用 `wait` 或 `waitpid` 函数来获取其退出状态，那么这个进程就被称为僵尸进程。僵尸进程会占用系统资源，因此需要及时处理。


## wait and waitpid

在C语言中，`wait` 和 `waitpid` 是用于进程控制的系统调用，通常用于父进程等待子进程的终止并获取其退出状态。这两个函数定义在 `<sys/wait.h>` 头文件中。

### `wait` 函数

#### 函数原型：
```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t wait(int *status);
```

#### 功能：
`wait` 函数用于等待任意一个子进程终止，并获取其退出状态。如果没有子进程终止，父进程会阻塞，直到有一个子进程终止。

#### 参数：
- `status`：指向一个整型变量的指针，用于存储子进程的退出状态。可以通过一些宏来解析这个状态值（如 `WIFEXITED`, `WEXITSTATUS`, `WIFSIGNALED`, `WTERMSIG` 等）。如果不需要获取状态，可以传入 `NULL`。

#### 返回值：
- 成功时，返回终止的子进程的进程ID（PID）。
- 失败时，返回 `-1`，并设置 `errno`。

#### 示例：
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        // 子进程
        printf("Child process is running\n");
        sleep(2);
        exit(42);  // 子进程退出，返回状态码42
    } else if (pid > 0) {
        // 父进程
        int status;
        pid_t child_pid = wait(&status);

        if (WIFEXITED(status)) {
            printf("Child process %d exited with status %d\n", child_pid, WEXITSTATUS(status));
        }
    } else {
        // fork失败
        perror("fork");
        exit(EXIT_FAILURE);
    }

    return 0;
}
```

### `waitpid` 函数

#### 函数原型：
```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t waitpid(pid_t pid, int *status, int options);
```

#### 功能：
`waitpid` 函数用于等待指定的子进程终止，并获取其退出状态。与 `wait` 不同，`waitpid` 可以指定要等待的子进程，并且可以通过 `options` 参数控制其行为。

#### 参数：
- `pid`：指定要等待的子进程的PID。
  - `pid > 0`：等待进程ID为 `pid` 的子进程。
  - `pid == -1`：等待任意子进程，等同于 `wait`。
  - `pid == 0`：等待与调用进程同组的任意子进程。
  - `pid < -1`：等待进程组ID等于 `pid` 绝对值的任意子进程。
- `status`：与 `wait` 函数中的 `status` 参数相同，用于存储子进程的退出状态。
- `options`：控制 `waitpid` 的行为，常用的选项有：
  - `WNOHANG`：如果没有子进程终止，立即返回，而不是阻塞。
  - `WUNTRACED`：如果子进程被暂停（例如通过 `SIGSTOP` 信号），也返回其状态。
  - `WCONTINUED`：如果子进程从暂停状态恢复（例如通过 `SIGCONT` 信号），也返回其状态。

#### 返回值：
- 成功时，返回终止的子进程的进程ID（PID）。
- 如果指定了 `WNOHANG` 选项且没有子进程终止，返回 `0`。
- 失败时，返回 `-1`，并设置 `errno`。

#### 示例：
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        // 子进程
        printf("Child process is running\n");
        sleep(2);
        exit(42);  // 子进程退出，返回状态码42
    } else if (pid > 0) {
        // 父进程
        int status;
        pid_t child_pid = waitpid(pid, &status, 0);

        if (WIFEXITED(status)) {
            printf("Child process %d exited with status %d\n", child_pid, WEXITSTATUS(status));
        }
    } else {
        // fork失败
        perror("fork");
        exit(EXIT_FAILURE);
    }

    return 0;
}
```

### 状态宏

`wait` 和 `waitpid` 返回的状态可以通过以下宏进行解析：

- `WIFEXITED(status)`：如果子进程正常退出（通过 `exit` 或 `return`），返回真。
- `WEXITSTATUS(status)`：如果 `WIFEXITED` 为真，返回子进程的退出状态码。
- `WIFSIGNALED(status)`：如果子进程因为信号而终止，返回真。
- `WTERMSIG(status)`：如果 `WIFSIGNALED` 为真，返回导致子进程终止的信号编号。
- `WIFSTOPPED(status)`：如果子进程被暂停（例如通过 `SIGSTOP` 信号），返回真。
- `WSTOPSIG(status)`：如果 `WIFSTOPPED` 为真，返回导致子进程暂停的信号编号。
- `WIFCONTINUED(status)`：如果子进程从暂停状态恢复（例如通过 `SIGCONT` 信号），返回真。

### 总结

- `wait` 用于等待任意一个子进程终止，而 `waitpid` 可以指定等待特定的子进程。
- `waitpid` 提供了更多的控制选项，如 `WNOHANG` 可以避免阻塞。
- 通过状态宏可以解析子进程的退出状态，了解子进程是正常退出还是被信号终止。

这两个函数在多进程编程中非常有用，尤其是在需要父进程管理子进程的生命周期时。



































