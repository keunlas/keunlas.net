---
title: CMake常用命令
comments: true
description: <center>《CMake构建实战：项目开发卷》读书笔记第四章。这一章的笔记，记得不是很好。第四章几乎全是一句话介绍命令，看这一章就像是在看文档一样。还是之后查文档吧。</center>
tags:
  - CMake
  - 笔记
categories: CMake构建实战：项目开发卷
abbrlink: dfeba917
date: 2025-01-15 11:10:28
---

这一章简单了解即可，随时查阅，不必仔细通读。

# 数值操作 math

```cmake
math(EXPR <结果变量> "<表达式字符串>" [OUTPUT_FORMAT <格式选项>])
```

- <结果变量> 运算结果
- <表达式字符串> 数学表达式，支持十进制和十六进制，与C语言相同
- <格式选项>
  - HEXADECIMAL 十六进制表示结果
  - DECIMAL 缺省值，十进制表示结果

# 字符串操作 string

## 搜索

```cmake
string(FIND <字符串> <子字符串> <结果变量> [REVERSE])
```

- 在<字符串>中搜索<子字符串>
- <第一次出现的位置存放到<结果变量>中
- REVERSE 反向搜索

## 替换

```cmake
string(REPLACE <匹配字符串> <替换字符串>
  <结果变量>
  <输入字符串> [<输入字符串>...]
)
```

- 会将若干<输入字符串>连接在一起
- 将其中出现的所有<匹配字符串>替换为<替换字符串>
- 并将最终结果存入<结果变量>


## 取字符串长度

```cmake
string(LENGTH <输入字符串> <结果变量>)
```




## 字符串变换

### 追加

```cmake
string(APPEND <字符串变量> [<输入字符串>...])
```

### 前插

```cmake
string(PREPEND <字符串变量> [<输入字符串>...])
```


### 连接

```cmake
string(CONCAT <结果变量> [<输入字符串>...])
```


### 分隔符连接

```cmake
string(JOIN <分隔符> <结果变量> [<输入字符串>...])
```

### 大小写转换

```cmake
string(TOLOWER <输入字符串> <结果变量>)
string(TOUPPER <输入字符串> <结果变量>)
```

### 重复字符串

```cmake
string(REPEAT <输入字符串> <重复次数> <结果变量>)
```

### 取子字符串


```cmake
string(SUBSTRING <输入字符串> <起始位置> <截取长度> <结果变量>)
```

### 删除首尾空白符

```cmake
string(STRIP <输入字符串> <结果变量>)
```

### 删除生成器表达式

```cmake
string(GENEX_STRIP <输入字符串> <结果变量>)
```

## 比较字符串



```cmake
string(COMPARE LESS <字符串1> <字符串2> <结果变量>) 
string(COMPARE GREATER <字符串1> <字符串2> <结果变量>) 
string(COMPARE NOTEQUAL <字符串1> <字符串2> <结果变量>) 
string(COMPARE LESS_EQUAL <字符串1> <字符串2> <结果变量>) 
string(COMPARE GREATER_EQUAL <字符串1> <字符串2> <结果变量>) 
string(COMPARE EQUAL <字符串1> <字符串2> <结果变量>) 
```

## 取哈希值

```cmake
string(<哈希算法> <结果变量> <输入字符串>)
```

- MD5 一种信息摘要（Message-Digest，MD）算法
- SHA1 第一代安全散列算法（Secure Hash Algorithm，SHA）
- SHA224 第二代安全散列算法，摘要长度为224位
- SHA256
- SHA384
- SHA512
- SHA3_224 SH第三代安全散列算法，又称Keccak算法，摘要长度为224位
- SHA3_256
- SHA3_384
- SHA3_512


## 字符串生成

### ASCII转字符串

```cmake
string(ASCII <ASCII值> [<ASCII值>...] <结果变量>)
```

### 字符串转十六进制表示

```cmake
string(HEX <输入字符串> <结果变量>)
```


### 生成C标识符

```cmake
string(MAKE_C_IDENTIFIER <输入字符串> <结果变量>)
```


### 生成随机文本

```cmake
string(RANDOM [LENGTH <长度>] 
  [ALPHABET <字符集合>]
  [RANDOM_SEED <随机种子>] 
  <结果变量>
)
```

### 生成时间戳

```cmake
string(TIMESTAMP <结果变量> [<时间戳格式>] [UTC])

```

时间戳格式描述符如下：

