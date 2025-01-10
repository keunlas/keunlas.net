---
title: C语言中的qsort排序函数
tags:
  - C
  - qsort
categories: C/C++
comments: true
abbrlink: 1b979af1
date: 2025-01-10 09:24:17
---


C语言中的qsort 函数是一个功能强大的标准库函数，调用它需要包含头文件<stdlib.h>。

正如它的函数名一样，它的作用是进行排序，而由于在函数内部采用快速排序算法(quick sort)，所以它被命名为"qsort"。

该函数的声明是：

```c
void qsort(void *base, size_t num, size_t size, int (*compare)(const void *, const void *));
```

- base：通用指针类型，表示要排序的数组，可以是任意类型数组。
- num：数组中的元素数量，也就是待排序的base数组的长度。
- size：base数组中每个元素的大小，通常使用 sizeof 运算符得到。
- compare：函数指针类型，表示该函数需要传入一个**返回值类型是int**，形参列表是```const void *, const void *```的表示比较规则的函数。

<!--more-->

# 比较规则和compare函数指针

```c
int compare(const void *a, const void *b);
```

qsort函数只会进行**升序排序**，compare函数就是用来判断两个元素的大小关系的。
- 返回值等于0，则 a == b.
- 返回值大于0，则 a > b.
- 返回值小于0，则 a < b.

如果我们想升序排序，只需要a小于b的时候返回负数，a大于b的时候返回正数，a等于b的时候返回零。

降序排序的逻辑和升序相反。当a实际上小于b的时候返回正数，那么qsort就会认为a逻辑上大于b，**而qsort函数只会进行升序排序**，所以a作为实际上小的数字，但是因为a逻辑上大，所以a会排在后面，这就相当于降序排序。



# 排序基本数据类型的数组

以int为例

```c
int my_cmp(const void *a, const void *b){
    // 将a和b转换成int*类型，然后才可以进行比较操作
    const int *num1 = a;
    const int *num2 = (const int*)b;    // 对于C语言而言，强转语法可写可不写
    // 后续表示大小规则的逻辑
}
```


# 排序自定义结构体数组

假设有一个自定义类型Student

```c
typedef struct{
    int stu_id;
    char name[25];
    int age;
}Student, Students[1024];

int my_cmp(const void *a, const void *b){
    // 将a和b转换成Student*类型，然后才可以进行比较操作
    const Student *s1 = a;
    const Student *s2 = (const Student*)b;  // 对于C语言而言，强转语法可写可不写
    // 后续表示大小规则的逻辑
}
```

# 排序C风格字符串数组

C风格字符串本身就是字符指针，所以字符串的指针本身就是二级指针，这一点非常值得注意。

```c
int my_cmp(const void *a, const void *b){
    // 将a和b转换成char*类型，然后才可以进行比较操作
    const char *s1 = *(const char **)a; 
    // 强转成二级指针再解引用一次，才能得到char*字符串的一级指针
    const char *s2 = *(const char **)b; 
    // 注意：此时的强转语法是不能省略的。
    // 因为通用指针类型是明确不能直接解引用的！！！

    // 后续表示大小规则的逻辑
}
```