---
title: Shell指令速记
comments: true
tags:
  - linux
  - shell
categories: 随手记
abbrlink: 615a942b
date: 2025-01-20 21:22:17
---

# 探查进程(ps, top)



```sh
# -e所有进程
# -f详细信息
ps -ef
# -l长列表
ps -l
# l长列表, BSD风格
ps l
# --forest显示进程间父子关系, GNU风格
ps --forest
# 实时查看进程
top
```

# 结束进程(kill, killall)

```ch
kill <PID>
kill -s HUP <PID>

killall <name(可使用通配符)>
```

# 查看磁盘(du)

```ch
# 查看总的大小(磁盘块数)
du -c
# 人类可阅读的形式(空间大小)
du -h
# 只输出总的合计
du -s
du -chs
```

# 排序(sort)
```ch
# 字符序
sort file
# 数字序
sort -n file
# 月份大写序
sort -M file
```

还有很多序...








