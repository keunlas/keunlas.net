---
title: POSIX线程
comments: true
tags:
  - c
  - linux
  - thread
  - pthread
  - posix
categories: Linux基础
abbrlink: b581fc6f
date: 2025-02-07 22:24:26
---




以下是一份详细的 `pthread` 使用教程，涵盖基本概念、API 接口、线程同步及示例代码。

---

## 一、POSIX 线程（pthread）简介
POSIX 线程（`pthread`）是 UNIX/Linux 系统下的多线程编程标准接口，允许程序在单个进程中创建和管理多个线程，共享内存空间和资源。

---

## 二、环境配置
### 1. 头文件
```c
#include <pthread.h>
```

### 2. 编译选项
在编译时需添加 `-pthread` 选项：
```bash
gcc program.c -o program -pthread
```

---

## 三、核心 API 接口

### 1. 线程创建
```c
int pthread_create(
    pthread_t *thread,         // 线程标识符指针
    const pthread_attr_t *attr,// 线程属性（NULL 为默认）
    void *(*start_routine)(void *), // 线程执行的函数
    void *arg                  // 传递给函数的参数
);
```

#### 示例：创建线程
```c
#include <stdio.h>
#include <pthread.h>

void* print_message(void *msg) {
    printf("Message: %s\n", (char*)msg);
    return NULL;
}

int main() {
    pthread_t thread;
    char *message = "Hello from thread!";
    pthread_create(&thread, NULL, print_message, message);
    pthread_join(thread, NULL); // 等待线程结束
    return 0;
}
```

---

### 2. 线程终止
- **线程主动退出**：使用 `pthread_exit`
  ```c
  void pthread_exit(void *retval);
  ```

- **主线程等待子线程结束**：
  ```c
  int pthread_join(pthread_t thread, void **retval);
  ```

#### 示例：等待线程结束
```c
void* task(void *arg) {
    int *num = (int*)arg;
    printf("Thread processing: %d\n", *num);
    pthread_exit(NULL);
}

int main() {
    pthread_t tid;
    int value = 42;
    pthread_create(&tid, NULL, task, &value);
    pthread_join(tid, NULL);
    printf("Main thread exits.\n");
    return 0;
}
```

---

### 3. 线程同步

#### (1) 互斥锁（Mutex）
- **初始化/销毁互斥锁**：
  ```c
  pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER; // 静态初始化
  // 或动态初始化：
  pthread_mutex_init(&mutex, NULL);
  pthread_mutex_destroy(&mutex);
  ```

- **加锁/解锁**：
  ```c
  pthread_mutex_lock(&mutex);   // 阻塞等待锁
  pthread_mutex_unlock(&mutex); // 释放锁
  ```

#### 示例：互斥锁保护共享资源
```c
int counter = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void* increment(void *arg) {
    for (int i = 0; i < 100000; i++) {
        pthread_mutex_lock(&mutex);
        counter++;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;
    pthread_create(&t1, NULL, increment, NULL);
    pthread_create(&t2, NULL, increment, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    printf("Final counter: %d\n", counter); // 应为 200000
    return 0;
}
```

---

#### (2) 条件变量（Condition Variables）
- **初始化/销毁条件变量**：
  ```c
  pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
  pthread_cond_init(&cond, NULL);
  pthread_cond_destroy(&cond);
  ```

- **等待条件**：
  ```c
  pthread_cond_wait(&cond, &mutex); // 自动释放锁并等待
  ```

- **通知条件**：
  ```c
  pthread_cond_signal(&cond);  // 唤醒一个等待线程
  pthread_cond_broadcast(&cond); // 唤醒所有等待线程
  ```

#### 示例：生产者-消费者模型
```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
int data_ready = 0;

void* producer(void *arg) {
    pthread_mutex_lock(&mutex);
    data_ready = 1;
    printf("Produced data.\n");
    pthread_cond_signal(&cond);
    pthread_mutex_unlock(&mutex);
    return NULL;
}

void* consumer(void *arg) {
    pthread_mutex_lock(&mutex);
    while (!data_ready) {
        pthread_cond_wait(&cond, &mutex);
    }
    printf("Consumed data.\n");
    pthread_mutex_unlock(&mutex);
    return NULL;
}

int main() {
    pthread_t prod, cons;
    pthread_create(&cons, NULL, consumer, NULL);
    sleep(1); // 确保消费者先等待
    pthread_create(&prod, NULL, producer, NULL);
    pthread_join(prod, NULL);
    pthread_join(cons, NULL);
    return 0;
}
```

---

