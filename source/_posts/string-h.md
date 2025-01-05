---
title: C语言中的string.h头文件
tags:
  - C
  - string.h
categories: C/C++
comments: true
abbrlink: 46aa2375
date: 2025-01-05 17:43:17
---


主要的几个函数如下所示.

```c
size_t  strlen  (const char *str);
char   *strcpy  (char *dest, const char *src);
char   *strncpy (char *dest, const char *src, size_t n);
char   *strcat  (char *dest, const char *src);
char   *strncat (char *dest, const char *src, size_t n);
int     strcmp  (const char *str1, const char *str2);
```

<!--more-->

# strlen

```c
size_t my_strlen(const char *str);
```

- 返回当前字符串的长度, 即字符串末尾空字符```'\0'```前面的字符数量.




# strcpy和strncpy

```c
char *strcpy(char *dest, const char *src);
char *strncpy(char *dest, const char *src, size_t n);
```

返回值为指向dest的指针.

- ```strcpy```
  - ```strcpy```函数将```src```中以空字符```'\0'```结束的字符串复制到```dest```中
  - 返回指向```dest```的指针
  - 如果```dest```不够大, 就会因数组越界而产生未定义行为
- ```strncpy```
  - ```strncpy```函数会将最多n个字符从```src```中复制到```dest```中
  - ```n <= strlen(src)``` 时src末尾```'\0'```不会被复制过去
  - ```n > strlen(src)``` 剩余的字符由```'\0'```补上
  
实际开发中, 建议将n的值设定为dest长度-1, 并主动将dest的最后一个元素设置为空字符, 这样总能安全地得到一个字符串.




# strcat和strncat

```c
char *strcat(char *dest, const char *src);
char *strncat(char *dest, const char *src, size_t n);
```

返回值为指向dest的指针.

函数将src拼接到dest末尾, **假设dest_end为dest末尾空字符的地址**, 则函数可以如下理解.

- ```strcat(dest, end);```
  - 可以理解为```strcpy(dest_end, src);```
- ```strncat(dest, end, n);```
  - 可以理解为```strncpy(dest_end, src, n);```

实际开发中可以这样设置n
```c
// dest数组的长度 - 已存储字符串的长度 - 1 (留1个位置给空字符)
int n = sizeof(dest) - strlen(dest) - 1;    
```
并主动将dest的最后一个元素设置为空字符, 这样总能安全地得到一个字符串.


# strcmp


```c
int strcmp(const char *str1, const char *str2);
```

strcmp函数按照字典顺序比较两个字符串的大小, 即ASCII码的编码值大的一方大.

该函数会依次比较同一位置的字符大小, 直到两字符不同或者到达'\0'字符.

- 若str1和str2所有字符相同, 则返回值为```0```
- str1大, 返回值为正
- str2大, 返回值为负
  
若有一方到达了空字符, 另一方未到达空字符, 由于空字符的编码值为0, 必定小于其他字符, 即 **字符串长度大的 > 字符串长度小的**



