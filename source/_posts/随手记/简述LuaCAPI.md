---
title: 简述 Lua CAPI
tags:
  - c
  - cpp
  - lua
categories: 随手记
comments: true
abbrlink: 6429c750
date: 2025-02-08 18:08:57
---


# 写一个简单的Lua解释器

从这个简单的解释器, 我们可以初步的了解到如何在`C/C++`中使用Lua.

```cpp
#include <iostream>
#include <lua.hpp>
#include <string>
using namespace std;

int main() {
  // 获取一个lua_State栈
  lua_State *L = luaL_newstate();
  // 打开所有标准库(不打开的话, 后面可能无法执行输入的lua代码)
  luaL_openlibs(L);

  cout << "欢迎使用 Lua解释器•伪" << endl;

  string line; // 接受输入的lua代码
  while (getline(cin, line)) {

    // 执行lua代码, 无错误则返回0.
    // luaL_dostring(L, line.c_str()) 是一个宏, 相当于下面这行代码
    // luaL_loadstring(L, line.c_str()) || lua_pcall(L, 0, 0, 0)
    int ret = luaL_dostring(L, line.c_str());

		// 返回不为0, 则打印错误信息.
    if (ret) { 
			// 下一部分会讨论lua_State虚拟栈
			// 错误信息会入栈, -1代表栈顶, 1代表栈底
			// 将栈顶元素转为c风格字符串
      cout << lua_tostring(L, -1) << endl;
			// 把错误信息弹栈, 这里的1是出栈元素数量
      lua_pop(L, 1); 
    }
  }

  lua_close(L);
  return 0;
}
```

# Lua_State虚拟栈

lua和c之间依靠一个虚拟栈来进行数据的通信. 我们可以像操作栈一样, 进行数据交互.

**压栈函数**
- lua_pushnil
- lua_pushboolean
- lua_pushnumber
- lua_pushinteger
- lua_pushlstring
- lua_pushstring

**检查栈空间, 可能的话这两个函数将尝试扩大空间**
- lua_checkstack
- luaL_checkstack

**检查栈中的一个元素是否为特定类型**
- lua_is*
- lua_isnumber
- lua_istable
- lua_isstring
- ...


**通用栈操作函数**

- `lua_gettop`：返回栈中元素的个数，即栈顶元素的索引
- `lua_settop`：将栈顶设置为一个指定的值，即修改栈中元素的数量。`lua_settop(L, 0)`用于清空栈
- `lua_pushvalue`：用于将指定索引上的元素的副本压入栈
- `lua_rotate`：将指定索引到栈顶位置之间的元素旋转 n 个位置。n 为正数，表示将元素向栈顶方向移动，n 为负数则表示向相反方向移动
- `lua_remove`：删除指定索引的元素，并将该位置之上的所有元素向下移动以填补空缺
- `lua_insert`：用于将栈顶元素移动到指定位置，并上移指定位置之上的所有元素以开辟出一个元素的空间
- `lua_replace`：将栈顶元素移动到指定索引上（并且将这个栈顶元素弹出），不移动任何元素
- `lua_copy`：将一个索引上的值复制到另一个索引上，原值不受影响


