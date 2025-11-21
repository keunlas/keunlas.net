---
title: 《SQLite3》SQLite3 C/C++ 基础接口（01）
comments: true
tags:
  - sqlite
  - sqlite3
categories:
  - SQLite
abbrlink: 8f11705a
date: 2025-11-21 23:08:25
---

# 介绍

最基本的使用 sqlite3 的 c api 还是挺简单的。只需要记住三个接口即可。

- 打开数据库：`sqlite3_open()`

- 关闭数据库：`sqlite3_close()`

- 执行 SQL 语句：`sqlite3_exec()`

# 开启数据库

先来看看如何开启数据库，我们需要提供两个参数，数据库名称和一个 sqlite3 指针的指针。

```c
int sqlite3_open(
  const char *filename,   /* Database filename (UTF-8) */
  sqlite3 **ppDb          /* OUT: SQLite db handle */
);
```

具体的使用基本如下：

```c
char* db_name = "test.db";
sqlite3* db;
sqlite3_open(db_name, &db);
```

SQLite3 还提供了可以更加精细的控制打开数据库方式的接口：`sqlite3_open_v2()`

```c
int sqlite3_open_v2(
  const char *filename,   /* Database filename (UTF-8) */
  sqlite3 **ppDb,         /* OUT: SQLite db handle */
  int flags,              /* Flags */
  const char *zVfs        /* Name of VFS module to use */
);
```

可以看到多了两个参数，可以控制数据库只读或其他更精细的操作。`sqlite3_open()` 可以看作是下述特定参数 `sqlite3_open_v2()`。

```c
char* db_name = "test.db";
sqlite3* db;
sqlite3_open_v2(db_name, &db, SQLITE_OPEN_READWRITE | SQLITE_OPEN_CREATE, 0);
```

具体的 `sqlite3_open_v2()` 使用可以参考官方文档 [Opening A New Database Connection](https://sqlite.org/c3ref/open.html).

# 关闭数据库

关闭数据库只需要把 `sqlite3* db` 作为参数传入即可。使用十分简单。

就像 open 接口有多个版本一样，close 接口也有多个版本。

```c
int sqlite3_close(sqlite3*);
int sqlite3_close_v2(sqlite3*);
```

主要区别是，当 db 还有某些未完成的任务时，两个接口会有不同的行为。

`sqlite3_close` 会返回 `SQLITE_BUSY`。

`sqlite3_close_v2` 会返回 `SQLITE_OK`。但不会立即释放数据库连接，而是将数据库连接标记为不可用的“僵尸”状态，并安排在所有预处理语句完成、所有 BLOB 句柄关闭以及所有备份操作结束后自动释放数据库连接。



# 执行 SQL 语句

```c
int sqlite3_exec(
  sqlite3* db,                                  /* 数据库连接 */
  const char* sql,                              /* 要执行的 SQL 语句 */
  int (*callback)(void*, int, char**, char**),  /* 回调函数 */
  void* data,                                   /* 传递给回调函数的参数 */
  char** errmsg                                 /* 错误信息指针 */
);
```

`db` 是打开的数据库连接。

`sql` 是要执行的 SQL 语句。

`callback` 是执行完 SQL 语句后进行的回调函数。（可选参数，不需要可传递空指针）

`data` 是传给 `callback` 的第一个参数，可用于传递上下文。（可选参数，不需要可传递空指针）

`errmsg` 会返回相应的错误信息，如果无错误则被设为空指针。（可选参数，不需要可传递空指针）

## 回调函数

```c
int callback(void* context, int col_size, char** col_values, char** col_names);
```

只要理解了这个回调函数会**对结果的每一行数据执行一次**，我们就可以轻松的理解这个回调的后面三个和列相关的参数。

- `context`：从 sqlite3_exec() 传递过来的用户数据
- `col_size`：结果行的列数
- `col_values`：列值的字符串数组
- `col_names`：列名的字符串数组

返回值为 `0` 时，继续查询。

返回值为 `1` 时，终止查询。
