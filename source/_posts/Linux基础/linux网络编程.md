---
title: Linux 网络编程
comments: true
tags:
  - c
  - linux
  - poisx
  - socket
  - network
categories: Linux基础
abbrlink: 18f036ed
date: 2025-02-21 21:15:14
---


**写的比较简略，待完善**



# 大小端转化


```c
#include <arpa/inet.h>

uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);
uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);
```

# 地址转化

```C
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
// 将一个点分十进制的IP地址字符串  -> 转换为网络字节序的32位整型数表示。
in_addr_t inet_addr(const char* cp);
// 将一个点分十进制的IP地址字符串  -> 转换为网络字节序的32位整型数表示。
int inet_aton(const char* cp, struct in_addr* inp);
// 将网络地址  -> 转换为点分十进制IP地址的字符串形式。
char* inet_ntoa(struct in_addr in);
```

# TCP

- socket
- bind
- listen
- connect
- accept
- send & recv
- close


# UDP

- socket
- bind
- sendto & recvfrom
- close
