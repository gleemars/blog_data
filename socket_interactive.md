---
title: socket
tags: 
categorires: 
author: 
top: 
---

## sockaddr_in
```c
struct sockaddr_in{
	sa_family_t sin_family;            // short int 地址类型: ipv6...  
	uint16_t sin_port;                    // 16 位端口号, htons(port_num)
	struct in_addr sin_addr;          // 32bits  ip
	char sin_zero[n];            	
};

struct sockaddr_in6 { 
    sa_family_t sin6_family;  //(2)地址类型，取值为AF_INET6
    in_port_t sin6_port;  //(2)16位端口号
    uint32_t sin6_flowinfo;  //(4)IPv6流信息
    struct in6_addr sin6_addr;  //(4)具体的IPv6地址
    uint32_t sin6_scope_id;  //(4)接口范围ID
};

struct sockaddr
  {
    __SOCKADDR_COMMON (sa_);	/* Common data: address family and length.  */
    char sa_data[14];		/* Address data.  */
  };
```

## socket domain
```C
AF_INET 2 // IPv4
AF_INET 26 // IPv6
```

## socket(int domain, int type, int protocol)
> int domain: 协议簇, socket使用ipv4还是ipv6等等
> int type: socket类型, 并非指使用ipv4还是ipv6
> int protocol: 协议, tcp、udp等等

## int bind(int sock_fd, struct sockaddr \*addr, socklen_t addrlen) 
> 把socket绑定到某个ip与端口中, ip和对应端口已经储存在 sockaddr 这个结构体中

## lisetn(int sock, int backlog)
> 让sock进入监听状态
> backlog表示请求队列的最大长度


## send

## recv