### 4. 线程属性设置
通过 `pthread_attr_t` 自定义线程属性：
```c
pthread_attr_t attr;
pthread_attr_init(&attr);
pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED); // 设置为分离线程
pthread_create(&thread, &attr, start_routine, arg);
pthread_attr_destroy(&attr);
```

---












`pthread_cancel` 和 `pthread_testcancel` 是 POSIX 线程库中用于线程取消的接口。它们允许一个线程请求取消另一个线程，并在目标线程中设置取消点以响应取消请求。以下是它们的详细用法和示例。

---

### 5. 进程被动终止



**`pthread_cancel`函数原型**
```c
int pthread_cancel(pthread_t thread);
```

**功能**
向指定的线程发送取消请求。目标线程是否被取消取决于其取消状态和取消类型。

**参数**
- `thread`：目标线程的标识符（`pthread_t` 类型）。

**返回值**
- 成功返回 `0`，失败返回错误码。

---

**`pthread_testcancel`函数原型**
```c
void pthread_testcancel(void);
```

**功能**
在调用线程中设置一个取消点。如果线程收到取消请求且取消状态为启用，则线程会在调用此函数时终止。

**使用场景**
- 在长时间运行的循环或阻塞操作中，手动设置取消点以响应取消请求。


---





### 6. 资源清理


当线程开始执行时，会创建一个清理栈（thread's stack of  thread-cancellation），用于存储注册的清理函数和参数。

而我们可以通过 `pthread_cleanup_push` 和 `pthread_cleanup_pop `通过向这个清理栈中添加清理函数，以及pop出清理函数执行。从而确保线程的正确终止和资源回收。 

```c
#include <pthread.h>
// push thread cancellation clean-up handlers
void pthread_cleanup_push( 
	void (*routine)(void *),// 指向清理函数的指针，该函数将在线程取消时被调用
	void *arg // 传递给清理函数的参数
);


#include <pthread.h>
// pop thread cancellation clean-up handlers
void pthread_cleanup_pop( 
	int execute // 设置参数0代表弹出栈顶函数并且'不执行'这个函数， 非0代表代表弹出栈顶函数并且'执行'这个函数
);
```


- pthread_cleanup_push()和pthread_cleanup_pop()必须在同一个作用域中要成对出现。（此处可参考: /usr/include/pthread.h 对这两个方法的定义 -> 可以发现push和pop的宏定义不是语义完全的，它们必须在同一作用域中成对出现才能使花括号成功匹配。）
- 线程运行到pthread_cleanup_pop()方法,当execute参数0代表弹出栈顶函数并且'不执行'这个函数，当execute参数非0代表代表弹出栈顶函数并且'执行'这个函数。
- 通过pthread_cancel取消线程， 线程取消，所有入栈的清理函数将按照顺序依次弹栈并执行。
- 调用pthread_exit主动退出线程， 所有入栈的清理函数将按照顺序依次弹栈并执行。
- 当线程因为在start_routine/入口函数中因为return结束线程， 清理函数栈将不会弹栈。（这取决于操作系统）



例子：

```c
#include <testfun.h>

void cleanheap(void *argp){
   printf("cleanheap is runing \n");
   free(argp);
}
void cleanfile(void *filep){
   printf("cleanfile is runing \n");
   int *fdP = (int *)filep;
   close(*fdP);
}
void * func(void *arg){
   void *p1 = malloc(4);
   pthread_cleanup_push(cleanheap, p1);
   void *p2 = malloc(4);
   pthread_cleanup_push(cleanheap, p2);

   int fd = open("file1", O_RDWR); // ps： 注意，代码未必能执行到这个位置
   pthread_cleanup_push(cleanfile, &fd);

   // sleep(2);

   pthread_cleanup_pop(1);
   pthread_cleanup_pop(1);
   pthread_cleanup_pop(1);
}
int main()
{
   pthread_t  pid;
   int res = pthread_create(&pid, NULL, func, NULL);
   THREAD_ERROR_CHECK(res, "pthread_create");

   // sleep(1);
   int res_cancel = pthread_cancel(pid);
   THREAD_ERROR_CHECK(res_cancel, "pthread_cancel");

   int res_join = pthread_join(pid, NULL);
   THREAD_ERROR_CHECK(res_join, "pthread_join");

   return 0;
}
```






---

## 四、注意事项
1. **资源竞争**：始终使用互斥锁保护共享资源。
2. **死锁**：避免多个锁的嵌套使用，或按固定顺序加锁。
3. **线程安全函数**：确保调用的库函数是线程安全的（如 `rand_r` 替代 `rand`）。
4. **内存泄漏**：分离线程（detached）无法被 `pthread_join`，需自行释放资源。

