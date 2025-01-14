---
title: 如何书写 .gitignore 文件
tags:
  - Git
  - .gitignore
categories: 知识速记
comments: true
description: <center>.gitignore规则的书写。</center>
abbrlink: 47a1fb62
date: 2025-01-13 15:11:32
---

虽然已经用了多年的git，但是至今都没怎么好好的看一下ignore规则。

今天心血来潮，重新仔细的看一下 .gitignore 到底怎么好好的去写。

这篇文章，也将作为知识速记这个类别的首篇文章。

参考文章：[.gitignore File – How to Ignore Files and Folders in Git](https://www.freecodecamp.org/news/gitignore-file-how-to-ignore-files-and-folders-in-git/)

# 如何在 Git 中忽略一个文件或文件夹

## 如何忽略一个文件或目录

```
/text.txt           # 忽略根目录下 text.txt 文件
/test/text.txt      # 忽略根目录下 test 目录中的 text.txt 文件

text.txt            # 忽略所有名为 text.txt 的文件和目录
test/               # 忽略所有名为 test 的目录

test                # 忽略所有名字带有 test 的文件和目录
img*                # 忽略所有名字开头是 img 的文件和目录
*.md                # 忽略所有名字结尾是 .md 的文件和目录
```

## 如何不忽略一个文件或目录

加入我们忽略了所有md文件，但唯独不忽略README.md的情况，如下所示。

```
.md                 # 忽略所有名字带有 .md 的文件和目录
!README.md          # 不忽略所有名字带有 README.md 的文件和目录
```

## 无法排除已经被忽略的目录内的一个文件

```
test/               # 忽略所有名字带有 test 的目录
!test/example.md    # 试图在被忽略的目录内排除一个文件是行不通的
```

# 如何忽略以前提交的文件

假如你不小心把 .build 文件提交了，现在想去除这个文件。过程如下所示。

```sh

# 给 .gitignore 添加 .build 文件
echo ".build" >> .gitignore

# 从 git 版本库中去除该文件
git rm --cached .build

# 暂存新的 .gitignore 并提交
git add .gitignore
git commit -m "更新了 .gitignore 文件"

```








