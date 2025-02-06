---
title: CMake 基础语法
comments: true
abbrlink: 8fcb2c76
date: 2025-01-14 22:10:28
tags:
  - cmake
  - note
  - book
categories: CMake构建实战：项目开发卷
---

本文为《CMake构建实战：项目开发卷》一书, 第三章的学习笔记. 

# CMake程序

- CMakeLists.txt 文件

CMakeLists.txt 文件用于组织构建项目源程序的目录结构

- .cmake 后缀的文件

扩展名为 .cmake 的程序又分为脚本程序和模块程序两种类
型

脚本使用 cmake -P 执行, 与构建相关的命令不允许出现在脚本中

模块可以被 include 等命令引用, 是一种主要的代码复用单元

# 注释

```cmake
# 单行注释


#[[
  括号注释
]]


#[===[
  括号之间可以用多个等号
  等号的数量需要与起始标记相同
]===]
```


# 命令调用与参数

## 命令调用

和编程语言中的函数调用相同

```cmake

message(hello world) # 输出“helloworld”

message(a b c) # 输出“abc”

message(a;b;c) # 输出“abc”

```

空格和分号是分隔符


## 参数类型


### 引号参数

```cmake

# 引号参数
message("hello
world") 

message("hello\
world") 

```

引号中的所有字符都是参数, 包括空白和换行. 

在换行前输入反斜杠, 可以避免参数中出现换行. 


### 非引号参数

非引号参数中不能包含任何空白符, 也不能包含圆括号、#符号、双引号和反斜杠, 除非经过转义. 

以空格符号和分号切割. 



```cmake
message("x;y;z") # x;y;z
message(x y z) # xyz
message(x;y;z) # xyz
```

### 变量引用

```cmake

set(var_a hello)
set(var_b a)
message(${var_${var_b}})
# hello
```

set 为变量赋值

#{变量名} 将会替换为变量的值


### 括号参数
```cmake
message([===[
随便写终止方括号并不会导致文本结束, 
因此右边这两个括号]]也会包括在原始文本中. 
下一行中最后的括号也是原始文本的一部分, 
因为等号的数量与起始括号不匹配. ]==]
]===])

#随便写终止方括号并不会导致文本结束, 
#因此右边这两个括号]]也会包括在原始文本中. 
#下一行中最后的括号也是原始文本的一部分, 
#因为等号的数量与起始括号不匹配. ]==]
```

# 变量

## 分类

- 普通变量
- 缓存变量
- 环境变量


## 作用域

- 函数作用域：在用户自定义的函数命令中会有一个独立的作用域. 
- 目录作用域：对于CMake的目录程序而言, 每一个目录层级, 都有
它的一个作用域. 



## 保留标识符



- CMAKE_ (不区分大小写)
- \_CMAKE\_ (不区分大小写)
- _message (下划线开头加上CMake中任意预定义名称)



## 预定义变量

- `CMAKE_ARGC` 表示CMake脚本程序在被cmake -P命令行调用执行时, 命令行传递的参数个数. 
- `CMAKE_ARGV0`、`CMAKE_ARGV1`表示CMake脚本程序在被命令行调用执行时, 命令行传递的第一个、第二个参数. 如果有更多参数, 可以以此类推增加变量名称末尾的数值来获得. 
- `CMAKE_COMMAND` 表示CMake命令行程序所在的路径. 
- `CMAKE_HOST_SYSTEM_NAME` 表示宿主机操作系统（运行CMake的操作系统）名称. 
- `CMAKE_SYSTEM_NAME` 表示CMake构建的目标操作系统名称. 默认与宿主机操作系统一致, 一般用于交叉编译时, 由开发者显式设置. 
- `CMAKE_CURRENT_LIST_FILE` 表示当前运行中的CMake程序对应文件的绝对路径. 
- `CMAKE_CURRENT_LIST_DIR` 表示当前运行中的CMake程序所在目录的绝对路径. 
- `MSVC` 表示在构建时CMake当前使用的编译器是否为MSVC. 
- `WIN32` 表示当前目标操作系统是否为Windows. 
- `APPLE` 表示当前目标操作系统是否为苹果操作系统（包括macOS、iOS、tvOS、watchOS等）. 
- `UNIX` 表示当前目标操作系统是否为UNIX或类UNIX平台（包括Linux、苹果操作系统及Cygwin平台）. 

## 定义变量

### 普通变量

```
set(<变量名> <值>... [PARENT_SCOPE])
```

- 变量的值可以由若干参数来提供, 这些参数会被分号分隔连接成一个列表的形式, 作为最终的变量值. 
- 值参数被省略时, 相当于对该变量调用了unset命令. 
- 可选参数PARENT_SCOPE将变量定义到父级作用域中. （父目录 or 函数调用所在的作用域）

示例脚本代码：

```cmake
function(f)
  set(a "我是f的a")
  set(b "我是f的b")
  set(c "我是f的c" PARENT_SCOPE)
endfunction()

set(a "我是a")

f()
message("a: ${a}")
message("b: ${b}")
message("c: ${c}")
```

