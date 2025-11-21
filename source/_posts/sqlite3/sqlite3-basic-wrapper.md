---
title: 《SQLite3》面向对象包装 SQLite3 C/C++ 基础接口（02）
comments: true
abbrlink: 813fa779
date: 2025-11-21 23:14:50
tags:
  - sqlite
  - sqlite3
categories:
  - SQLite
---

# 线程安全问题

SQLite3 在编译的时候可以通过添加定义来控制整个库的线程安全问题。

当 `SQLITE_THREADSAFE` 这个宏为 `0` 时，整个库是线程不安全的。

当 `SQLITE_THREADSAFE` 这个宏为 `1` 时，整个库是线程安全的。

当 `SQLITE_THREADSAFE` 这个宏为 `2` 时，整个库是线程安全的，但是同一个 `sqlite3* db` 不能被多个线程同时使用。

当编译的时候 `SQLITE_THREADSAFE` 为 `1` 或者 `2` 时，可以通过 `sqlite3_config()` 这个接口来更改线程安全模式：
- `SQLITE_CONFIG_SINGLETHREAD` 单线程
- `SQLITE_CONFIG_MULTITHREAD` 多线程
- `SQLITE_CONFIG_SERIALIZED` 串行化

# 面向对象类

```cxx
#include <functional>
#include <string>
#include <string_view>
#include <unordered_map>

class SQLite3 {
 public:
  using TableRow = std::unordered_map<std::string, std::string>;
  using ExecCallback = std::function<bool(TableRow)>;

 public:
  SQLite3(const std::string_view& db_name);
  ~SQLite3();

  int close();

  void exec(const std::string_view& sql);
  void exec(const std::string_view& sql, ExecCallback callback);

 private:
  std::string db_name_;
  sqlite3* db_{nullptr};
};
```

我设置了两个成员变量，一个是数据库连接，另一个是打开的数据库名称。

```cxx
std::string db_name_;
sqlite3* db_{nullptr};
```

# 数据库的开启

首先是类的构造函数，主要是打开数据库连接，并在出错时抛出异常信息。

```cxx
SQLite3::SQLite3(const std::string_view& db_name) : db_name_(db_name) {
  auto errcode = sqlite3_open(db_name.data(), &db_);
  if (errcode != SQLITE_OK) {
    // 抛出或处理异常信息
  }
}
```

# 数据库的关闭

然后是析构和关闭数据库连接。

析构函数因为不能抛出异常，所以我使用了 `sqlite3_close_v2` 来关闭连接。

主动 close 则是使用 `sqlite3_close` 接口，并返回错误代码。

```cxx
SQLite3::~SQLite3() {
  if (db_) {
    sqlite3_close_v2(db_);
    db_ = nullptr;
  }
}

int SQLite3::close() {
  auto errcode = sqlite3_close(db_);
  if (errcode == SQLITE_OK) {
    db_ = nullptr;
  }
  return errcode;
}
```

# 执行 SQL 语句

## 无回调函数

无回调函数的操作，只需要执行 SQL 语句，并抛出或者处理执行失败的情况即可。

```cxx
void SQLite3::exec(const std::string_view& sql) {
  char* errstr = nullptr;
  int errcode = sqlite3_exec(db_, sql.data(), nullptr, nullptr, &errstr);

  if (errcode != SQLITE_OK) {
    std::string msg = errstr ? errstr : "Unknown error";
    sqlite3_free(errstr);
    throw std::runtime_error(
        std::format("Failed to execute SQL: {}\nError SQL: {}", msg, sql));
  }
}
```

## 有回调函数

当执行一些带有结果的 SQL 语句时，就需要回调函数去处理结果。

我定义了两个类型用来封装 `sqlite3_exec` 中的回调函数。

`ExecCallback` 对象的返回值为 bool 类型，用来指示回调函数是否成功执行。

并且 `ExecCallback` 对象拥有一个参数 `TableRow`，这是一个哈希表，存储该行的每一个列数据的 name 和 value 的键值对。

```cxx
using TableRow = std::unordered_map<std::string, std::string>;
using ExecCallback = std::function<bool(TableRow)>;
```

我通过 `void* context` 传递 `ExecCallback` 对象。

当回调函数触发时，填充 `TableRow row` 对象的数据，并以此为参数调用 `ExecCallback` 对象。

```cxx
void SQLite3::exec(const std::string_view& sql, ExecCallback callback) {
  auto c_callback = [](void* context, int col_size, char** col_values,
                       char** col_names) -> int {
    auto* cb = static_cast<ExecCallback*>(context);
    TableRow row;
    for (int i = 0; i < col_size; i++) {
      std::string col_name = col_names[i];
      std::string value = col_values[i] ? col_values[i] : "";
      row[col_name] = value;
    }
    bool success = (*cb)(row);
    return success ? 0 : 1;
  };

  char* errstr = nullptr;
  int errcode = sqlite3_exec(db_, sql.data(), c_callback, &callback, &errstr);
  if (errcode != SQLITE_OK) {
    std::string msg = errstr ? errstr : "Unknown error";
    sqlite3_free(errstr);
    throw std::runtime_error(std::format(
        "Failed to execute SQL with callback: {}\nError SQL: {}", msg, sql));
  }
}
```

# 测试一下这个类

```cxx
int main() {
  SQLite3 db("test.db");

  try {
    db.exec("create table if not exists users (id integer primary key autoincrement, name text);");
    db.exec("insert into users (id, name) values (1, 'Alice');");
    db.exec("insert into users (id, name) values (2, 'Bob');");
    db.exec("insert into users (name) values ('Charlie');");
    db.exec("select * from users;", [](SQLite3::TableRow row) {
      std::cout << "User ID: " << row["id"] << ", Name: " << row["name"] << '\n';
      return true;  // Continue iteration
    });
  } catch (const std::exception& e) {
    std::cerr << e.what() << '\n';
  }

  return 0;
}
```

运行上述 main 函数后，获得输出如下：

```
User ID: 1, Name: Alice
User ID: 2, Name: Bob
User ID: 3, Name: Charlie
```

再次运行该代码，输出如下：

```
Failed to execute SQL: UNIQUE constraint failed: users.id
Error SQL: insert into users (id, name) values (1, 'Alice');
```

测试完毕，这样一个初步的面向对象封装就完成了。
