---
title: CMake构建及其目标和属性
tags:
  - CMake
  - 笔记
categories: CMake构建实战：项目开发卷
comments: true
description: <center>《CMake构建实战：项目开发卷》读书笔记第六章和第七章。</center>
abbrlink: d5c757b3
date: 2025-01-15 20:42:38
---


# CMake项目的生命周期

## CMake的配置和生成

### 源代码以及CMake配置

```c
// lib.c
int add(int a, int b) {
  return a + b;
}
```

```cmake
cmake_minimum_required(VERSION 3.20)

# 定义项目名称和版本
project(mylib VERSION 1.0.0)

# 添加静态库目标mylib
# 其源代码包含lib.c
add_library(mylib STATIC lib.c)

# 生成安装规则为mylib构建目标
install(TARGETS mylib)

# CPack打包规则
# CPACK_PACKAGE_NAME 是打包后的名字
# CPACK_PACKAGE_NAME-VERSION
set(CPACK_PACKAGE_NAME "mylib")
# 放置打包源码的时候把build打包进去
set(CPACK_SOURCE_IGNORE_FILES
  ${CPACK_SOURCE_IGNORE_FILES}
  ${PROJECT_SOURCE_DIR}/build
)
include(CPack)
```

### 生成

```shell

# 可以额外指定 build type
# -DCMAKE_BUILD_TYPE=Debug
# -DCMAKE_BUILD_TYPE=Release
cmake -S . -B ./build

```

### 构建

CMake有四种构建模式：
- Debug调试模式，禁用代码优化，便于调试。
- Release发布模式，启用代码优化并针对速度优化，启用内联并丢失调试符号，几乎无法调试。
- RelWithDebInfo发布调试模式，启用代码优化，但保留符号且不会内联函数，仍可调试。
- MinSizeRel最小体积发布模式，启用代码优化，但针对二进制体积进行优化，使其尽可能小。

```shell
cmake --build ./build --config Release
```

命令执行完成后，可以在build目录中找到构建好的静态库libmylib.a

### 安装

```shell

cmake --install ./build --prefix ./install

```

- `--install ./build`指定了CMake的构建目录
- `--prefix ./install`指定了libmylib.a将被安装在哪个目录的`lib`子目录中

### 打包

```shell

cd build

# 打包程序 使用 TGZ
cpack -G TGZ --config CPackConfig.cmake
# 打包源代码 使用 TGZ
cpack -G TGZ --config CPackSourceConfig.cmake

```

- `mylib-1.0.0-Linux.tar.gz`
- `mylib-1.0.0-Source.tar.gz`

打包好之后这些文件就会出现在`_CPack_Packages`目录中，以及`build`目录中

如果是`Windows`系统的话，可以使用`NSIS`打包程序安装包的可执行文件。



# 构建目标和属性

## 可执行文件目标

```cmake

add_executable(<目标名称>
  [WIN32][MACOSX_BUNDLE]
  [EXCLUDE_FROM_ALL]
  [<源文件>...]
)

```

- `WIN32` 表示使用`WinMain`而不是`main`作为入口函数
- `MACOSX_BUNDLE` 仅用于Apple平台中，表示构建为bundle, 常用语图形界面程序
- `EXCLUDE_FROM_ALL` 表示只有在`cmake --build`的时候，用`--target`手动地构建时，才会被构建


## 一般库目标文件

```cmake
add_library(<目标名称> <库类型>
  [EXCLUDE_FROM_ALL]
  [<源文件>...]
)
```

<库类型>参数有以下三个取值：
- `STATIC`，代表该构建目标为静态库构建目标
- `SHARED`，代表该构建目标为动态库构建目标
- `MODULE`，代表该构建目标为模块库构建目标。模块库是一种插件形式的动态链接库，不会在构建时被链接到任何一个程序中，仅用于运行时动态链接（通过LoadLibrary或dlopen等API）。

## 接口目标库

```cmake
add_library(<目标名称> INTERFACE)
```