示例脚本输出为：

```
a: 我是a
b:
c: 我是f的c
```

### 缓存变量

```
set(<变量名> <值>... CACHE <变量类型> <变量描述> [FORCE])
```

- <变量类型> 有5种取值
  - BOOL 布尔类型
  - FILEPATH 文件路径类型
  - PATH 目录路径类型
  - STRING 文本型
  - INTERNAL 内部使用（隐含FORCE参数）
- <变量描述> 参数用于给出这个缓存变量的详细说明

**布尔类型缓存变量**还可以使用 option 命令定义：

```cmake
option(<变量名> <变量描述> [<ON|OFF>])
```

### 环境变量

```
set(ENV{<环境变量名>} [<值>])
```

直接看示例：

```cmake
# main.cmake
message("main \$ENV{PATH}: $ENV{PATH}")

set(ENV{PATH} "path") # 设置为 path
message("main \$ENV{PATH}: $ENV{PATH}")

execute_process(
  COMMAND ${CMAKE_COMMAND} -P setenv.cmake
  OUTPUT_VARIABLE out
)
message("${out}")

message("main \$ENV{PATH}: $ENV{PATH}")
```

```cmake
# setenv.cmake
message("before setenv \$ENV{PATH}: $ENV{PATH}")

set(ENV{PATH}) # 清空

message("after setenv \$ENV{PATH}: $ENV{PATH}")
```

示例输出为：
```
main $ENV{PATH}: C:\Program Files\...
main $ENV{PATH}: path
before setenv $ENV{PATH}: path
after setenv $ENV{PATH}:
main $ENV{PATH}: path
```

CMake的set命令定义的环境变量仅对当前CMake进程有效, CMake子进程将PATH环境变量的值清空并不影响CMake父进程中的PATH环境变量, 因此最终的输出仍是path. 


# 列表

列表只需要将多个元素用空格或者分号隔开即可

也可以混用


```cmake

set(list1 "a;b;c")
set(list2 a b c)
set(list3 a;b;c)

set(list4 a "b;c")

```

# 控制结构

## if

```cmake
if(<条件>)

  <命令>...

elseif(<条件>)

  <命令>...

else()

  <命令>...

endif()
```

## while

```cmake
while(<条件>)

  <命令>...

endwhile()
```

## foreach

```cmake
foreach(<循环变量> <循环项的列表>)

  <命令>...

endforeach()
```

### 区间遍历

```cmake
foreach(<循环变量> RANGE [<起始值>] <终止值> [<步进>])
```

- <起始值>被省略时, 默认为0
- <步进>被省略时, 默认为1
- <起始值>必须小于<终止值>
- <起始值>、<终止值>和<步进>这三个参数都是非负整数


### 高级列表遍历

```cmake
foreach(<循环变量> IN [LISTS [<列表变量名的列表>]] [ITEMS [<循项的列表>]])

```

### 打包遍历

```cmake
foreach(<循环变量>... IN ZIP_LISTS <列表变量名的列表>)

```

- 打包遍历会对每一个列表变量同时进行遍历, 并把各个列表当次遍历到的的元素赋值给不同的循环变量. 

- 如果只指定了一个<循环变量>, 那么当前遍历到的每一个列表变量的元素会依次赋值给“<循环变量>_<N>”（其中“N”对应列表变量的次序）

- 如果指定了多个<循环变量>, <循环变量>的个数应当与<列表变量名的列表>中的元素个数一致

- 遍历循环次数以最长的列表变量元素个数为准. 如果<列表变量名的列表>中某个列表变量的元素个数比其他列表少, 则遍历到后面时会将其对应元素的值视为空字符串

### break 和 continue

```cmake

break()

continue()

```


# 条件语法

| 常量类型 | 常量值                                                                            | 条件结果 |
| -------- | --------------------------------------------------------------------------------- | -------- |
| 真值常量 | 1、ON、YES、TRUE、Y、任何非零数值(不区分大小写)                                   | 真       |
| 假值常量 | 0、OFF、NO、FALSE、N、空字符串、NOTFOUND、以“-NOTFOUND”结尾的字符串(不区分大小写) | 假       |

## 逻辑运算

- AND
- OR
- NOT

## 单参数条件

```
if(COMMAND <命令名称>)
```

当<命令名称>确实存在时, 条件为真, 否则为假

其他的语法如下

| 条件语法                | 条件判断类型       | 备注                                               |
| ----------------------- | ------------------ | -------------------------------------------------- |
| COMMAND <命令名称>      | 命令判断           |                                                    |
| POLICY <策略名称>       | 策略判断           |                                                    |
| TARGET <目标名称>       | 目标判断           | add_executable, add_library, add_custom_target命令 |
| TEST <测试名称>         | 测试判断           | add_test命令                                       |
| DEFINED <变量名称>      | 变量定义判断       |                                                    |
| CACHE <缓存变量名称>    | 缓存变量定义判断   |                                                    |
| ENV{<环境变量名称>}     | 环境变量定义判断   |                                                    |
| EXISTS <文件或目录路径> | 文件或目录存在判断 | 要求绝对路径                                       |
| IS_DIRECTORY <目录路径> | 目录判断           | 要求绝对路径                                       |
| IS_SYMLINK <文件路径>   | 符号链接判断       | 要求绝对路径                                       |
| IS_ABSOLUTE <路径>      | 绝对路径判断       | 绝对路径为真, 否则为假                             |


