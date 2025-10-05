---
title: 《SDL3 教程》第一个窗口（01）
comments: true
abbrlink: 1d11277a
date: 2025-10-06 05:45:51
tags:
  - sdl
  - sdl3
categories:
  - SDL3
---

# 渲染第一个 SDL 窗口

首先我们从一个简单的例子讲起，我们渲染一个绿色背景的窗口。

基本步骤就是创建窗口和渲染器，然后在循环中不断的渲染窗口并展示。

`SDL_Init`，`SDL_CreateWindow` 和 `SDL_CreateRenderer` 拥有 bool 类型的返回值，下列代码中省略了返回值判断。

```cxx
#include <SDL3/SDL.h>

int main(int argc, char* argv[]) {
  SDL_Init(SDL_INIT_VIDEO); // 初始化 SDL
  SDL_Window* window = SDL_CreateWindow("Hello World", 800, 600, 0);  // 创建窗口
  SDL_Renderer* renderer = SDL_CreateRenderer(window, nullptr); // 创建渲染器

  SDL_Event e;
  bool quit = false;
  while (!quit) {
    while (SDL_PollEvent(&e)) {
      if (e.type == SDL_EVENT_QUIT) {
        quit = true;  // 当获取到 QUIT 事件时， 终止循环
      }
    }

    SDL_SetRenderDrawColor(renderer, 0, 255, 0, 255); // 渲染器的颜色
    SDL_RenderClear(renderer);  // 清空窗口为渲染器的颜色
    SDL_RenderPresent(renderer);  // 展示渲染好的窗口
  }

  // 清理资源
  SDL_DestroyRenderer(renderer);
  SDL_DestroyWindow(window);
  SDL_Quit();
  return 0;
}
```

# 更加的模块化 SDL3

SDL3 中只需要 `#define SDL_MAIN_USE_CALLBACKS 1` 并 `#include <SDL3/SDL_main.h>` 即可使用回调函数代替 main 函数，从而更加方便的编写程序。

最关键的四个回调函数为：

- 程序启动时执行：`SDL_AppResult SDL_AppInit(void **appstate, int argc, char *argv[]);`
- 有新事件时执行：`SDL_AppResult SDL_AppEvent(void *appstate, SDL_Event *event);`
- 程序每帧都执行：`SDL_AppResult SDL_AppIterate(void *appstate);`
- 程序退出时执行：`void SDL_AppQuit(void *appstate, SDL_AppResult result);`

具体代码结构如下：

```cxx
#define SDL_MAIN_USE_CALLBACKS 1 // 必须设置这个宏
#include <SDL3/SDL.h>
#include <SDL3/SDL_main.h> // 并且引入这个头文件
#include <iostream>

static SDL_Window *window = nullptr;
static SDL_Renderer *renderer = nullptr;

// 当程序启动时执行 SDL_AppInit 函数
SDL_AppResult SDL_AppInit(void **appstate, int argc, char *argv[]) {
  SDL_Init(SDL_INIT_VIDEO);
  window = SDL_CreateWindow("Hello World", 800, 600, 0);
  renderer = SDL_CreateRenderer(window, nullptr);

  return SDL_APP_CONTINUE;
}

// 当新的事件发生时触发 SDL_AppEvent 函数
SDL_AppResult SDL_AppEvent(void *appstate, SDL_Event *event) {
  if (event->type == SDL_EVENT_QUIT) {
    return SDL_APP_SUCCESS;
  }

  return SDL_APP_CONTINUE;
}

// 每帧运行一次 SDL_AppIterate 函数
SDL_AppResult SDL_AppIterate(void *appstate) {
  SDL_SetRenderDrawColor(renderer, 0, 255, 0, 255);
  SDL_RenderClear(renderer);
  SDL_RenderPresent(renderer);

  return SDL_APP_CONTINUE;
}

// 当程序退出时执行 SDL_AppQuit 函数
void SDL_AppQuit(void *appstate, SDL_AppResult result) {
  std::cerr << "SDL_AppQuit." << std::endl;
}
```

