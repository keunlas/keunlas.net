---
title: C++ 中的异常处理
comments: true
tags:
  - cpp
  - exceptions
  - throw
categories: C/C++基础
abbrlink: 401f63bd
date: 2025-02-10 14:56:04
---

异常处理是一个很方便的东西, 在很多时候都可以非常方便的去处理一些异常.

# 抛出异常

```cpp
throw <expression>
```

异常是一个表达式, 它的值可以是基本类型, 也可以是类.

```C++
double division(double x, double y)
{
	if(y == 0) {
		throw "Division by zero condition!";
	}
	return x / y;
}
```

# 捕获异常

try-catch语句块的catch可以有多个, 至少要有一个, 否则会报错.

``` c++
double x = 100, y = 0;
try{
	cout << division(x,y) << endl;
}catch(const char *msg){
	cout << "const char *: " << msg << endl;
}catch(double x){
	cout << "double: " << x << endl;
}catch(int x){
	cout << "int: " << x << endl;
}
```









