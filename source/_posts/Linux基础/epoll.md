---
title: I/O 多路复用 epoll
comments: true
tags:
  - c
  - linux
  - poisx
  - epoll
categories: Linux基础
date: 2025-02-21 21:25:14
---

**写的比较简略，待完善**


# epoll_create

创建epoll对象: (man 2 epoll_create)

```C
#include <sys/epoll.h>
// open an epoll file descriptor
int epoll_create(
    	int size	// 历史遗留参数, 已无任何意义, 大于0即可
);
// 返回值: 成功返回epoll文件对象的文件描述符,  失败返回-1
```

# epoll_ctl

调整监听事件集合: (man 2 epoll_ctl )

```C
#include <sys/epoll.h>
// control interface for an epoll file descriptor
int epoll_ctl(
        int epfd,	// epoll的文件描述符
        int op,		// 操作类型:EPOLL_CTL_ADD、EPOLL_CTL_MOD、EPOLL_CTL_DEL, 分别表示添加、修改和删除事件
        int fd,		// 要被监听对应操作的文件描述符
        struct epoll_event *event	// 指定对监听的文件描述符的监听事件。只有在添加、修改文件描述符时，这个参数才是必需的；在删除操作时，通常设置为NULL
);
// 返回值: 
```

```C
// (man 2 epoll_ctl )

struct epoll_event {
        uint32_t     events;	// 事件:  EPOLLIN/读取操作、EPOLLOUT/写入操作、...、EPOLLET/边缘触发模式 (其他不重要)
        epoll_data_t data;		// 上述事件的对应的文件描述符
};
```

```C
// (man 2 epoll_ctl )

typedef union epoll_data {
        void        *ptr;
        int          fd;	// 文件描述符
        uint32_t     u32;
        uint64_t     u64;
} epoll_data_t;
```

# epoll_wait

进入阻塞状态，直到监听的设备就绪或者超时:  ( man 2 epoll_wait  )

```C
#include <sys/epoll.h>
// wait for an I/O event on an epoll file descriptor
int epoll_wait(
        int epfd,	// epoll的文件描述符
        struct epoll_event *events,	// 用于接收就绪集合的数组
        int maxevents,				// 最大就绪集合长度
        int timeout	// 超时时间(毫秒), -1则一直等待
);
// 返回值: 成功返回就绪个数, 失败返回-1
```