| 描述符 | 含义            |
| ------ | --------------- |
| %%     | 转义%           |
| %d     | 日 01-31        |
| %H     | 24进制小时      |
| %l     | 12进制小时      |
| %j     | 日 当年 001-366 |
| %m     | 月 01-12        |
| %b     | 月 英文缩写     |
| %B     | 月 英文         |
| %M     | 分 00-59        |
| %s     | UNIX时间        |
| %S     | 秒 00-60        |
| %U     | 周 当年 00-53   |
| %w     | 周 0-6          |
| %a     | 周 英文缩写     |
| %A     | 周 英文         |
| %y     | 年 两位数 00-99 |
| %Y     | 年 四位数       |



### 生成UUID


```cmake
string(UUID <结果变量> NAMESPACE <命名空间UUID> 
                       NAME <名称>
                       TYPE <MD5|SHA1> 
                       [UPPER]
)
```


### 字符串模板

```cmake
string(CONFIGURE <模板字符串> <结果变量> [@ONLY] [ESCAPE_QUOTES])
```







# 正则

## 正则语法

CMake所支持的正则表达式（regular expression）没有采用标准的语法，而是使用CMake自定义的一套简单语法，支持的特性并不丰富，但也足够日常使用。

| 语法结构 | 描述                                                                                                                            |
| -------- | ------------------------------------------------------------------------------------------------------------------------------- |
| ^        | 匹配输入的起始点                                                                                                                |
| $        | 匹配输入的终止点                                                                                                                |
| .        | 匹配任意单个字符                                                                                                                |
| \<char>  | 匹配指定的字符 or 转义特殊字符                                                                                                  |
| []       | 匹配方括号中任意某个字符                                                                                                        |
| [^]      | 匹配任意不在方括号中的字符                                                                                                      |
| -        | 在方括号中表示字符区间 or 放在开头和结尾时不表示区间，表示本身                                                                  |
| *        | 匹配前面的模式零次或多次                                                                                                        |
| +        | 匹配前面的模式一次或多次                                                                                                        |
| ?        | 匹配前面的模式零次或一次                                                                                                        |
| \|       | 匹配其两侧的模式之一                                                                                                            |
| ()       | 用于声明一个子表达式，可在字符串的正则替换命令中引用。正则匹配后，可通过变量CRAKE_MATCH_<n>获取各个子表达式的匹配结果，即捕获组 |


## 单次正则匹配

```cmake
string(REGEX MATCH <正则表达式>
  <结果变量>
  <输入字符串> [<输入字符串>...]
)
```

## 全部正则匹配

```cmake
string(REGEX MATCHALL <正则表达式>
  <结果变量>
  <输入字符串> [<输入字符串>...]
)
```

结果以列表的形式存入<结果变量中>

## 正则替换

```
string(REGEX REPLACE <正则表达式> <替换表达式>
  <结果变量>
  <输入字符串> [<输入字符串>...]
)
```

## 捕获组变量

捕获组（capture group）即正则表达式中子表达式匹配的结果。

捕获组可以在正则替换的替换表达式中通过反斜杠语法来访问。

除此之外，捕获组还可以在正则匹配后通过CMake的预定义变量来访问。

相关的预定义变量如下：

- `CMAKE_MATCH_COUNT` 其值为捕获组的数量
- `CMAKE_MATCH_<n>` 其值为第n个捕获组的匹配内容
- `CMAKE_MATCH_<n>` 当n为0时，其值为正则匹配的完整结果而非子表达式匹配的结果

这些变量会在正则匹配后被CMake赋值，这包括：

- 单次正则匹配后
- 全部正则匹配后（对应最后一次正则匹配的捕获组）
- 正则替换后（对应最后一次正则匹配的捕获组）
- 字符串匹配条件判断后（即if(<字符串变量> MATCHES <正则表达式>)）

<!-- 
# JSON操作

太多没必要，到时候再看。
 -->


# 列表操作 list

## 索引

list(GET <列表变量> <元素索引> [<元素索引>...] <结果变量>)

## 搜索

list(FIND <列表变量> <搜索值> <结果变量>)

## 长度

list(LENGTH <列表变量> <结果变量>)

## 追加

list(APPEND <列表变量> [<元素值>...])

## 前插

list(PREPEND <列表变量> [<元素值>...])


## 插入

list(INSERT <列表变量> <插入索引> [<元素值>...])


## 弹出末尾

