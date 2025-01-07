---
title: C语言中的枚举类型
comments: true
date: 2025-01-07 10:43:24
tags:
  - C
categories: C/C++
---

在编程中, 我们可能会碰到一个场景, 需要对一系列的状态和离散的数值进行逻辑处理. 

比如在一个购物程序中, 需要根据订单的不同状态执行相应的逻辑. 这些状态可能包括: 新建订单(NEW), 已支付(PAID), 已发货(SHIPPED), 已送达(DELIVERED), 已完成(COMPLETED), 已取消(CANCELLED)等. 

使用整数或宏定义来表示这些状态是一种解决方法, 但是**使用枚举可以让代码更清晰**. 并且在调试时能够看到状态的实际名称, 而不仅仅是一个数字. 

使用枚举的好处: 
- 增强代码可读性
- 增强代码的可维护性

<!--more-->

# 枚举类型的声明与初始化


```c
// 定义一个名为 order_status 的枚举类型，表示订单的不同状态
enum order_status {
    NEW,        // 新订单
    PAID,       // 已支付
    SHIPPED,    // 已发货
    DELIVERED,  // 已送达
    COMPLETED,  // 已完成
    CANCELLED   // 已取消  
};

int main(void) {
    // 声明一个枚举类型变量
    enum order_status a_status;
    // 初始化一个枚举类型变量
    enum order_status b_status = NEW;
    return 0;
}
```

# typedef也可以为枚举类型取别名

```c
// 定义一个别名为 OrderStatus 的枚举类型，表示订单的不同状态
typedef enum {
    NEW,        // 新订单
    PAID,       // 已支付
    SHIPPED,    // 已发货
    DELIVERED,  // 已送达
    COMPLETED,   // 已完成
    CANCELLED   // 已取消  
} OrderStatus;

int main(void) {
    // 声明一个枚举类型变量，此时可以省略enum关键字
    OrderStatus status;
    return 0;
}
```

# 枚举类型的成员本质上就是一个整数

```c
typedef enum {
    NEW = 888, // 默认值为0, 也可以手动赋值, 但是没有必要这么做
    PAID,      // 自动累加 889    
    SHIPPED,   // 890
    DELIVERED,  
    COMPLETED,  
    CANCELLED   
} OrderStatus;
```

# 使用枚举的注意事项

- 枚举类型的成员会被编译器当成整型(一般是int)处理, 这意味着C语言的***枚举类型不是类型安全的***.
- 可以将任何整数赋值给枚举类型变量, 甚至不同枚举类型变量之间都可以相互赋值.
- 将枚举类型作为函数的形参, 实际上还是可以传参整数值.

为了避免这些潜在的问题, 使用枚举类型枚举类型变量的赋值, 应该使用枚举类型中定义的成员, 不要使用整数进行赋值, 更不应该用其它枚举类型进行赋值.

C语言的枚举类型设计是十分简单和功能弱小的, 但在特定的场景中, 它也是足够用的.