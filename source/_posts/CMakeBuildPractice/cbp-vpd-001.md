---
title: CMake基础语法
comments: true
description: <center>《CMake构建实战：项目开发卷》读书笔记，因为第一章的内容比较简单通俗，早已掌握。第二章的内容无实质性的知识，只是对CMake的介绍。所以笔记从第三章开始。</center>
abbrlink: 8fcb2c76
date: 2025-01-14 22:10:28
tags:
  - CMake
  - 笔记
categories: CMake构建实战：项目开发卷
---

本文为《CMake构建实战：项目开发卷》一书，第三章的学习笔记。

前两章内容可以看我这一类别的文章：[C/C++项目构建](/categories/C-C-项目构建/)


# CMake程序

- CMakeLists.txt 文件

CMakeLists.txt 文件用于组织构建项目源程序的目录结构

- .cmake 后缀的文件

扩展名为 .cmake 的程序又分为脚本程序和模块程序两种类
型

脚本使用 cmake -P 执行，与构建相关的命令不允许出现在脚本中

模块可以被 include 等命令引用，是一种主要的代码复用单元

# 注释

```cmake
# 单行注释


#[[
  括号注释
]]


#[===[
  括号之间可以用多个等号
  等号的数量需要与起始标记相同
]===]
```


# 命令调用与参数

## 命令调用

和编程语言中的函数调用相同

```cmake

message(hello world) # 输出“helloworld”

message(a b c) # 输出“abc”

message(a;b;c) # 输出“abc”

```

空格和分号是分隔符


## 参数类型


### 引号参数

```cmake

# 引号参数
message("hello
world") 

message("hello\
world") 

```

引号中的所有字符都是参数，包括空白和换行。

在换行前输入反斜杠，可以避免参数中出现换行。


### 非引号参数

非引号参数中不能包含任何空白符，也不能包含圆括号、#符号、双引号和反斜杠，除非经过转义。

以空格符号和分号切割。



```cmake
message("x;y;z") # x;y;z
message(x y z) # xyz
message(x;y;z) # xyz
```

### 变量引用

```cmake

set(var_a hello)
set(var_b a)
message(${var_${var_b}})
# hello
```

set 为变量赋值

#{变量名} 将会替换为变量的值


### 括号参数
```cmake
message([===[
随便写终止方括号并不会导致文本结束，
因此右边这两个括号]]也会包括在原始文本中。
下一行中最后的括号也是原始文本的一部分，
因为等号的数量与起始括号不匹配。]==]
]===])

#随便写终止方括号并不会导致文本结束，
#因此右边这两个括号]]也会包括在原始文本中。
#下一行中最后的括号也是原始文本的一部分，
#因为等号的数量与起始括号不匹配。]==]
```

# 变量

## 分类

- 普通变量
- 缓存变量
- 环境变量


## 作用域

- 函数作用域：在用户自定义的函数命令中会有一个独立的作用域。
- 目录作用域：对于CMake的目录程序而言，每一个目录层级，都有
它的一个作用域。



## 保留标识符



- CMAKE_ (不区分大小写)
- \_CMAKE\_ (不区分大小写)
- _message (下划线开头加上CMake中任意预定义名称)



## 预定义变量

- `CMAKE_ARGC` 表示CMake脚本程序在被cmake -P命令行调用执行时，命令行传递的参数个数。
- `CMAKE_ARGV0`、`CMAKE_ARGV1`表示CMake脚本程序在被命令行调用执行时，命令行传递的第一个、第二个参数。如果有更多参数，可以以此类推增加变量名称末尾的数值来获得。
- `CMAKE_COMMAND` 表示CMake命令行程序所在的路径。
- `CMAKE_HOST_SYSTEM_NAME` 表示宿主机操作系统（运行CMake的操作系统）名称。
- `CMAKE_SYSTEM_NAME` 表示CMake构建的目标操作系统名称。默认与宿主机操作系统一致，一般用于交叉编译时，由开发者显式设置。
- `CMAKE_CURRENT_LIST_FILE` 表示当前运行中的CMake程序对应文件的绝对路径。
- `CMAKE_CURRENT_LIST_DIR` 表示当前运行中的CMake程序所在目录的绝对路径。
- `MSVC` 表示在构建时CMake当前使用的编译器是否为MSVC。
- `WIN32` 表示当前目标操作系统是否为Windows。
- `APPLE` 表示当前目标操作系统是否为苹果操作系统（包括macOS、iOS、tvOS、watchOS等）。
- `UNIX` 表示当前目标操作系统是否为UNIX或类UNIX平台（包括Linux、苹果操作系统及Cygwin平台）。

## 定义变量

### 普通变量

```
set(<变量名> <值>... [PARENT_SCOPE])
```

- 变量的值可以由若干参数来提供，这些参数会被分号分隔连接成一个列表的形式，作为最终的变量值。
- 值参数被省略时，相当于对该变量调用了unset命令。
- 可选参数PARENT_SCOPE将变量定义到父级作用域中。（父目录 or 函数调用所在的作用域）

示例脚本代码：

```cmake
function(f)
  set(a "我是f的a")
  set(b "我是f的b")
  set(c "我是f的c" PARENT_SCOPE)
endfunction()

set(a "我是a")

f()
message("a: ${a}")
message("b: ${b}")
message("c: ${c}")
```

示例脚本输出为：

```
a: 我是a
b:
c: 我是f的c
```

### 缓存变量

```
set(<变量名> <值>... CACHE <变量类型> <变量描述> [FORCE])
```

- <变量类型> 有5种取值
  - BOOL 布尔类型
  - FILEPATH 文件路径类型
  - PATH 目录路径类型
  - STRING 文本型
  - INTERNAL 内部使用（隐含FORCE参数）
- <变量描述> 参数用于给出这个缓存变量的详细说明

**布尔类型缓存变量**还可以使用 option 命令定义：

```cmake
option(<变量名> <变量描述> [<ON|OFF>])
```

### 环境变量

```
set(ENV{<环境变量名>} [<值>])
```

直接看示例：

```cmake
# main.cmake
message("main \$ENV{PATH}: $ENV{PATH}")

set(ENV{PATH} "path") # 设置为 path
message("main \$ENV{PATH}: $ENV{PATH}")

execute_process(
  COMMAND ${CMAKE_COMMAND} -P setenv.cmake
  OUTPUT_VARIABLE out
)
message("${out}")

message("main \$ENV{PATH}: $ENV{PATH}")
```

```cmake
# setenv.cmake
message("before setenv \$ENV{PATH}: $ENV{PATH}")

set(ENV{PATH}) # 清空

message("after setenv \$ENV{PATH}: $ENV{PATH}")
```

示例输出为：
```
main $ENV{PATH}: C:\Program Files\...
main $ENV{PATH}: path
before setenv $ENV{PATH}: path
after setenv $ENV{PATH}:
main $ENV{PATH}: path
```

CMake的set命令定义的环境变量仅对当前CMake进程有效，CMake子进程将PATH环境变量的值清空并不影响CMake父进程中的PATH环境变量，因此最终的输出仍是path。


# 列表

p124：待写：2025-01-14