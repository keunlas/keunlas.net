---
title: C++ 协程简述
comments: true
abbrlink: daf63410
date: 2026-03-11 20:03:02
tags:
  - cxx
  - corouting
categories:
  - 开发
  - CXX
---

# promise_type

首先我们需要知道 promise 类型，其写法有一套固定的模板如下。

```cxx
class Task {
 public:
  class promise_type {
   public:
    Task get_return_object() { return {}; }
    std::suspend_never initial_suspend() { return {}; }
    std::suspend_never final_suspend() noexcept { return {}; }
    void unhandled_exception() {}
    void return_void() {}
  };
};
```

# co_await

符合 Awaitable 的类可以对其使用 co_await.





# co_return



# co_yield

