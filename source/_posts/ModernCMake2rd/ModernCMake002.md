---
title: CMake依赖关系
comments: true
tags:
  - cmake
  - note
  - book
categories: Modern CMake for C++ 2rd Edition
abbrlink: 5586ae64
date: 2025-01-12 18:06:44
---

# 引出例子

假设有一个银行项目, 这个项目有五个target. 

- Calculations
- Drawing
- TerminalApp
- GuiApp
- Checksum

这五个target的依赖关系如下图所示. 

![银行项目依赖关系图](/assets/202501120001.png)

# 理清关系

我们就可以这样去编写cmake的列表文件. 

```cmake
cmake_minimum_required(VERSION 3.26)
project(BankApp CXX)

add_executable(terminal_app terminal_app.cpp)
add_executable(gui_app gui_app.cpp)

target_link_libraries(terminal_app calculations)
target_link_libraries(gui_app calculations drawing)

add_library(calculations calculations.cpp)
add_library(drawing drawing.cpp)

add_custom_target(checksum ALL
  COMMAND sh -c "cksum terminal_app>terminal.ck"
  COMMAND sh -c "cksum gui_app>gui.ck"
  BYPRODUCTS terminal.ck gui.ck
  COMMENT "Checking the sums..."
)

add_dependencies(checksum terminal_app gui_app)
```

这里我们看到了列表文件中先`target_link_libraries()`
再`add_library()`. 这实际上是没有问题的. 它们之间的依赖关系是不会乱的. 

但是为了保证checksum的依赖关系, 在末尾添加了一行`add_dependencies(checksum terminal_app gui_app)`, 这一行显式的宣布了checksum的依赖关系. 