接口库目标通常用于对头文件库的抽象，同样使用add_library命令创建。

该命令会创建一个目标文件库的构建目标，只有<目标名称>和INTERFACE两个参数。毕竟，接口库自身并不需要被构建，也就无须指定源文件。


## obj文件库目标

```cmake
add_library(<目标名称> OBJECT
  [<源文件>...]
)
```

使用`$<TARGET_OBJECTS:<目标文件库的目标名称>>`，
就可以在构建其他可执行文件或者库的时候，
连接obj文件库对应的obj文件。

示例如下：

```cmake
cmake_minimum_required(VERSION 3.20)
project(myObj)

add_library(myObj OBJECT
  test01.c
  test02.c
)

add_executable(main
  main.c
  $<TARGETS_OBJECTS:myObj>
)

```

## 指定源文件的方式

在项目开发中，最佳的实践是：
永远一一罗列全部源文件。


尽管使用aux_source_directory可以遍历指定目录中的全部源文件。
但不是最佳实践。

```
aux_source_directory(<目录> <结果变量>)
```

## 导入外部目标

### 外部可执行程序

```cmake
cmake_minimum_required(VERSION 3.20)
project(import-notepad)

add_executable(notepad_exe IMPORTED)

set_target_properties(notepad_exe PROPERTIES
  IMPORTED_LOCATION "C:/Windows/System32/notepad.exe")

add_custom_target(run-notepad ALL notepad_exe ${CMAKE_CURRENT_LIST_FILE})
```

### 库导入目标

```cmake
add_library(<目标名称>
  <STATIC|SHARED|MODULE|UNKNOWN|OBJECT|INTERFACE>
  IMPORTED [GLOBAL])
```

### 按构建模式导入

```cmake
add_library(test SHARED IMPORTED)

set_property(TARGET test PROPERTY
  IMPORTED_CONFIGURATIONS DEBUG;RELEASE 
)

set_target_properties(test PROPERTIES
  IMPORTED_LOCATION_DEBUG "testd.so"
  IMPORTED_LOCATION_RELEASE "test.so"
)
```


### 别名目标

```cmake
add_executable(<目标名称> ALIAS <指向的实际目标名称>)
add_library(<目标名称> ALIAS <指向的实际目标名称>)
```

示例如下：

```cmake
cmake_minimum_required(VERSION 3.20)
project(alias-target)

add_library(my_liba STATIC "a.cpp")

# 判断是否使用外部liba
if(USE_EXTERNAL_LIBA)
  add_library(liba STATIC IMPORTED)
  set_target_properties(liba PROPERTIES IMPORTED_LOCATION "liba.lib")
else()
  # 不使用外部，可以添加别名，使用内部的my_liba
  add_library(liba ALIAS my_liba)
endif()

add_executable(main main.cpp)
target_link_libraries(main liba)
```

# 子目录：add_subdirectory

```cmake

add_subdirectory(<源文件目录> [<二进制目录>] [EXCLUDE_FROM_ALL])

```


# 项目：project

简单形式：
```
project(<项目名称> [<编程语言>...])
```

详细形式：
```
project(<项目名称>
  [VERSION <主版本号>[.<次版本号>[.<补丁版本号>[.<修订版本号>]]]]
  [DESCRIPTION <项目描述>]
  [HOMEPAGE_URL <项目主页URL>]
  [LANGUAGES <编程语言>...])
```

## 获取和指定项目相关信息

| 变量                     | 描述                               |
| ------------------------ | ---------------------------------- |
| PROJECT_NAME             | 获取当前最近项目的名称             |
| CMAKE_PROJECT_NAME       | 获取顶层项目的名称                 |
| <项目名称>_SOURCE_DIR    | 指定项目的源文件目录的绝对路径     |
| <项目名称>_BINARY_DIR    | 指定项目的二进制文件目录的绝对路径 |
| <项目名称>_VERSION       | 指定项目的版本号                   |
| <项目名称>_VERSION_MAJOR | 指定项目的主版本号                 |
| <项目名称>_VERSION_MINOR | 指定项目的次版本号                 |
| <项目名称>_VERSION_PATCH | 指定项目的补丁版本号               |
| <项目名称>_VERSION_TWEAK | 指定项目的修订版本号               |
| <项目名称>_DESCRIPTION   | 指定项目的描述文本                 |
| <项目名称>_HOMEPAGE_URL  | 指定项目的主页URL                  |

