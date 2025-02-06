---
title: CMake系统与环境
comments: true
tags:
  - cmake
  - note
  - book
categories: Modern CMake for C++ 2rd Edition
abbrlink: 7c3e5a98
date: 2025-01-19 16:29:03
---

# 检测操作系统

不同的操作系统之间, 很多细节都不同.
在Windows和Unix中, 大小写敏感, 文件路径结构, 扩展名等等,都是非常不同的.
我们可以检查`CMAKE_SYSTEM_NAME`变量, 以便采取不同的行动.

示例如下:

```cmake
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  message(STATUS "Doing things with Linux")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
  message(STATUS "Doing things with Darwin")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  message(STATUS "Doing things with Windows")
elseif(CMAKE_SYSTEM_NAME STREQUAL "AIX")
  message(STATUS "Doing things with AIX")
else()
  message(STATUS "This is ${CMAKE_SYSTEM_NAME}.")
endif()
```

但是有一个建议, 那就是尽量让解决方案与系统无关.

比如对文件系统的操作, 可以使用CMake内置的file()命令.


# 交叉编译

交叉编译是指在一个机器上编译代码以在另一个目标平台上执行的过程. 例如, 使用适当的工具集, 可以在 Windows 机器上运行 CMake 来编译 Android 应用程序. 

将`CMAKE_SYSTEM_NAME`和`CMAKE_SYSTEM_VERSION`设置为适当的值是必要操作.

不过无论这两个值配置如何, 主机的信息都可以通过`HOST`查到. 例如: `CMAKE_HOST_SYSTEM`, `CMAKE_HOST_SYSTEM_NAME`等.

# 系统变量简写

- ANDROID, APPLE, CYGWIN, UNIX, IOS, WIN32, WINCE, WINDOWS_PHONE
- CMAKE_HOST_APPLE, CMAKE_HOST_SOLARIS, CMAKE_HOST_UNIX, CMAKE_HOST_WIN32

UNIX的值对于Linux, macOS, Cygwin都为真.

WIN32, CMAKE_HOST_WIN32的值对于32位和64位的Windows都为真.

# 主机系统信息

CMake可以查看主机的系统信息.

```cmake
cmake_host_system_information(RESULT <VARIABLE> QUERY <KEY>...)
```

其中, 关键字有很多, 这里列举几类.

## 环境与操作系统信息

![](/assets/202501190001.png)

## 处理器的信息

![](/assets/202501190002.png)

![](/assets/202501190003.png)



# 32位还是64位的平台

可以通过指针的大小来判断, 32位指针为4字节, 64位指针为8字节.

```cmake

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  message(STATUS "Target is 64 bits")
endif()

```

# 字节序

大多数情况下, 字节序并不重要.
但是当编写需要可移植性的位操作代码时, CMake 将提供BIG_ENDIAN 或 LITTLE_ENDIAN 值, 这些值存储在 `CMAKE_<LANG>_BYTE_ORDER` 变量中, 其中 `<LANG>` 是 C、CXX、OBJC 或 CUDA. 




