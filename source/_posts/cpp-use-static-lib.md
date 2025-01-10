---
title: 在C++中使用.a静态库
comments: true
tags:
  - C++
  - 静态库
  - 编译与链接
categories: C/C++
abbrlink: a8135667
date: 2025-01-06 20:48:44
---

在archlinux上成功使用.a静态库. 


<!--more1-->


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

使用```g++ -c myadd.cpp -o myadd.o```来编译```myadd.cpp```成机器码```myadd.o```

使用```ar rcs libmyadd.a myadd.o```将机器码```myadd.o```打包成静态库```libmyadd.a```

```shell
# 汇编成机器码
g++ -c myadd.cpp -o myadd.o
# 打包
ar rcs libmyadd.a myadd.o
```

关于ar命令的详细内容,可以看我的这篇文章.

- [Linux中的ar命令](../8c6254fb/)

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

使用```g++ main.cpp -static -L. -lmyadd && ./a.out```获得如下输出

```
add(1, 2) = 3
```

## 一些注释

- ```-L. -lmyadd```参数的意思是在```./```目录去寻找```libmyadd.a```
- ```-static```将所有的库以静态的方式链入可执行程序, 这样生成的可执行程序, 不再依赖任何库



# 目录结构

```
test/
├── a.out
├── main.cpp
├── myadd.cpp
├── myadd.h
├── myadd.o
└── libmyadd.a
```