## 代码注入

| 变量                                    | 描述                          |
| --------------------------------------- | ----------------------------- |
| CMAKE_PROJECT_INCLUDE_BEFORE            | 将CMake程序注入所有项目定义前 |
| CMAKE_PROJECT_INCLUDE                   | 将CMake程序注入所有项目定义后 |
| CMAKE_PROJECT_<项目名称>_INCLUDE_BEFORE | 将CMake程序注入指定项目定义前 |
| CMAKE_PROJECT_<项目名称>_INCLUDE        | 将CMake程序注入指定项目定义后 |

# 属性：property

- 全局属性
- 目录属性
- 目标属性
- 源文件属性
- 缓存变量属性

# 属性相关命令

## 设置目标链接库：target_link_libraries

```cmake
target_link_libraries(<构建目标> <库文件|库目标>...)

target_link_libraries(<构建目标>
  <PRIVATE|INTERFACE|PUBLIC> <库文件|库目标>...
  [<PRIVATE|INTERFACE|PUBLIC> <库文件|库目标>...]...
)
```

默认为`PUBLIC`

- PRIVATE参数表示将紧跟其后的<库文件|库目标>仅设置为<构建目标>的构建要求。
- INTERFACE参数表示将紧跟其后的<库文件|库目标>仅设置为<构建目标>的使用要求。
- PUBLIC参数表示将紧跟其后的<库文件|库目标>同时设置为<构建目标>的构建要求和使用要求。

## 设置头文件目录：include_directories

```cmake
include_directories([AFTER|BEFORE] [SYSTEM] <目录>...)
```

默认为`AFTER`，
即追加到头文件目录列表的末尾。

## 设置目标头文件目录：target_include_directories


```cmake
target_include_directories(<构建目标>
  [SYSTEM] [AFTER|BEFORE]
  <PRIVATE|INTERFACE|PUBLIC> <目录>...
  [<PRIVATE|INTERFACE|PUBLIC> <目录>...]...
)
```

该命令用于将<目录>加入到<构建目标>的头文件搜索目录列表中。
<目<><><录>可以是绝对路径或相对于当前源文件目录的相对路径。
其他参数与`include_directories`命令中的对应参数功能一致。


## 设置链接库：link_libraries

```cmake
link_libraries([库文件或库目标]...)
```

此命令之后的构建目标，将被设置链接库属性。

**不推荐使用**，请使用`target_link_libraries`.


## 设置链接目录：link_directories

```cmake
link_directories([AFTER|BEFORE] <目录>...)
```

默认为 `AFTER`

该命令仅对当前目录及其子目录中的构建目标生效，
用于将<目录>设置为构建目标的链接库搜索目录。
<目录>可以是绝对路径或相对于当前源文件目录的相对路径。



## 设置目标链接目录：target_link_directories

```cmake
target_link_directories(<构建目标>
  [BEFORE]
  <PRIVATE|INTERFACE|PUBLIC> <目录>...
  [<PRIVATE|INTERFACE|PUBLIC> <目录>...]...
)
```

## 设置目标源文件：target_sources

```cmake
target_sources(<构建目标>
  <PRIVATE|INTERFACE|PUBLIC> <源文件>...
  [<PRIVATE|INTERFACE|PUBLIC> <源文件>...]...
)
```

有了这个命令，
就不必在创建目标的时候把全部源文件设置好了，
甚至可以在创建目标时不提供任何源文件，
而后再通过该命令为构建目标设置源文件。



# 一点小吐槽

感觉看这本书，像是在看官方文档。

只不过这本书最大的优点是，它是中文。


