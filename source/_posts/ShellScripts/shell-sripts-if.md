---
title: Shell脚本 - if语句
comments: true
description: <center>Shell脚本 - if语句</center>
tags:
  - Shell
  - Linux
  - bash
  - if
  - Shell脚本
categories: Shell脚本
abbrlink: 953151c2
date: 2025-01-25 20:31:51
---

# 判断命令状态返回值

```bash
#!/bin/bash

if pwd; then
	echo "pwd works!!!"
else
	echo "pwd doesn't work!!!"
fi
```
```bash
#!/bin/bash

testuser=NotExistUser

if grep $testuser /etc/passwd; then
	echo "The user $testuser exists on this system"
elif ls -d /home/$testuser/; then
	echo "The user $testuser does not exist on this system"
	echo "But a directory named $testuser does"
else
	echo "The user $testuser does not exist on this system"
fi
```

# test判断条件

```bash
#!/bin/bash

if test; then
	echo "No expression returns a True"
else
	echo "No expression returns a False"
fi
# No expression returns a False

my_variable="HelloWorld"
if test $my_variable; then
	echo "The \"$my_variable\" expression returns a True"
else
	echo "The \"$my_variable\" expression returns a False"
fi
# The "HelloWorld" expression returns a True

my_variable=""
if test $my_variable; then
	echo "The \"$my_variable\" expression returns a True"
else
	echo "The \"$my_variable\" expression returns a False"
fi
# The "" expression returns a False
```

# 中括号判断条件

```
# 比较数字
# -eq	Equal to
# -ne	Not equal to
# -lt	Less than
# -le	Less than or equal to
# -gt	Greater than
# -ge	Greater than or equal to

# 检查字符串是否为空
# -z  String is null (length is zero)
# -n  String is not null (length is non-zero)
# =   Strings are equal
# !=  Strings are not equal
# <   String1 sorts before String2
# >   String1 sorts after String2


# 检查文件或目录是否存在及权限
# -f	File exists and is a regular file
# -d	File exists and is a directory
# -e	File exists
# -r	File exists and is readable
# -w	File exists and is writable
# -x	File exists and is executable
# -s	File exists and has a size greater than zero
# -O	File exists and is owned by the effective user ID
# -G	File exists and is owned by the effective group ID
# -nt	File1 is newer than File2
# -ot	File1 is older than File2


# 逻辑运算符(常用于连接多个条件)
# -a Logical AND
# -o Logical OR
# ! Logical NOT
```


# for

```bash
#!/bin/bash

for i in 1 2 3 4 5
do
	echo "The value is: $i"
done
# The value is: 1
# The value is: 2
# The value is: 3
# The value is: 4
# The value is: 5

for i in {1..5}
do
	echo "The value is: $i"
done
# The value is: 1
# The value is: 2
# The value is: 3
# The value is: 4
# The value is: 5
```

```bash
#!/bin/bash
# reading values from a file
file="states"
for state in $(cat $file)
do
	echo "Visit beautiful $state"
done
# $ cat states
# Alabama
# Alaska
# Arizona
# Arkansas
# Colorado
# Connecticut
# Delaware
# Florida
# Georgia
```


# IFS - 分隔符

Internal Field Separator

```bash
#!/bin/bash
# reading values from a file
#!/bin/bash
# reading values from a file

IFS_OLD=$IFS
IFS=$'\n'

file="states"
for state in $(cat $file); do
	echo "Visit beautiful $state"
done

IFS=$IFS_OLD

# $ cat states
# Alabama
# Alaska
# Arizona
# Arkansas
# Colorado
# Connecticut
# Delaware
# Florida
# Georgia
# New Jersey
```

# C语言风格分隔符

```bash
#!/bin/bash
# testing the C-style for loop
for (( i=1; i <= 10; i++ ))
do
	echo "The next number is $i"
done
```

```bash
#!/bin/bash
# multiple variables
for (( a=1, b=10; a <= 10; a++, b-- ))
do
	echo "$a - $b"
done
```