## 双参数条件

### 数值比较

```
if(<字符串|变量> LESS <字符串|变量>)
```

存在变量则取变量的值, 否则取字符串本身. 

- LESS
- GREATER
- EQUAL
- LESS_EQUAL
- GREATER_EQUAL


### 字符串比较

- STRLESS
- STRGREATER
- STREQUAL
- STRLESS_EQUAL
- STRGREATER_EQUAL


### 字符串匹配

```
if(<字符串|变量> MATCHES <正则表达式>)
```

### 版本号比较

版本号格式如下

```
主版本号[.次版本号[.补丁版本号[.修订版本号]]]
```

- VERSION_LESS
- VERSION_GREATER
- VERSION_EQUAL
- VERSION_LESS_EQUAL
- VERSION_GREATER_EQUAL

### 列表元素判断

```
if(<字符串|变量> IN_LIST <列表变量>)
```

- IN_LIST




## 括号和条件优先级

CMake中条件语法求值的优先级由高到低依次为：

- 当前最内层括号中的条件
- 单参数条件
- 双参数条件
- 逻辑运算条件 NOT
- 逻辑运算条件 AND
- 逻辑运算条件 OR


# 命令定义

## 宏定义

在CMake中, 命令的名称不区分大小写, 宏作为一种命令, 其名称也不例外. 

```cmake
macro(<宏名> [<参数1>...])

  <命令>...

endmacro()

```

宏定义中通过set命令定义的变量, 在宏之外也能访问到. 

证实了宏不会产生作用域这一点. 


## 函数定义

```cmake
function(<函数名> [<参数1>...])

  <命令>...

endfunction()

```


函数会产生一个新的作用域, 因此函数内部直接使用set命令定义的变量是不能被外部访问的. 
为了实现这个目的, 必须为set命令指定PARENT_SCOPE参数, 使得变量定义到外部作用域. 


## 参数的访问

### 直接引用

```
function(my_func a b)
  set(result "参数a: ${a}, 参数b: ${b}" PARENT_SCOPE)
endfunction()
```

代码中的 `${a}`  `${b}` 即为引用形式参数


### 列表或索引访问

- `${ARGC}`表示参数的个数
- `${ARGV}`表示完整的实际参数列表
- `${ARGN}`表示无对应形式参数的实际参数列表, 其元素为从第(N+1)个用户传递的参数开始的每一个参数, N为函数或宏定义中形式参数的个数
- `${ARGV0}`、`${ARGV1}`、`${ARGV2}`依次表示第1个、第2个、第3个实际参数的值, 以此类推
 

### cmake_parse_arguments

```cmake
cmake_parse_arguments(
  <结果变量前缀名>
  <开关选项关键字列表> <单值参数关键字列表> <多值参数关键字列表>
  <将被解析的参数>...
)
```

- <结果变量前缀名>是一个前缀, 后面加上 `-<参数>` 即可访问到解析的参数
- 不同类型的参数应当分开写
- 末尾以 `${ARGN}` 结尾比较好

示例如下：

```cmake
function(abc_f)
  cmake_parse_arguments(abc "A0;A1" "B0;B1" [=[C0;C1]=] ${ARGN})
  message("A0: ${abc_A0}\nA1: ${abc_A1}")
  message("B0: ${abc_B0}\nB1: ${abc_B1}")
  message("C0: ${abc_C0}\nC1: ${abc_C1}")
endfunction()

abc_f(A0 A1 B0 a B1 b C0 x y C1 c d)
```

### cmake_parse_arguments针对函数优化的形式

```cmake
cmake_parse_arguments(PARSE_ARGV <N>
  <结果变量前缀名>
  <开关选项关键字列表> <单值参数关键字列表> <多值参数关键字列表>
)
```

- 该命令形式只能在函数中使用, 不支持在宏中使用
- 它直接对每一个函数参数进行解析, 因此无须通过列表的形式传递函数参数
- `<N>`是一个从0开始的整数, 表示从函数的第几个实际参数开始解析参数
- 前N个参数都是不需要关键字、需要调用者直接依次传参的参数


# 对这本书的吐槽

我来说一下几个比较影响观感的点吧：

- 变量名全是 a b c 之类的
- 示例代码无注释

总体来说, 第三章是将CMake的基础语法介绍了一遍. 但是这代码的观感真的很差, 原本我以为变量用 a b c 是网络段子, 没想到真让我看到了😰. 

顺便说一下第一章的一些小事故吧, 第一章最后一部分, 设置程序动态节中的 `RUNPATH` 这一个部分的命令在Archlinux上使用bash没法复现. 

最后是凭借我自己的知识, 试出来了一个可以正确复现的写法. 

























































































































































































































































































































































































































































































































































