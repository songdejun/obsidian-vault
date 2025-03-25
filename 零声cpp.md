[零声教育 第14代 Linux C/C++开发（后端/音视频/游戏/嵌入式/高性能网络/存储/基础架构/安全）成长体系课程 - v1.8](https://0voice.com/uiwebsite/html/courses/v14.8.html)
# 一、网络
## （一）网络io与io多路复用
### 1. 基础写法
为每一个连接单独开一个线程写法
```c
#include <stdio.h>
// sockaddr socket() bind() accept() recv()
#include <sys/socket.h>
// perror
#include <errno.h>
// sockaddr_in
#include <netinet/in.h>
// memset()
#include <string.h>
// close() 来自systemcall
#include <unistd.h>
// 处理多个客人，pthread_t
#include <pthread.h>

void *client_thread(void *arg) {
    
    int clientfd = *(int *)arg;

    while (1) {
        // 后厨收到了顾客要的菜
        char buffer[128] = {0};
        int count = recv(clientfd, buffer, 128, 0);
        // count = 0说明客户端调用了close()
        if (count == 0) {
            break;
        }
        // 发给客户的菜
        buffer[0] = 'p';
        send(clientfd, buffer, count, 0);
        printf("clientfd = %d, count = %d, buffer = %s\n",clientfd, count, buffer);
    }

    close(clientfd);
    return NULL;
}

int main() {
    // 套接字，类比酒店的服务员
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);

    // 类比酒店的饭桌
    struct sockaddr_in serveraddr;
    memset(&serveraddr, 0, sizeof(struct sockaddr_in));

    // sin = socket internet
    // s = socket
    serveraddr.sin_family = AF_INET;
    // htons = host to networt short，转化到网络字节序（大端）
    // INADDR_ANY = 0.0.0.0，任意ip都接受
    serveraddr.sin_addr.s_addr = htons(INADDR_ANY);
    serveraddr.sin_port = htons(2048);

    // 类比分配服务员到哪片区域
    // struct sockaddr和sockaddr_in是一样的，至于为什么不用一个是历史遗留原因
    if (-1 == bind(sockfd, (struct sockaddr*)&serveraddr, sizeof(struct sockaddr))) {
        perror("bind");
        return -1;
    }

    // 服务员准备开始上班
    // 10代表socket队列的最大数量，一般5 10 20
    listen(sockfd, 10);

    printf("accept\n");

    // 准备接受多个客人请求
    while (1) {
        // 接收到一个客人的请求
        struct sockaddr_in clientaddr;
        socklen_t len = sizeof(struct sockaddr);
        // 获得客人的名字
        int clientfd = accept(sockfd, (struct sockaddr*)&clientaddr, &len);
		
        // 每个客人找一个后厨给他处理
        pthread_t thid;
        pthread_create(&thid, NULL, client_thread, &clientfd);
    }

    getchar();

    return 0;
}
```

### 2. select 多路复用
用`select()`实现IO多路复用写法

```c
#include <stdio.h>
// sockaddr socket() bind() accept() recv()
#include <sys/socket.h>
// perror
#include <errno.h>
// sockaddr_in
#include <netinet/in.h>
// memset()
#include <string.h>
// close() 来自systemcall
#include <unistd.h>
#include <sys/select.h>

int main() {
    // 套接字，类比酒店的服务员
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);

    // 类比酒店的饭桌
    struct sockaddr_in serveraddr;
    memset(&serveraddr, 0, sizeof(struct sockaddr_in));

    // sin = socket internet
    // s = socket
    serveraddr.sin_family = AF_INET;
    // htons = host to networt short，转化到网络字节序（大端）
    // INADDR_ANY = 0.0.0.0，任意ip都接受
    serveraddr.sin_addr.s_addr = htons(INADDR_ANY);
    serveraddr.sin_port = htons(2048);

    // 类比分配服务员到哪片区域
    // struct sockaddr和sockaddr_in是一样的，至于为什么不用一个是历史遗留原因
    if (-1 == bind(sockfd, (struct sockaddr*)&serveraddr, sizeof(struct sockaddr))) {
        perror("bind");
        return -1;
    }

    // 服务员准备开始上班
    listen(sockfd, 10);

    printf("accept\n");

    fd_set rfds, rset;
    FD_ZERO(&rfds);
    FD_SET(sockfd, &rfds);

    int maxfd = sockfd;

    while (1) {
        rset = rfds;
        int nready = select(maxfd+1, &rset, NULL, NULL, NULL);

        if (FD_ISSET(sockfd, &rset)) {
            struct sockaddr_in clientaddr;
            memset(&clientaddr, 0, sizeof(clientaddr));
            socklen_t len = sizeof(clientaddr);
            int clientfd = accept(sockfd, (struct sockaddr*)&clientaddr, &len);

            printf("sockfd = %d\n", clientfd);

            FD_SET(clientfd, &rfds);
            maxfd = clientfd;
        }

        for (int i = sockfd + 1; i <= maxfd; i ++) {
            if (FD_ISSET(i, &rfds)) {
                char buffer[128] = {0};
                int count = recv(i, buffer, 128, 0);
                if (count == 0) {
                    printf("disconnect\n");
                    FD_CLR(i, &rfds);
                    close(i);
                    break;
                }
                send(i, buffer, count, 0);
                printf("sockfd = %d, buffer = %s\n", i, buffer);
            }
        }
    }

    getchar();

    return 0;
}
```

**使用`select()`时为什么要设置`rfds`和`rset`两个`FD_SET`？**

在使用`select()`进行多路复用时，通常会设置两个`FD_SET`：`rfds`和`rset`。通过介绍`select()`函数的特点以及它们的作用和区别，解释为什么需要设置两个`FD_SET`，而不是只设置一个。

#### 1. `select()`函数的特点

- **破坏性**：`select()`函数只会保留准备好的（如可读）文件描述符，其余的文件描述符会被清除。

#### 2. `rfds`的作用

- **管理**：`rfds`用于管理全局逻辑上的监听套接字和已连接的客户端套接字。
- **动态更新**：当产生新的连接或连接断开时，动态更新`rfds`。

#### 3. `rset`的作用

- **视图**：`rset`作为`rfds`的一个视图，在不破坏`rfds`正确性的前提下，使用`select()`函数挑选出需要被处理的套接字。


#### 总结

为进程的套接字`rfds`创建一个视图`rset`，使得在使用`select()`函数时不会破坏`rfds`的正确性。

#### 举例说明

假设 `rfds` 中包含了以下文件描述符：

- `sockfd`（监听套接字）
- `clientfd1`（客户端 1）
- `clientfd2`（客户端 2）

如果直接传递 `rfds` 给 `select()`，而只有 `clientfd1` 有数据可读，那么 `select()` 返回后，`rfds` 中只会剩下 `clientfd1`，`sockfd` 和 `clientfd2` 会被清除。

这样会导致以下问题：

- 下一次循环时，`rfds` 中只有 `clientfd1`，`sockfd` 和 `clientfd2` 不再被监视。
- 如果有新的客户端尝试连接，`sockfd` 不会被检测到，导致无法接受新的连接。
- 如果 `clientfd2` 后续有数据到达，它也不会被检测到，因为 `clientfd2` 已经从 `rfds` 中被清除。

因此，通过设置 `rset` 作为 `rfds` 的临时视图，可以避免这些问题，确保 `rfds` 的完整性和正确性。

---

通过以上分析，我们可以看到，设置`rfds`和`rset`两个`FD_SET`是为了在多路复用中保持文件描述符的正确性和完整性，避免因`select()`的破坏性操作而导致的问题。



### 3. poll 多路复用
用`poll()`实现多路复用

```c
#include <stdio.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h> 
#include <sys/poll.h>
#include <sys/select.h>
#include <errno.h>
#include <unistd.h>

int main() {
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);

    struct sockaddr_in sockaddr;
    memset(&sockaddr, 0, sizeof(sockaddr));

    sockaddr.sin_family = AF_INET;
    sockaddr.sin_addr.s_addr = htons(INADDR_ANY);
    sockaddr.sin_port = htons(2048);

    if (-1 == bind(sockfd, (struct sockaddr *)&sockaddr, sizeof(sockaddr))) {
        perror("bind");
        return -1;
    }

    listen(sockfd, 10);

    struct pollfd fds[1024];
    fds[sockfd].fd = sockfd;
    fds[sockfd].events = POLLIN;

    int maxfd = sockfd; 

    while(1) {
        int nready = poll(fds, maxfd+1, -1);
        if (fds[sockfd].revents & POLLIN) {
            struct sockaddr_in clientaddr;
            memset(&clientaddr, 0, sizeof(clientaddr));
            socklen_t len = sizeof(clientaddr);
            int clientfd = accept(sockfd, (struct sockaddr *)&clientaddr, &len);
            printf("accept\n");

            fds[clientfd].fd = clientfd;
            fds[clientfd].events = POLLIN;
            maxfd = clientfd;
        }

        for (int i = sockfd+1; i <= maxfd; i ++) {
            if(fds[i].revents & POLLIN) {
                char buffer[128];
                int count = recv(i, buffer, 128, 0);
                printf("clientfd: %d, recv: %s\n", i, buffer);
                if (count == 0) {
                    fds[i].fd = -1;
                    fds[i].events = 0;
                    close(i);
                    continue;
                }
                send(i, buffer, count, 0);
            }
        }
    }

}
```
`poll()`相较于`select()`的优点就是参数少了，只管理一个`struct pollfd[]` 。

### 4. epoll 多路复用

用`epoll()`实现多路复用

```c
#include <stdio.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#include <sys/select.h>
#include <sys/epoll.h>

int main() {
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);

    struct sockaddr_in sockaddr;
    socklen_t len = sizeof(sockaddr);

    sockaddr.sin_family = AF_INET;
    sockaddr.sin_addr.s_addr = htons(INADDR_ANY);
    sockaddr.sin_port = htons(2048);

    if (bind(sockfd, (struct sockaddr *)&sockaddr, sizeof(sockaddr)) == -1) {
        perror("bind");
        return -1;
    }

    listen(sockfd, 10);

    // 原来参数的作用是表示epoll的容量，现在改成链表实现了，参数弃用，大于1即可
    int epfd = epoll_create(1);
    struct epoll_event ev;
    ev.events = EPOLLIN;
    ev.data.fd = sockfd;

    epoll_ctl(epfd, EPOLL_CTL_ADD, sockfd, &ev);

    struct epoll_event events[1024] = {0};

    while(1) {
        int nready = epoll_wait(epfd, events, 1024, -1);
        for (int i = 0; i < nready; i ++) {
            int connfd = events[i].data.fd;
            if(connfd == sockfd) {
                struct sockaddr_in clientaddr;
                socklen_t len = sizeof(clientaddr);
                int clientfd = accept(sockfd, (struct sockaddr *)&clientaddr, &len);

                ev.events = EPOLLIN;
                ev.data.fd = clientfd;

                epoll_ctl(epfd, EPOLL_CTL_ADD, clientfd, &ev);
            } else if(events[i].events & EPOLLIN) {
                char buffer[128];
                int count = recv(connfd, buffer, 128, 0);
                if(count == 0) {
                    epoll_ctl(epfd, EPOLL_CTL_DEL, connfd, &events[i]);
                    close(connfd);
                    continue;
                }

                send(connfd, buffer, count, 0);
            }
        }
    }
}
```

> `poll()`在系统调用前后都需要拷贝`struct pollfd[]`，当管理的文件描述符很多的时候会发生频繁的拷贝而成为性能瓶颈。`epoll()`使用内存映射(mmap)技术实现事件表`struct epoll_event[]`在内核空间和用户空间共享。
