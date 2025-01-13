---
title: CMake初步使用
comments: true
description: <center>使用CMake进行项目的构建，简单初探。</center>
tags:
  - C语言
  - C++
  - CMake
categories: C/C++项目构建
abbrlink: ce53047e
date: 2025-01-12 17:23:54
---

# 准备要构建的试验项目

## 目录结构

```
.
├── add
│   ├── add.cpp
│   ├── add.h
│   └── CMakeLists.txt
├── CMakeLists.txt
└── main.cpp
```

## 源代码

```cpp
// add.h
#ifndef ADD_H
#define ADD_H

#define N 1000

int add_two_num(int a, int b);
int add_N(int a);

#endif // !ADD_H
```

```cpp
// add.cpp
#include "add.h"

int add_two_num(int a, int b) {
  return a + b;
}

int add_N(int a) {
  return a + N;
}
```

```cpp
// main.cpp
#include <iostream>
using std::cout, std::endl;
#include "add.h"

int main() {
  cout << "add_two_num(5, 6) = " << add_two_num(5, 6) << endl;
  cout << "add_N(6) = " << add_N(6) << endl;
  return 0;
}
```

# 编写 CMakeLists.txt

## add子目录下的cmake文件

```cmake
# ./add/CMakeLists.txt
# 生成一个全局可见的目标
add_library(add OBJECT
  add.cpp
)

# 将 add 目录添加到公共包含目录中
# 这样 main.cpp 可以直接 #include "add.h"
target_include_directories(add PUBLIC .)

```

## 根目录下的cmake文件

```cmake
# ./CMakeLists.txt
# 指定最低的CMake版本
cmake_minimum_required(VERSION 3.31)
# 定义语言和元数据
project(cmake_starter # ${PROJECT_NAME}
  VERSION 0.0.1
  DESCRIPTION "This project is for cmake_starter."
  LANGUAGES CXX)

# 生成可执行文件
add_executable(${PROJECT_NAME} main.cpp)

# 添加子目录
add_subdirectory(add)

# 将 add 连接到 可执行文件
target_link_libraries(${PROJECT_NAME} PRIVATE add)


```

# 执行构建

```shell
# -S . 指定源文件目录为当前目录
# -B ./build 指定构建目录为build目录
# 建议把构建目录写入 .gitignore
# 该命令把构建文件输出到 build 目录中
cmake -S . -B ./build

# --build 表示执行构建
# build 即为构建目录
# 该命令利用构建目录中的文件进行整个项目的构建
cmake --build build

```

