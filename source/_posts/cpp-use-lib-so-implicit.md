---
title: 在C++中隐式的使用.so动态库
comments: true
tags:
  - C++
  - 动态库
  - 编译与链接
categories: C/C++
abbrlink: acafc476
date: 2025-01-06 18:14:20
---

在archlinux上成功实现了隐式调用动态库. 

虽然遇到了一些小问题, 但是在网络搜索引擎的帮助下, 成功解决. 


<!--more-->


# 主要代码

## myadd.cpp和myadd.h

```cpp
// myadd.h
int add(int a, int b);
```
```cpp
// myadd.cpp
#include "myadd.h"
#define N 0

int add(int x, int y) { 
  return x + y + N; 
}
```

使用```g++ -fPIC -shared myadd.cpp -o libmyadd.so```来编译```myadd.cpp```成动态库```libmyadd.so```

## main.cpp

```cpp
// main.cpp
#include <iostream>
#include "myadd.h"

int main()
{
    int sum = add(1, 2);
    std::cout << "add(1, 2) = " << sum << std::endl;
    return 0;
}
```

使用```g++ main.cpp -L. -lmyadd && ./a.out```获得如下输出

```-L. -lmyadd```参数的意思是在```./```目录去寻找```libmyadd.so```

```
./a.out: error while loading shared libraries: libmyadd.so: cannot open shared object file: No such file or directory
```

动态库的隐式调用失败.

# 寻找问题发生的原因

为什么```g++ main.cpp -L. -lmyadd```这条命令成功的在```./```当前目录找到了动态库并且编译连接, 但是运行时, 却找不到动态库呢?

这是由于在编译链接期间, 其找寻```.so```是根据```-L```和默认的```/usr/lib```, ```/lib```和```LIBRARY_PATH```

而在运行期间,寻找```.so```是根据```/usr/lib```,```/lib```和```LD_LIBRARY_PATH```

# 解决问题

```shell
# 先临时的把当前目录 /path/to/test/ 添加到 LD_LIBRARY_PATH
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/test/
# 然后运行程序
./a.out
```

成功获得输出

```
add(1, 2) = 3
```

成功完成动态库的隐式调用.


# 目录结构

```
test/
├── a.out
├── main.cpp
├── myadd.cpp
├── myadd.h
└── libmyadd.so
```


