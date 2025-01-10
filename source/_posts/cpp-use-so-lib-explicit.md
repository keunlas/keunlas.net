---
title: 在C++中显式的使用.so动态库
comments: true
tags:
  - C++
  - 动态库
  - 编译与链接
categories: C/C++
abbrlink: 5e4caed8
date: 2025-01-06 17:20:27
description: 显式的使用 .so 动态库需要用到dlfcn.h头文件，本文给出了使用示例。
---

在archlinux上成功实现了显式调用动态库.

总结如下
- 显示调用动态库需要 extern "C", 不然会找不到函数
- 加载动态库, dlopen
- 获取函数指针, dlsym
- 关闭动态库, dlclose

<!--more1-->


# 主要代码

## myadd.cpp

```cpp
// myadd.cpp

#define N 0

extern "C" int add(int x, int y) {
    return (x + y + N);
}
```

使用```g++ -fPIC -shared myadd.cpp -o myadd.so```来编译```myadd.cpp```成动态库```myadd.so```

```shell
# 上面的命令也可以分为这两步
# 先将 myadd.cpp 汇编成机器码 myadd.o
g++ -c -fPIC myadd.cpp -o myadd.o
# 再生成动态库
g++ -shared myadd.o -o myadd.so
```

## main.cpp

```cpp
// main.cpp
#include <dlfcn.h>
#include <iostream>

int main() {
    // 加载动态库 
    // RTLD_NOW 立即加载外部符号
    // RTLD_LAZY 懒加载
    auto *handle = dlopen("./myadd.so", RTLD_LAZY); 
    // 检查加载是否失败
    if (handle == nullptr) {
        std::cerr << "error: dlopen" << std::endl;
        return 1;
    }

    // 获取函数指针
    using MYADD_TYPE = int (*)(int, int);
    auto add = (MYADD_TYPE)dlsym(handle, "add");
    // 检查获取是否失败
    if (add == nullptr) {
        std::cerr << "error: dlsym" << std::endl;
        dlclose(handle);
        return 1;
    }

    // 调用函数
    int sum = add(1, 2);
    std::cout << "add(1, 2) = " << sum << std::endl;

    // 关闭动态库
    dlclose(handle);
    return 0;
}
```

使用```g++ main.cpp && ./a.out```获得如下输出

```
add(1, 2) = 3
```

成功完成动态库的显示调用.


# 目录结构

```
test/
├── a.out
├── main.cpp
├── myadd.cpp
└── myadd.so
```