**例子**
```cpp
#include <iostream>
#include <lua.hpp>
#include <string>
using namespace std;

static void stack_dump(lua_State *L) {
  int top = lua_gettop(L);

  for (int i = 1; i <= top; ++i) {
    int t = lua_type(L, i);
    switch (t) {
    case LUA_TSTRING: {
      cout << lua_tostring(L, i);
      break;
    }
    case LUA_TBOOLEAN: {
      cout << (lua_toboolean(L, i) ? "true" : "false");
      break;
    }
    case LUA_TNUMBER: {
      cout << lua_tonumber(L, i);
      break;
    }
    default: {
      cout << lua_typename(L, t);
      break;
    }
    }
    cout << "  ";
  }

  cout << endl;
}

int main(void) {
  lua_State *L = luaL_newstate();

  lua_pushboolean(L, 1);
  lua_pushnumber(L, 10);
  lua_pushnil(L);
  lua_pushstring(L, "hello");
  cout << "origin data:" << endl;
  stack_dump(L);

  lua_pushvalue(L, -4);
  cout << "lua_pushvalue(L, -4):" << endl;
  stack_dump(L);

  lua_replace(L, 3);
  cout << "lua_replace(L, 3):" << endl;
  stack_dump(L);

  lua_settop(L, 6);
  cout << "lua_settop(L, 6):" << endl;
  stack_dump(L);

  lua_rotate(L, 3, 1);
  cout << "lua_rotate(L, 3, 1):" << endl;
  stack_dump(L);

  lua_remove(L, -3);
  cout << "lua_remove(L, -3):" << endl;
  stack_dump(L);

  lua_settop(L, -5);
  cout << "lua_settop(L, -5):" << endl;
  stack_dump(L);

  lua_close(L);
  return 0;
}
```
```
// output
origin data:
true  10  nil  hello  
lua_pushvalue(L, -4):
true  10  nil  hello  true  
lua_replace(L, 3):
true  10  true  hello  
lua_settop(L, 6):
true  10  true  hello  nil  nil  
lua_rotate(L, 3, 1):
true  10  nil  true  hello  nil  
lua_remove(L, -3):
true  10  nil  hello  nil  
lua_settop(L, -5):
true
```

# 操作 Lua 表

Lua C API 提供了一系列函数用于操作表, 以下简略的说几个.


**创建表**

- **`void lua_newtable (lua_State *L);`**  
  创建一个空表并压入栈顶。

- **`void lua_createtable (lua_State *L, int narr, int nrec);`**  
  创建预分配大小的表：  
  - `narr`: 数组部分的初始容量  
  - `nrec`: 哈希部分的初始容量  


**设置/获取键值对**

**通用方法**

- **`void lua_gettable (lua_State *L, int index);`**  
  获取表中键对应的值：  
  - 操作的表位于 `index`  
  - 弹出栈顶的键，将结果值压入栈顶。

- **`void lua_settable (lua_State *L, int index);`**  
  设置表中键值对：  
  - 操作的表位于 `index`  
  - 弹出栈顶的值和键，设置到表中。

**简化方法**

- **`void lua_getfield (lua_State *L, int index, const char *k);`**  
  获取键 `k` 对应的值，结果压入栈顶。

- **`void lua_setfield (lua_State *L, int index, const char *k);`**  
  将栈顶值弹出，并设置为表 `index` 中键 `k` 的值。


# 调用 Lua 函数

调用 Lua 函数的 API 规范很简单，首先将待调用的函数压栈，然后压入函数的参数，接着调用 lua_pcall 进行实际的调用，最后从栈中取出结果。


**假设 Lua 配置脚本中有如下函数：**
```lua
function f(x, y)
    return (x^2 * math.sin(y)) / (1 - x)
end
```

**从 C 语言中调用 Lua 函数 f 的函数：**
```c
double f(lua_State *L, double x, double y) {
    int isnum;
    double z;

    lua_getglobal(L, "f"); // 函数 f 入栈
    lua_pushnumber(L, x);  // 入栈第一个参数
    lua_pushnumber(L, y);  // 入栈第二个参数

		// 调用函数会将函数本身和参数全部出栈
		// lua_pcall的4个参数分别为 lua_State *L, int nargs, int nresults, int errfunc
		// 2 是参数数量, 1 是返回值数量, 最后一个参数一般填写为 0
    if (lua_pcall(L, 2, 1, 0) != LUA_OK) {
        error(L, "error running function 'f': %s", lua_tostring(L, -1));
    }

		// 检查返回时是否是数字
    z = lua_tonumberx(L, -1, &isnum);
    if (!isnum) {
        error(L, "function 'f' should return a number");
    }

    lua_pop(L, 1);
    return z;
}
```