list(POP_BACK <列表变量> [<结果变量>...])

## 弹出头部

list(POP_FRONT <列表变量> [<结果变量>...])

## 索引移除

list(REMOVE_AT <列表变量> <索引> [<索引>...])


## 移除元素值

list(REMOVE_ITEM <列表变量> <值> [<值>...])

## 移除重复元素

list(REMOVE_DUPLICATES <列表变量>)


## 连接列表

list(JOIN <列表变量> <分隔字符串> <结果变量>)


## 取子列表

list(SUBLIST <列表变量> <起始索引> <截取长度> <结果变量>)

## 列表筛选

list(FILTER <列表变量> <INCLUDE|EXCLUDE> REGEX <正则表达式>)

## 翻转列表

list(REVERSE <列表变量>)

## 排序列表

list(SORT <列表变量> [COMPARE <排序依据>] [CASE <大小写敏感>] [ORDER <排序方向>])

- 排序依据
  - STRING 字典序 缺省值
  - FILE_BASENAME 按路径排序
  - NATURAL 自然序
- 大小写敏感
  - SENSITIVE 启用大小写敏感 缺省值
  - INSENSITIVE 关闭
- 排序方向
  - ASCENDING 升序 缺省值
  - DESCENDING 降序

## 变换操作

list(TRANSFORM <列表变量> <变换操作> [<元素选择器>] [OUTPUT_VARIABLE <结果变量>])

也就是把列表元素当作字符串处理

## 元素选择器

元素选择器用于在列表中选择需要被变换的元素，可以是下列三种元素选择器之一

list(TRANSFORM ... AT <索引> [<索引>...] ...)

list(TRANSFORM ... FOR <起始索引> <终止索引> [<索引步进>] ...)

list(TRANSFORM ... REGEX <正则表达式> ...)



<!-- 
# 文件操作命令 file

等待以后再看

# 路径操作命令 get_filename_component

等待以后再看

# 配置模板文件 configure_file

等待以后再看

# 日志输出命令 message

等待以后再看
 -->



# 执行程序 execute_process

同其他脚本程序一样，
CMake脚本程序中也常常需要执行外部程序，
以进一步扩展CMake功能。
CMake提供了用于执行程序（进程）的命令execute_process，
使用起来非常简单，
功能也很完备。

其命令形式如下：



```cmake
execute_process(COMMAND <命令1> [<命令行参数>...]
  [COMMAND <命令2> [<命令行参数>...]]...
  [WORKING_DIRECTORY <工作目录>] # 设置工作目录
  [TIMEOUT <超时秒数>] # 设置超时时长
  [RESULT_VARIABLE <返回值变量>] # 获取进程返回值
  [RESULTS_VARIABLE <返回值列表变量>] # 获取进程返回值列表
  [OUTPUT_VARIABLE <标准输出变量>] # 设置输出变量
  [ERROR_VARIABLE <标准错误输出变量>] # 设置输出变量
  [INPUT_FILE <标准输入文件路径>] # 设置输入输出文件
  [OUTPUT_FILE <标准输出文件路径>] # 设置输入输出文件
  [ERROR_FILE <标准错误输出文件路径>] # 设置输入输出文件
  [OUTPUT_QUIET] # 屏蔽输出
  [ERROR_QUIET] # 屏蔽输出
  [COMMAND_ECHO<STDERR|STDOUT|NONE>] # 输出命令行调用
  [OUTPUT_STRIP_TRAILING_WHITESPACE] # 删除输出尾部空白
  [ERROR_STRIP_TRAILING_WHITESPACE] # 删除输出尾部空白
  [ENCODING <NONE|AUTO|ANSI|OEM|UTF8|UTF-8>] # 设置输出编码
  [ECHO_OUTPUT_VARIABLE]
  [ECHO_ERROR_VARIABLE]
  [COMMAND_ERROR_IS_FATAL <ANY|LAST>] # 设置失败条件
)
```

该命令的参数很多，
后面会分类介绍。
现在重点关注 COMMAND参数，
它后面跟着想要执行的命令（程序路径）
及其参数。
COMMAND参数可以多次指定以执行多个命令。



# 一点小吐槽


关于本章的内容，我只能说一句话。

重温填鸭式教育！

只能说填鸭式有填鸭式的好处，但不多。

其实这一章很多东西，并没有记录下来，因为真的实在是太多了，而且全是简单的一句话介绍命令。

还是到时候查文档吧。






























