---
title: 迭代和并发服务器
date: 2015-06-11 15:57:40
tags:
  - 进程
  - linux
categories:
  - 程序设计
---

## 介绍

本文介绍Linux服务端开发中常用的迭代（iterative）模型和并发（concurrent）模型，通过简单的回射服务器程序来说明。在这两种模型下，套接字均使用默认的阻塞（blocking）方式，如果I/O操作不能立即完成，用于读写的系统调用会阻塞。

<!--more-->

## 迭代模型

迭代模型下，服务器每次只能处理一个客户的请求，处理完当前客户的请求之后，才能处理下一个客户的请求。典型的示例代码如下：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h> 

#define PORT_NUM 40713

void error(char *msg) {
    perror(msg);
    exit(EXIT_FAILURE);
}

void do_echo(int fd) {
    char c, buf[100];
    int cnt = 0;
    for (;;) {
        int n = read(fd, &c, 1);
        if (n < 0) {
            error("ERROR read from client");
        } else if (n == 0) {
            break;
        }
        buf[cnt++] = c;
        if (c == '\n') {
            if (write(fd, buf, cnt) < 0) {
                error("ERROR write to client");
            }
            cnt = 0;
            continue;
        }
    }
}

int main(int argc, char **argv) {
    int lfd;
    struct sockaddr_in server_addr;
    struct sockaddr_in client_addr;

    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    server_addr.sin_port = htons(PORT_NUM);

    if ((lfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        error("ERROR socket");
    }
    if (bind(lfd, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        error("ERROR bind");
    }
    if (listen(lfd, 16) < 0) {
        error("ERROR listen");
    }

    for (;;) {
        socklen_t len = sizeof(client_addr);
        int cfd = accept(lfd, (struct sockaddr *)&client_addr, &len);
        if (cfd < 0) {
            error("ERROR accept");
        }
        do_echo(cfd);
    }

    close(lfd);
    exit(EXIT_SUCCESS);
}
```

对于简单的业务来说，迭代服务器够用了。然而当服务一个客户请求需要时间较长时，我们不希望整个服务器被单个客户长期占用，希望同时服务多个客户，这种情况下可以使用并发模型。

## 并发模型

并发模型下，服务器可以同时处理多个客户的请求。对每一个客户，服务器都派生一个子进程（线程），子进程（线程）通过已连接套接字服务客户，I/O如果阻塞，只能阻塞在子进程（线程）内。而主进程（线程）则通过监听套接字等待新的连接。相比多线程，多进程代码较为简洁，因此本文使用多进程来实现。典型的示例代码如下：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <arpa/inet.h> 

#define PORT_NUM 40713

void error(char *msg) {
    perror(msg);
    exit(EXIT_FAILURE);
}

void do_echo(int fd) {
    char c, buf[100];
    int cnt = 0;
    for (;;) {
        int n = read(fd, &c, 1);
        if (n < 0) {
            error("ERROR read from client");
        } else if (n == 0) {
            break;
        }
        buf[cnt++] = c;
        if (c == '\n') {
            if (write(fd, buf, cnt) < 0) {
                error("ERROR write to client");
            }
            cnt = 0;
            continue;
        }
    }
}

int main(int argc, char **argv) {
    int lfd;
    struct sockaddr_in server_addr;
    struct sockaddr_in client_addr;

    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    server_addr.sin_port = htons(PORT_NUM);

    if ((lfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        error("ERROR socket");
    }
    if (bind(lfd, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        error("ERROR bind");
    }
    if (listen(lfd, 16) < 0) {
        error("ERROR listen");
    }

    for (;;) {
        socklen_t len = sizeof(client_addr);
        int cfd = accept(lfd, (struct sockaddr *)&client_addr, &len);
        if (cfd < 0) {
            error("ERROR accept");
        }

        switch(fork()) {
        case -1:
            perror("fork");
            close(cfd);
            break;
        case 0:
            close(lfd);
            do_echo(cfd);
            _exit(EXIT_SUCCESS);
        default:
            close(cfd);
            while (wait(NULL) != -1) {
                continue;
            }
            if (errno != ECHILD) {
                error("ERROR wait");
            }
        }
    }

    close(lfd);
    exit(EXIT_SUCCESS);
}
```

上述代码中，子进程服务完客户后马上退出，而频繁地创建和销毁子进程会耗费较多的资源。可以改进的是，服务器可以在启动时预先派生（prefork）子进程，创建一个子进程池，每个客户请求由当前子进程池中的某个闲置子进程处理，任务处理完成后，不立即退出，而是继续等待服务新的客户，这就是所谓的进程池（process pool）技术。对应地，当使用多线程实现并发服务器时，可以使用线程池（thread pool）技术以提高性能。
