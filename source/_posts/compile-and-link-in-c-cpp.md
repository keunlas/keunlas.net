---
title: C/C++中的编译与链接
comments: true
tags:
  - C
  - C++
  - 编译与链接
categories: C/C++
abbrlink: dfd36be8
date: 2025-01-07 19:39:22
description: C/C++中的编译和链接分为四个阶段：预处理、编译、汇编和链接。
---

编译与链接主要为下述步骤，其中前三个步骤就是广义上的编译。

- 预处理，把一个```.c```源文件处理成```.i```预处理文件。
- 编译，把```.i```预处理文件进一步处理成```.s```汇编文件。（狭义上的编译）
- 汇编，把```.s```汇编文件最终处理得到```.o```机器码文件。
- 链接，把多个```.o```机器码文件链接成可执行文件。

<!--more1-->


# 准备源代码

为了演示这个过程，我编写了以下两个文件作为源代码。

```h
// myheader.h
#define N 666
#define M 999
```

```c
// main.c

#include "myheader.h"
/**
 * 我是注释
 * 我也是注释
 */
int main() {
  // 我还是注释
  int i = M + N;
  return 0;
}
```

接下来，我将以此文件为基础进行演示。

# 广义上的编译

![广义上的编译过程](../assets/202501070001.png "广义上的编译过程")

## 预处理

首先是预处理，通过```gcc -E main.c -o main.i```命令将main.c预处理成main.i

main.i文件内容如下：

```c
# 0 "main.c"
# 0 "<built-in>"
# 0 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 0 "<command-line>" 2
# 1 "main.c"
# 1 "myheader.h" 1
# 2 "main.c" 2






int main() {

  int i = 999 + 666;
  return 0;
}
```

可以看的出来```#include```和```#define```这些预处理指令，都已经成功的执行。

- ```#include```成功的把```myheader.h```引入到了```main.c```中
- ```#define```的宏定义也完成了文本替换
- 源代码中的注释也全部被去掉了


## 编译（狭义上的编译）


接着是狭义上的编译，通过```gcc -S main.i -o main.s```命令将main.i编译成main.s

main.s文件内容如下：

```c
	.file	"main.c"
	.text
	.globl	main
	.type	main, @function
main:
.LFB0:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movl	$1665, -4(%rbp)
	movl	$0, %eax
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	main, .-main
	.ident	"GCC: (GNU) 14.2.1 20240910"
	.section	.note.GNU-stack,"",@progbits
```

可以看的出来预处理好的文件已经被编译成了汇编代码。

懂一些汇编的话，应该能看出来```movl	$1665, -4(%rbp)```这一行汇编代码就是给变量i赋值，rbp寄存器偏移-4的位置就是i的地址。


## 汇编

接着是汇编，把汇编代码汇编成机器码，通过```gcc -c main.s -o main.o```命令将main.s汇编成main.o

main.o是机器码文件，已经是一个二进制文件了。之前讲过的打包静态库所使用到的就是```.o```文件。

详见[在C++中使用.a静态库](../a8135667/)和[Linux中的ar命令](../8c6254fb/)这两篇文章。


# 链接

最后我们可以使用```gcc main.o -o main```将编译获得的多个```.o```文件链接成为可执行程序。在本文的例子中，只有一个```.o```文件。

最终```main```就是可执行程序。



# 目录结构

```
test/
├── main
├── main.c
├── main.i
├── main.s
├── main.o
└── myheader.h
```

