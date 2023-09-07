> RPC在哪一层？

## socket编程——多线程TCP

对应[大丙的第四章](https://subingwen.cn/linux/#%E7%AC%AC4%E7%AB%A0-%E5%A5%97%E6%8E%A5%E5%AD%97%E9%80%9A%E4%BF%A1)

​	典型的网络应用是由一对程序(即客户程序和服务器程序)组成的,它们位于两个不同的端系统中。当运行这两个程序时，创建了一个客户进程和一个服务器进程，同时它们通过从套接字读出和写人数据在彼此之间进行通信。

> 完成了简单的服务端和客户端通信代码的编写，并增加了std::thread和threadpool两种多线程的实现

对于多线程，共享代码段，堆区，全局数据区，打开的文件 (文件描述符表) 都是线程共享的。

new和malloc都分配在堆上，局部变量在栈上

### server.cpp——pthread

> thread和线程池实现都没定义数组保存文件描述符，会发生数据覆盖么？测试一下。封装成类的话呢？？？

不同的进程有着自己的PCB，因此文件描述符都是独有的

而线程共享文件描述符，可能会互相覆盖。在编写多线程版并发服务器代码的时候，需要注意父子线程共用同一个地址空间中的文件描述符，因此每当在主线程中建立一个新的连接，都需要将得到文件描述符值保存起来，不能在同一变量上进行覆盖，这样做丢失了之前的文件描述符值也就不知道怎么和客户端通信了。

在上面示例代码中是将成功建立连接之后得到的用于通信的文件描述符值保存到了一个全局数组中，每个子线程需要和不同的客户端通信，需要的文件描述符值也就不一样，只要保证存储每个有效文件描述符值的变量对应不同的内存地址，在使用的时候就不会发生数据覆盖的现象，造成通信数据的混乱了

文件描述符初始化为-1，代表这是可用的结构体，在创建子线程之前先判断哪一个结构体是可用的，如果128个客户端都被用了，则睡眠、i-1，直到有可用的文件描述符

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>
#include <pthread.h>

struct SockInfo
{
    int fd;                      // 通信
    pthread_t tid;               // 线程ID
    struct sockaddr_in addr;     // 地址信息
};

struct SockInfo infos[128];

void* working(void* arg)
{
    while(1)
    {
        struct SockInfo* info = (struct SockInfo*)arg;
        // 接收数据
        char buf[1024];
        int ret = read(info->fd, buf, sizeof(buf));
        if(ret == 0)
        {
            printf("客户端已经关闭连接...\n");
            info->fd = -1;
            break;
        }
        else if(ret == -1)
        {
            printf("接收数据失败...\n");
            info->fd = -1;
            break;
        }
        else
        {
            write(info->fd, buf, strlen(buf)+1);
        }
    }
    return NULL;
}

int main()
{
    // 1. 创建用于监听的套接字
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if(fd == -1)
    {
        perror("socket");
        exit(0);
    }

    // 2. 绑定
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;          // ipv4
    addr.sin_port = htons(8989);        // 字节序应该是网络字节序
    addr.sin_addr.s_addr =  INADDR_ANY; // == 0, 获取IP的操作交给了内核
    int ret = bind(fd, (struct sockaddr*)&addr, sizeof(addr));
    if(ret == -1)
    {
        perror("bind");
        exit(0);
    }

    // 3.设置监听
    ret = listen(fd, 100);
    if(ret == -1)
    {
        perror("listen");
        exit(0);
    }

    // 4. 等待, 接受连接请求
    int len = sizeof(struct sockaddr);

    // 数据初始化
    int max = sizeof(infos) / sizeof(infos[0]);
    for(int i=0; i<max; ++i)
    {
        bzero(&infos[i], sizeof(infos[i]));
        infos[i].fd = -1;
        infos[i].tid = -1;
    }

    // 父进程监听, 子进程通信
    while(1)
    {
        // 创建子线程
        struct SockInfo* pinfo;
        for(int i=0; i<max; ++i)
        {
            if(infos[i].fd == -1)
            {
                pinfo = &infos[i];
                break;
            }
            if(i == max-1)
            {
                sleep(1);
                i--;
            }
        }

        int connfd = accept(fd, (struct sockaddr*)&pinfo->addr, &len);
        printf("parent thread, connfd: %d\n", connfd);
        if(connfd == -1)
        {
            perror("accept");
            exit(0);
        }
        pinfo->fd = connfd;
        pthread_create(&pinfo->tid, NULL, working, pinfo);
        pthread_detach(pinfo->tid);
    }

    // 释放资源
    close(fd);  // 监听

    return 0;
}
```



### server.cpp——thread

```shell
g++ server.cpp -pthread
```

这里使用cpp中的std::thread实现服务端的多线程

注意，这里省略了pthread版程序中用数组存储文件描述符、id号等，一是因为thread不需要像pthread一样传入id号，二是因为存这个东西意义不大，懒得写了。所以区别就是这里没这个数组来记录，且没这个数组来限制子线程的数量

```cpp
#include<arpa/inet.h>
#include<unistd.h>
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#include<thread>
using namespace std;

void working(int connectFd, sockaddr_in& kehu){
    char ip[32] = {0};
    printf("客户端IP：%s, 端口：%d\n", 
    inet_ntop(AF_INET, &kehu.sin_addr.s_addr, ip, sizeof(ip)), ntohs(kehu.sin_port));
    
    while (1) {
		char buff[1024];
        memset(buff, 0, sizeof(buff));
		int len = read(connectFd, buff, sizeof(buff));
		if (len > 0) {
			printf("%s",buff);
            write(connectFd, buff, len);
		} else if (len == 0) {
			printf("断开连接\n");
			break;
		} else
        {
            perror("read");
            break;
        }
    }
    close(connectFd);
    
}

// 服务端
int main()
{
    int listenFd = socket(AF_INET, SOCK_STREAM, 0);  // 使用IPv4格式的ip，并使用TCP
    if (listenFd == -1) {
		perror("socket");
        exit(0);
    }
    // 本地的IP和端口的结构体
    struct sockaddr_in fuwu;
    fuwu.sin_family = AF_INET;
    // 转换为网络字节序的大端
    // 随便写个999端口
    fuwu.sin_port = htons(10000);
    // INADDR_ANY为任意的IP地址，读的是本地网卡的实际IP地址
    fuwu.sin_addr.s_addr = INADDR_ANY;
    int ifBind = bind(listenFd, (struct sockaddr*)&fuwu, sizeof(fuwu));

    // 监听
    ifBind = listen(listenFd, 128);

    struct sockaddr_in kehu;
    
    socklen_t addrlen = sizeof(kehu);
    while (1) {
        int connectFd = accept(listenFd, (struct sockaddr*)&kehu, &addrlen);
        if (connectFd == -1) {
            break;
        }
        thread t(working,connectFd, ref(kehu));
        t.detach();
    }


    close(listenFd);
    
    return 0;
}
```

### server.cpp——ThreadPool

```shell
g++ file.cpp -pthread -ltdpool -std=c++17 -o server
```

使用线程池实现服务端的多线程

```cpp
#include<arpa/inet.h>
#include<unistd.h>
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#include<thread>
using namespace std;
#include "threadpool.h"

class MyTask : public Task
{
public:
	MyTask(int connectFd, sockaddr_in& kehu)
		: connectFd(connectFd)
		, kehu(kehu)
	{}
	Any run() {
		std::cout << "tid:" << std::this_thread::get_id()
			<< "begin!" << std::endl;
		std::this_thread::sleep_for(std::chrono::seconds(5));
		
        char ip[32] = {0};
        printf("客户端IP：%s, 端口：%d\n", 
        inet_ntop(AF_INET, &kehu.sin_addr.s_addr, ip, sizeof(ip)), ntohs(kehu.sin_port));
        
        // 
        while (1) {
            char buff[1024];
            memset(buff, 0, sizeof(buff));
            int len = read(connectFd, buff, sizeof(buff));
            if (len > 0) {
                printf("%s",buff);
                write(connectFd, buff, len);
            } else if (len == 0) {
                printf("断开连接\n");
                break;
            } else
            {
                perror("read");
                break;
            }
        }
        close(connectFd);
		std::cout << "tid:" << std::this_thread::get_id()
			<< "end!" << std::endl;
        return 0;
	}
private:
	int connectFd;
    sockaddr_in& kehu;
};

// 服务端
int main()
{
    ThreadPool pool;
    pool.setMode(PoolMode::MODE_CACHED);
    pool.start(3);
    
    int listenFd = socket(AF_INET, SOCK_STREAM, 0);  // 使用IPv4格式的ip，并使用TCP
    if (listenFd == -1) {
		perror("socket");
        exit(0);
    }
    // 本地的IP和端口的结构体
    struct sockaddr_in fuwu;
    fuwu.sin_family = AF_INET;
    // 转换为网络字节序的大端
    // 随便写个999端口
    fuwu.sin_port = htons(10000);
    // INADDR_ANY为任意的IP地址，读的是本地网卡的实际IP地址
    fuwu.sin_addr.s_addr = INADDR_ANY;
    int ifBind = bind(listenFd, (struct sockaddr*)&fuwu, sizeof(fuwu));

    // 监听
    ifBind = listen(listenFd, 128);

    struct sockaddr_in kehu;
    
    socklen_t addrlen = sizeof(kehu);
    while (1) {
        int connectFd = accept(listenFd, (struct sockaddr*)&kehu, &addrlen);
        if (connectFd == -1) {
            break;
        }
        pool.submitTask(std::make_shared<MyTask>(connectFd, ref(kehu)));
    }


    close(listenFd);
    printf("over");
    
    return 0;
}
```

### kehu.c

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

int main()
{
    // 1. 创建通信的套接字
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if(fd == -1)
    {
        perror("socket");
        exit(0);
    }

    // 2. 连接服务器
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(10000);   // 大端端口
    inet_pton(AF_INET, "172.24.36.151", &addr.sin_addr.s_addr);

    int ret = connect(fd, (struct sockaddr*)&addr, sizeof(addr));
    if(ret == -1)
    {
        perror("connect");
        exit(0);
    }

    // 3. 和服务器端通信
    int number = 0;
    while(1)
    {
        // 发送数据
        char buf[1024];
        sprintf(buf, "你好, 服务器...%d\n", number++);
        write(fd, buf, strlen(buf)+1);
        
        // 接收数据
        memset(buf, 0, sizeof(buf));
        int len = read(fd, buf, sizeof(buf));
        if(len > 0)
        {
            printf("服务器say: %s\n", buf);
        }
        else if(len  == 0)
        {
            printf("服务器断开了连接...\n");
            break;
        }
        else
        {
            perror("read");
            break;
        }
        sleep(1);   // 每隔1s发送一条数据
    }

    close(fd);

    return 0;
}
```

## 对TCP通信操作进行封装

参照[大丙](https://subingwen.cn/linux/socket-class/)

源文件存放在云服务器的/root/linux/tcp目录下

```shell
# 编译客户端：
g++ file.cpp tcpp.cpp -o server -pthread
# 编译服务端
g++ kehu.cpp tcpp.cpp -o kehu
```

### 通信类TcpSocket

负责创建通信套接字、connect以及读写数据

```cpp
//////////////////////////// 声明
class TcpSocket
{
public:
    TcpSocket();
    TcpSocket(int socket);
    ~TcpSocket();
    int connectToHost(string ip, unsigned short port);
    int sendMsg(string msg);
    string recvMsg();

private:
    int readn(char* buf, int size);
    int writen(const char* msg, int size);

private:
    int m_fd;	// 通信的套接字
};


///////////////////////// 定义
// 在客户端使用，通过这个套接字对象再和服务器进行连接，之后就可以通信了
TcpSocket::TcpSocket() : m_fd(socket(AF_INET, SOCK_STREAM, 0)) {}
// 有参构造主要在服务器端使用，当服务器端得到了一个用于通信的套接字对象之后，就可以基于这个套接字直接通信，因此不需要再次进行连接操作
TcpSocket::TcpSocket(int socket) : m_fd(socket) {}

TcpSocket::~TcpSocket()
{
    if (m_fd > 0)
    {
        close(m_fd);
    }
}

int TcpSocket::connectToHost(string ip, unsigned short port)
{
    // 连接服务器IP port
    struct sockaddr_in saddr;
    saddr.sin_family = AF_INET;
    saddr.sin_port = htons(port);
    inet_pton(AF_INET, ip.data(), &saddr.sin_addr.s_addr);
    int ret = connect(m_fd, (struct sockaddr*)&saddr, sizeof(saddr));
    if (ret == -1)
    {
        perror("connect");
        return -1;
    }
    cout << "成功和服务器建立连接..." << endl;
    return ret;
}

int TcpSocket::sendMsg(string msg)
{
    // 申请内存空间: 数据长度 + 包头4字节(存储数据长度)
    char* data = new char[msg.size() + 4];
    int bigLen = htonl(msg.size());
    memcpy(data, &bigLen, 4);
    memcpy(data + 4, msg.data(), msg.size());
    // 发送数据
    int ret = writen(data, msg.size() + 4);
    delete[]data;
    return ret;
}

string TcpSocket::recvMsg()
{
    // 接收数据
    // 1. 读数据头
    int len = 0;
    readn((char*)&len, 4);
    len = ntohl(len);
    cout << "数据块大小: " << len << endl;

    // 根据读出的长度分配内存
    char* buf = new char[len + 1];
    int ret = readn(buf, len);
    if (ret != len)
    {
        return string();
    }
    buf[len] = '\0';
    string retStr(buf);
    delete[]buf;

    return retStr;
}
// 避免TCP粘包问题
int TcpSocket::readn(char* buf, int size)
{
    int nread = 0;
    int left = size;
    char* p = buf;

    while (left > 0)
    {
        if ((nread = read(m_fd, p, left)) > 0)
        {
            p += nread;
            left -= nread;
        }
        else if (nread == -1)
        {
            return -1;
        }
    }
    return size;
}
// 避免TCP粘包问题
int TcpSocket::writen(const char* msg, int size)
{
    int left = size;
    int nwrite = 0;
    const char* p = msg;

    while (left > 0)
    {
        if ((nwrite = write(m_fd, msg, left)) > 0)
        {
            p += nwrite;
            left -= nwrite;
        }
        else if (nwrite == -1)
        {
            return -1;
        }
    }
    return size;
}
```

### 服务器类TcpServer

负责创建监听套接字、listen、bind、accept

```cpp
////////////////////////////// 声明
class TcpServer
{
public:
    TcpServer();
    ~TcpServer();
    int setListen(unsigned short port);
    TcpSocket* acceptConn(struct sockaddr_in* addr = nullptr);

private:
    int m_fd;	// 监听的套接字
};

////////////////////////////// 定义
TcpServer::TcpServer()
{
    m_fd = socket(AF_INET, SOCK_STREAM, 0);
}

TcpServer::~TcpServer()
{
    close(m_fd);
}

int TcpServer::setListen(unsigned short port)
{
    struct sockaddr_in saddr;
    saddr.sin_family = AF_INET;
    saddr.sin_port = htons(port);
    saddr.sin_addr.s_addr = INADDR_ANY;  // 0 = 0.0.0.0
    int ret = bind(m_fd, (struct sockaddr*)&saddr, sizeof(saddr));
    if (ret == -1)
    {
        perror("bind");
        return -1;
    }
    cout << "套接字绑定成功, ip: "
        << inet_ntoa(saddr.sin_addr)
        << ", port: " << port << endl;

    ret = listen(m_fd, 128);
    if (ret == -1)
    {
        perror("listen");
        return -1;
    }
    cout << "设置监听成功..." << endl;

    return ret;
}

TcpSocket* TcpServer::acceptConn(sockaddr_in* addr)
{
    if (addr == NULL)
    {
        return nullptr;
    }

    socklen_t addrlen = sizeof(struct sockaddr_in);
    int cfd = accept(m_fd, (struct sockaddr*)addr, &addrlen);
    if (cfd == -1)
    {
        perror("accept");
        return nullptr;
    }
    printf("成功和客户端建立连接...\n");
    return new TcpSocket(cfd);
}
```

### 客户端

```cpp
int main()
{
    // 1. 创建通信的套接字
    TcpSocket tcp;

    // 2. 连接服务器IP port
    int ret = tcp.connectToHost("192.168.237.131", 10000);
    if (ret == -1)
    {
        return -1;
    }

    // 3. 通信
    int fd1 = open("english.txt", O_RDONLY);
    int length = 0;
    char tmp[100];
    memset(tmp, 0, sizeof(tmp));
    while ((length = read(fd1, tmp, sizeof(tmp))) > 0)
    {
        // 发送数据
        tcp.sendMsg(string(tmp, length));

        cout << "send Msg: " << endl;
        cout << tmp << endl << endl << endl;
        memset(tmp, 0, sizeof(tmp));

        // 接收数据
        usleep(300);
    }

    sleep(10);

    return 0;
}
```

### 服务器端

```cpp
#include<arpa/inet.h>
#include<unistd.h>
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#include<thread>
using namespace std;

void working(TcpServer& s, TcpSocket& tcp, sockaddr_in& kehu){
    char ip[32];
    printf("客户端IP：%s, 端口：%d\n", 
    inet_ntop(AF_INET, &kehu.sin_addr.s_addr, ip, sizeof(ip)), ntohs(kehu.sin_port));
    
    // 
    while (1) {
        printf("接收数据: .....\n");
        string msg = tcp.recvMsg();
        if (!msg.empty())
        {
            cout << msg << endl << endl << endl;
        }
        else
        {
            break;
        }
    }
    delete tcp;
}

// 服务端
int main()
{
    // 1. 创建监听的套接字
    TcpServer s;
    // 2. 绑定本地的IP port并设置监听
    s.setListen(10000);
    // 3. 阻塞并等待客户端的连接
    while (1) {
        sockaddr_in addr;
        TcpSocket* tcp = s.acceptConn(&addr);
        if (tcp == nullptr)
        {
            cout << "重试...." << endl;
            continue;
        }

        thread t(working, s, tcp, ref(addr));
        t.detach();
    }
    return 0;
}
```

## select

### select

sever.c

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/select.h>
#include <ctype.h>
int main()
{
    // 1. 创建监听的fd
    int lfd = socket(AF_INET, SOCK_STREAM, 0);

    // 2. 绑定
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(9999);
    addr.sin_addr.s_addr = INADDR_ANY;
    bind(lfd, (struct sockaddr*)&addr, sizeof(addr));

    // 3. 设置监听
    listen(lfd, 128);

    // 将监听的fd的状态检测委托给内核检测
    int maxfd = lfd;
    // 初始化检测的读集合
    fd_set rdset;  // 存储待检测的原始数据
    fd_set rdtemp;
    // 清零
    FD_ZERO(&rdset);
    // 将监听的lfd设置到检测的读集合中
    FD_SET(lfd, &rdset);
    // 通过select委托内核检测读集合中的文件描述符状态, 检测read缓冲区有没有数据
    // 如果有数据, select解除阻塞返回
    // 应该让内核持续检测
    while(1)
    {
        // 默认阻塞
        // rdset 中是委托内核检测的所有的文件描述符
        rdtemp = rdset;
        int num = select(maxfd+1, &rdtemp, NULL, NULL, NULL);
        // rdset中的数据被内核改写了, 只保留了发生变化的文件描述的标志位上的1, 没变化的改为0
        // 只要rdset中的fd对应的标志位为1 -> 缓冲区有数据了

        // 判断是不是监听的fd
        if(FD_ISSET(lfd, &rdtemp))
        {
            // 接受连接请求, 这个调用不阻塞
            struct sockaddr_in cliaddr;
            int cliLen = sizeof(cliaddr);
            int cfd = accept(lfd, (struct sockaddr*)&cliaddr, &cliLen);

            // 得到了有效的文件描述符
            // 通信的文件描述符添加到读集合
            // 在下一轮select检测的时候, 就能得到缓冲区的状态
            FD_SET(cfd, &rdset);
            // 重置最大的文件描述符
            maxfd = cfd > maxfd ? cfd : maxfd;
        }

        // 遍历所有的文件描述符，看这个范围内的文件描述符是否读缓冲区有数据
        for(int i=0; i<maxfd+1; ++i)
        {
            if(i != lfd && FD_ISSET(i, &rdtemp))
            {
                // 接收数据
                char buf[1024] = {0};
                // 一次只能接收10个字节, 客户端一次发送100个字节
                // 一次是接收不完的, 文件描述符对应的读缓冲区中还有数据
                // 下一轮select检测的时候, 内核还会标记这个文件描述符缓冲区有数据 -> 再读一次
                // 	循环会一直持续, 知道缓冲区数据被读完位置
                int len = read(i, buf, sizeof(buf));
                if(len == 0)
                {
                    printf("客户端关闭了连接...\n");
                    // 将检测的文件描述符从读集合中删除
                    FD_CLR(i, &rdset);
                    close(i);
                }
                else if(len > 0)
                {
                    // 收到了数据
                    printf("read buf = %s\n", buf);
                    // 小写转大写
                    for(int i = 0; i < len; i++)
                        buf[i] = toupper(buf[i]);
                    printf("after buf = %s\n", buf);
                    // 发送数据
                    write(i, buf, strlen(buf)+1);
                }
                else
                {
                    // 异常
                    perror("read");
                }
            }
        }
    }
    return 0;
}
```

kehu.c

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

int main()
{
    // 1. 创建用于通信的套接字
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if(fd == -1)
    {
        perror("socket");
        exit(0);
    }

    // 2. 连接服务器
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;     // ipv4
    addr.sin_port = htons(9999);   // 服务器监听的端口, 字节序应该是网络字节序
    inet_pton(AF_INET, "127.0.0.1", &addr.sin_addr.s_addr);
    int ret = connect(fd, (struct sockaddr*)&addr, sizeof(addr));
    if(ret == -1)
    {
        perror("connect");
        exit(0);
    }

    // 通信
    while(1)
    {
        // 读数据
        char recvBuf[1024];
        // 写数据
        // sprintf(recvBuf, "data: %d\n", i++);
        fgets(recvBuf, sizeof(recvBuf), stdin);
        write(fd, recvBuf, strlen(recvBuf)+1);
        // 如果客户端没有发送数据, 默认阻塞
        read(fd, recvBuf, sizeof(recvBuf));
        printf("recv buf: %s\n", recvBuf);
        sleep(1);
    }

    // 释放资源
    close(fd); 

    return 0;
}
```

### 多线程select

​	在while(1)中，检测监听描述符来建立连接和检测通信描述符是线性执行的。如果有多个客户端同时向服务器端发送请求，服务器端要一个个的进行处理

gcc sever.c -o sever -l pthread

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/select.h>
#include <sys/types.h>
#include <ctype.h>
#include <pthread.h>
// redset和maxfd是共享的，需要锁
pthread_mutex_t mutex;
// 定义一个结构体，方便之后传参数到子线程中
typedef struct fdinfo {
    int fd;
    int *maxfd;
    fd_set* rdset;
}FDInfo;

void* acceptConn(void* arg) {
    printf("子线程ID：%ld\n", pthread_self());
    FDInfo* info = (FDInfo*)arg;
    // 接受连接请求, 这个调用不阻塞
    struct sockaddr_in cliaddr;
    int cliLen = sizeof(cliaddr);
    int cfd = accept(info->fd, (struct sockaddr*)&cliaddr, &cliLen);

    // 得到了有效的文件描述符
    // 通信的文件描述符添加到读集合
    // 在下一轮select检测的时候, 就能得到缓冲区的状态
    pthread_mutex_lock(&mutex);
    FD_SET(cfd, info->rdset);
    // 重置最大的文件描述符
    *info->maxfd = cfd > *info->maxfd ? cfd : *info->maxfd;
    pthread_mutex_unlock(&mutex);
    free(info);
    return NULL;
}
// 仅完成收到数据-》转换为大写-》发送数据，就退出
void* communication(void* arg){
    printf("子线程ID：%ld\n", pthread_self());
    FDInfo* info = (FDInfo*)arg;

    // 接收数据
    char buf[1024] = {0};
    // 一次只能接收10个字节, 客户端一次发送100个字节
    // 一次是接收不完的, 文件描述符对应的读缓冲区中还有数据
    // 下一轮select检测的时候, 内核还会标记这个文件描述符缓冲区有数据 -> 再读一次
    // 	循环会一直持续, 知道缓冲区数据被读完位置
    int len = read(info->fd, buf, sizeof(buf));
    if(len == 0)
    {
        printf("客户端关闭了连接...\n");
        // 将检测的文件描述符从读集合中删除
        pthread_mutex_lock(&mutex);
        FD_CLR(info->fd, info->rdset);
        pthread_mutex_unlock(&mutex);
        close(info->fd);
        free(info);
        return NULL;
    }
    else if(len > 0)
    {
        // 收到了数据
        printf("read buf = %s\n", buf);
        // 小写转大写
        for(int i = 0; i < len; i++)
            buf[i] = toupper(buf[i]);
        printf("after buf = %s\n", buf);
        // 发送数据
        int ret = write(info->fd, buf, strlen(buf)+1);
        if (ret == -1) {
            perror("send error");
        }
        free(info);
        return NULL;
    }
    else
    {
        // 没有数据传输了
        perror("read");
        // 释放内存
        free(info);
        return NULL;
    }
}
int main()
{
    pthread_mutex_init(&mutex, NULL);
    // 1. 创建监听的fd
    int lfd = socket(AF_INET, SOCK_STREAM, 0);

    // 2. 绑定
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(9999);
    addr.sin_addr.s_addr = INADDR_ANY;
    bind(lfd, (struct sockaddr*)&addr, sizeof(addr));

    // 3. 设置监听
    listen(lfd, 128);

    // 将监听的fd的状态检测委托给内核检测
    int maxfd = lfd;
    // 初始化检测的读集合
    fd_set rdset;  // 存储待检测的原始数据
    fd_set rdtemp;
    // 清零
    FD_ZERO(&rdset);
    // 将监听的lfd设置到检测的读集合中
    FD_SET(lfd, &rdset);
    // 通过select委托内核检测读集合中的文件描述符状态, 检测read缓冲区有没有数据
    // 如果有数据, select解除阻塞返回
    // 应该让内核持续检测
    while(1)
    {
        // 默认阻塞
        // rdset 中是委托内核检测的所有的文件描述符
        pthread_mutex_lock(&mutex);
        rdtemp = rdset;
        pthread_mutex_unlock(&mutex);
        int num = select(maxfd+1, &rdtemp, NULL, NULL, NULL);
        // rdset中的数据被内核改写了, 只保留了发生变化的文件描述的标志位上的1, 没变化的改为0
        // 只要rdset中的fd对应的标志位为1 -> 缓冲区有数据了


        // 判断是不是监听的fd
        if(FD_ISSET(lfd, &rdtemp))
        {
            FDInfo* info = (FDInfo*)malloc(sizeof(FDInfo));
            info->fd = lfd;
            info->maxfd = &maxfd;
            info->rdset = &rdset;
            // 在子线程里对上两个数据做加锁操作
            pthread_t tid;
            pthread_create(&tid, NULL, acceptConn, info);
            pthread_detach(tid);
        }

        // 遍历所有的文件描述符，看这个范围内的文件描述符是否读缓冲区有数据
        for(int i=0; i<maxfd+1; ++i)
        {
            if(i != lfd && FD_ISSET(i, &rdtemp))
            {
                FDInfo* info = (FDInfo*)malloc(sizeof(FDInfo));
                info->fd = i;
                info->rdset = &rdset;
                pthread_t tid;
                pthread_create(&tid, NULL, communication, info);
                pthread_detach(tid);
            }
        }
    }
    close(lfd);
    pthread_mutex_destroy(&mutex);
    return 0;
}
```

kehu.c

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

int main()
{
    // 1. 创建用于通信的套接字
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if(fd == -1)
    {
        perror("socket");
        exit(0);
    }

    // 2. 连接服务器
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;     // ipv4
    addr.sin_port = htons(9999);   // 服务器监听的端口, 字节序应该是网络字节序
    inet_pton(AF_INET, "127.0.0.1", &addr.sin_addr.s_addr);
    int ret = connect(fd, (struct sockaddr*)&addr, sizeof(addr));
    if(ret == -1)
    {
        perror("connect");
        exit(0);
    }

    int num = 0;
    // 通信
    while(1)
    {
        // 读数据
        char recvBuf[1024];
        // 写数据
        sprintf(recvBuf, "data: %d\n", num++);
        write(fd, recvBuf, strlen(recvBuf)+1);
        // 如果客户端没有发送数据, 默认阻塞
        read(fd, recvBuf, sizeof(recvBuf));
        printf("recv buf: %s\n", recvBuf);
        sleep(1);
    }

    // 释放资源
    close(fd); 

    return 0;
}
```

## epoll

### LT

默认的水平模式，即LT模式

```cpp
// sever.c
#include <stdio.h>
#include <ctype.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <sys/epoll.h>

// server
int main(int argc, const char* argv[])
{
    // 创建监听的套接字
    int lfd = socket(AF_INET, SOCK_STREAM, 0);
    if(lfd == -1)
    {
        perror("socket error");
        exit(1);
    }

    // 绑定
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(9999);
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);  // 本地多有的ＩＰ
    
    // 设置端口复用，可忽略
    int opt = 1;
    setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

    // 绑定端口
    int ret = bind(lfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
    if(ret == -1)
    {
        perror("bind error");
        exit(1);
    }

    // 监听
    ret = listen(lfd, 64);
    if(ret == -1)
    {
        perror("listen error");
        exit(1);
    }

    // 现在只有监听的文件描述符
    // 所有的文件描述符对应读写缓冲区状态都是委托内核进行检测的epoll
    // 创建一个epoll模型，注意填入的参数需要大于1，没有实际性的含义
    int epfd = epoll_create(100);
    if(epfd == -1)
    {
        perror("epoll_create");
        exit(0);
    }

    // 往epoll实例中 添加 需要检测的节点, 现在只有监听的文件描述符
    struct epoll_event ev;
    ev.events = EPOLLIN;    // 检测lfd读读缓冲区是否有数据
    ev.data.fd = lfd;
    ret = epoll_ctl(epfd, EPOLL_CTL_ADD, lfd, &ev);
    if(ret == -1)
    {
        perror("epoll_ctl");
        exit(0);
    }

    // 
    struct epoll_event evs[1024];
    // size自动计算
    int size = sizeof(evs) / sizeof(struct epoll_event);

    // 持续检测
    while(1)
    {
        // 调用一次, 检测一次
        // 第二个参数是传出参数，这是一个结构体数组的地址。
        int num = epoll_wait(epfd, evs, size, -1);
        for(int i=0; i<num; ++i)
        {
            // 取出当前的文件描述符
            int curfd = evs[i].data.fd;
            // 判断这个文件描述符是不是用于监听的
            if(curfd == lfd)  // 监听的描述符
            {
                // 建立新的连接
                int cfd = accept(curfd, NULL, NULL);
                // 新得到的文件描述符添加到epoll模型中, 下一轮循环的时候就可以被检测了
                // 这里就直接在之前创建的ev上改了，因为反正是拷贝到epoll树里
                ev.events = EPOLLIN;    // 读缓冲区是否有数据
                ev.data.fd = cfd;
                ret = epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, &ev);
                if(ret == -1)
                {
                    perror("epoll_ctl-accept");
                    exit(0);
                }
            }
            else
            {
                // 处理通信的文件描述符
                // 接收数据
                char buf[1024];
                memset(buf, 0, sizeof(buf)); // 置零
                int len = recv(curfd, buf, sizeof(buf), 0);
                if(len == 0)
                {
                    printf("客户端已经断开了连接\n");
                    // 将这个文件描述符从epoll模型中删除
                    // 注意先删除再关闭
                    epoll_ctl(epfd, EPOLL_CTL_DEL, curfd, NULL);
                    close(curfd);
                }
                else if(len > 0)
                {
                    printf("客户端say: %s\n", buf);
                    send(curfd, buf, len, 0);
                }
                else
                {
                    perror("recv");
                    exit(0);
                } 
            }
        }
    }

    return 0;
}
```

### ET

​	如果使用 epoll 的边沿模式进行读事件的检测，有新数据达到只会通知一次，那么必须要保证得到通知后将数据全部从读缓冲区中读出。那么，应该如何读这些数据呢？

​	边沿模式需要我们循环读取数据，这时就需要把recv读取数据的函数放到循环里。而recv函数是堵塞的，即若读完之后没有其他数据读取了，整个程序就会堵塞在这个函数上。因此我们需要将这个通信的文件描述符设置为非堵塞的

```cpp
// 将文件描述符设置为非阻塞
int flag = fcntl(cfd, F_GETFL);  // F_GETFL 获取文件的状态标志
flag |= O_NONBLOCK;  // 将状态标志修改为 O_NONBLOCK 非阻塞模式
fcntl(cfd, F_SETFL, flag);  // F_SETFL 设置文件的状态标志
```

​	通过上述分析就可以得出一个结论：**epoll 在边沿模式下，必须要将套接字设置为非阻塞模式**，但是，这样就会引发另外的一个 bug，在非阻塞模式下，循环地将读缓冲区数据读到本地内存中，当缓冲区数据被读完了，调用的 read()/recv() 函数还会继续从缓冲区中读数据，此时函数调用就失败了，返回 - 1，对应的全局变量 errno 值为 EAGAIN 或者 EWOULDBLOCK 如果打印错误信息会得到如下的信息：Resource temporarily unavailable

```cpp
// 非阻塞模式下recv() / read()函数返回值 len == -1
int len = recv(curfd, buf, sizeof(buf), 0);
if(len == -1)
{
    if(errno == EAGAIN)
    {
        printf("数据读完了...\n");
    }
    else
    {
        perror("recv");
        exit(0);
    }
}
```

sever.c

```cpp
#include <stdio.h>
#include <ctype.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <fcntl.h>
#include <errno.h>

// server
int main(int argc, const char* argv[])
{
    // 创建监听的套接字
    int lfd = socket(AF_INET, SOCK_STREAM, 0);
    if(lfd == -1)
    {
        perror("socket error");
        exit(1);
    }

    // 绑定
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(9999);
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);  // 本地多有的ＩＰ
    // 127.0.0.1
    // inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr.s_addr);
    
    // 设置端口复用
    int opt = 1;
    setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

    // 绑定端口
    int ret = bind(lfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
    if(ret == -1)
    {
        perror("bind error");
        exit(1);
    }

    // 监听
    ret = listen(lfd, 64);
    if(ret == -1)
    {
        perror("listen error");
        exit(1);
    }

    // 现在只有监听的文件描述符
    // 所有的文件描述符对应读写缓冲区状态都是委托内核进行检测的epoll
    // 创建一个epoll模型
    int epfd = epoll_create(100);
    if(epfd == -1)
    {
        perror("epoll_create");
        exit(0);
    }

    // 往epoll实例中添加需要检测的节点, 现在只有监听的文件描述符
    struct epoll_event ev;
    ev.events = EPOLLIN;    // 检测lfd读读缓冲区是否有数据
    ev.data.fd = lfd;
    ret = epoll_ctl(epfd, EPOLL_CTL_ADD, lfd, &ev);
    if(ret == -1)
    {
        perror("epoll_ctl");
        exit(0);
    }


    struct epoll_event evs[1024];
    int size = sizeof(evs) / sizeof(struct epoll_event);
    // 持续检测
    while(1)
    {
        // 调用一次, 检测一次
        int num = epoll_wait(epfd, evs, size, -1);
        printf("==== num: %d\n", num);

        for(int i=0; i<num; ++i)
        {
            // 取出当前的文件描述符
            int curfd = evs[i].data.fd;
            // 判断这个文件描述符是不是用于监听的
            if(curfd == lfd)
            {
                // 建立新的连接
                int cfd = accept(curfd, NULL, NULL);
                // 将文件描述符设置为非阻塞
                // 得到文件描述符的属性
                int flag = fcntl(cfd, F_GETFL);
                flag |= O_NONBLOCK;
                fcntl(cfd, F_SETFL, flag);
                // 新得到的文件描述符添加到epoll模型中, 下一轮循环的时候就可以被检测了
                // 通信的文件描述符检测读缓冲区数据的时候设置为边沿模式
                ev.events = EPOLLIN | EPOLLET;    // 读缓冲区是否有数据
                ev.data.fd = cfd;
                ret = epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, &ev);
                if(ret == -1)
                {
                    perror("epoll_ctl-accept");
                    exit(0);
                }
            }
            else
            {
                // 处理通信的文件描述符
                // 接收数据
                char buf[5];
                memset(buf, 0, sizeof(buf));
                // 循环读数据
                while(1)
                {
                    int len = recv(curfd, buf, sizeof(buf), 0);
                    if(len == 0)
                    {
                        // 非阻塞模式下和阻塞模式是一样的 => 判断对方是否断开连接
                        printf("客户端断开了连接...\n");
                        // 将这个文件描述符从epoll模型中删除
                        epoll_ctl(epfd, EPOLL_CTL_DEL, curfd, NULL);
                        close(curfd);
                        break;
                    }
                    else if(len > 0)
                    {
                        // 通信
                        // 接收的数据打印到终端
                        write(STDOUT_FILENO, buf, len);
                        // 发送数据
                        send(curfd, buf, len, 0);
                    }
                    else
                    {
                        // len == -1
                        if(errno == EAGAIN)
                        {
                            printf("数据读完了...\n");
                            break;
                        }
                        else
                        {
                            perror("recv");
                            exit(0);
                        }
                    }
                }
            }
        }
    }

    return 0;
}
```

### 多线程ET epoll

epoll相关的函数都是线程安全的，因此不需要加锁

```cpp
```

## UDP

server.c

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

int main()
{
    // 1. 创建通信的套接字
    int fd = socket(AF_INET, SOCK_DGRAM, 0);
    if(fd == -1) {
        perror("socket");
        exit(0);
    }

    // 2. 通信的套接字和本地的IP与端口绑定
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(9999);    // 大端
    addr.sin_addr.s_addr = INADDR_ANY;
    int ret = bind(fd, (struct sockaddr*)&addr, sizeof(addr));
    if(ret == -1) {
        perror("bind");
        exit(0);
    }

    char buf[1024];
    char ipbuf[64];
    struct sockaddr_in cliaddr;
    int len = sizeof(cliaddr);
    // 3. 通信
    while(1)
    {
        // 接收数据
        memset(buf, 0, sizeof(buf));
        int rlen = recvfrom(fd, buf, sizeof(buf), 0, (struct sockaddr*)&cliaddr, &len);
        printf("客户端的IP地址: %s, 端口: %d\n",
               inet_ntop(AF_INET, &cliaddr.sin_addr.s_addr, ipbuf, sizeof(ipbuf)),
               ntohs(cliaddr.sin_port));
        printf("客户端say: %s\n", buf);

        // 回复数据
        // 数据回复给了发送数据的客户端
        sendto(fd, buf, rlen, 0, (struct sockaddr*)&cliaddr, sizeof(cliaddr));
    }

    close(fd);
    return 0;
}
```

kehu.c

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

int main()
{
    // 1. 创建通信的套接字
    int fd = socket(AF_INET, SOCK_DGRAM, 0);
    if(fd == -1)
    {
        perror("socket");
        exit(0);
    }
    
    // 初始化服务器地址信息
    struct sockaddr_in seraddr;
    seraddr.sin_family = AF_INET;
    seraddr.sin_port = htons(9999);    // 大端
    inet_pton(AF_INET, "192.168.1.100", &seraddr.sin_addr.s_addr);

    char buf[1024];
    char ipbuf[64];
    struct sockaddr_in cliaddr;
    int len = sizeof(cliaddr);
    int num = 0;
    // 2. 通信
    while(1)
    {
        sprintf(buf, "hello, udp %d....\n", num++);
        // 发送数据, 数据发送给了服务器
        sendto(fd, buf, strlen(buf)+1, 0, (struct sockaddr*)&seraddr, sizeof(seraddr));

        // 接收数据
        memset(buf, 0, sizeof(buf));
        recvfrom(fd, buf, sizeof(buf), 0, NULL, NULL);
        printf("服务器say: %s\n", buf);
        sleep(1);
    }

    close(fd);

    return 0;
}
```



IO多路复用就是将检测文件描述符读、写缓冲区的任务交给了内核，让内核来同时检测若干个文件描述符的状态，若条件满足（如读缓冲区有数据，写缓冲区有剩余），内核就将文件描述符通知给用户，用户调用时就不是堵塞的了。

​	对于select函数的第一个参数，为三个列表中最大的fd+1，因为select是基于线性表实现的，需要告诉内核我们要检测的范围，fd+1就是循环结束的条件。当然这里也可以写1024，因为文件描述符表中最多1024个文件描述符（0-1023）；

​	对于select的第二到第四个参数，传入的是指针类型，即地址，因为内核会对集合进行修改，传出修改后的集合。因此这三个参数是传入传出参数，传入时进行初始化，告诉内核要检测哪些文件描述符，传出满足条件的集合。

select poll epoll

* select是多平台，poll和epoll只能在linux下使用
* select和poll，底层是线性表；epoll底层是红黑树，检测事件触发时效率更高
* select的上限是1024，而poll、epoll没有限制















## 计算机网络——自顶向下方法



第三章 运输层

在发送端，运输层将带有TCP/UDP头部的报文段传输给网络层，网络层将其封装成网络层分组并向目的地发送。注意，中间经过的路由器仅作用于该数据报的网络层字段，即不检查封装在该数据报的运输层报文段的字段。在接收端，网络层从数据报中提取运输层报文段上交给运输层，运输层处理接收到的报文段，使报文段中的数据为应用进程使用。

**运输层和网络层的关系：**

网络层提供了主机之间的逻辑通信，使用IP协议，但不确保报文段的交付、不保证按序交付、不保证数据的完整性，被称为不可靠服务

而运输层为运行在不同主机上的进程之间提供了逻辑通信，将两个主机间IP的交付服务扩展为运行在主机上的两个进程之间的交付服务

**Socket**

在生成套接字时必须指定是选择UDP还是TCP

一个进程(作为网络应用的一部分)有一个或多个套接字
(socket)，它相当于从网络向进程传递数据和从进程向网络传递数据的门户。因此，在接收主机中的运输层实际上并没有直接将数据交付给进程，而是将数据交给了一个中间的套接字。由于在任一时刻，在接收主机上可能有不止一个套接字，所以每个套接字都有唯一的标识符，标识符的格式取决于它是UDP还是TCP套接字。而这个标识符就是端口号，在主机上的每个套接字能够分配一个端口号，当报文段到达主机时，运输层检查报文段中的目的端口号，并将其定向到相应的套接字。然后报文段中的数据通过套接字进入其所连接的进程。如我们将看到的那样，UDP大体上是这样做的。然而，也将如我们所见，TCP中的多路复用与多路分解更为复杂。

（注意，套接字与进程之间并非总是有着一一对应的关系，因为可以为每个新的客户连接创建一个具有新连接套接字的新线程）

​	一个UDP套接字是由目的IP地址和目的端口号来标识的。因此，如果两个UDP报文段有不同的源IP地址和/或源端口号，但具有相同的目的IP地址和目的端口号，那么这两个报文段将通过相同的目的套接字被定向到相同的目的进程。

​	而TCP套接字是由四元组来标识的，因此当一个TCP报文段从网络到达一台主机时，该主机使用全部4个值来将报文段定向(分解)到相应的套接字。特别与UDP不同的是，两个具有不同源IP地址或源端口号的到达TCP报文段将被定向到两个不同的套接字，除非TCP报文段携带了初始创建连接的请求

![image-20230306110406607](E:\MarkDown\picture\image-20230306110406607.png)



![image-20230306110427941](E:\MarkDown\picture\image-20230306110427941.png)

## **TCP/IP网络模型**



网络接口层的传输单位是帧（frame），IP 层的传输单位是包（packet），TCP 层的传输单位是段（segment），HTTP 的传输单位则是消息或报文（message）。但这些名词并没有什么本质的区分，可以统称为数据包。

![image-20230228205227688](E:\MarkDown\picture\image-20230228205227688.png)





**Linux网络协议栈**

应用程序需要通过系统调用，来跟 Socket 层进行数据交互。其中[LVS](https://juejin.cn/post/6966411996589719583)(Linux Virtual Server)即Linux虚拟服务器，是一个虚拟的服务器集群系统，被集成到Linux内核模块中，在Linux内核中实现了基于IP的数据请求负载均衡调度方案

![img](E:\MarkDown\picture\协议栈.png)













![img](E:\MarkDown\picture\7.jpg)

* `ICMP` **互联网控制报文协议**用于告知网络包传送过程中产生的错误以及各种控制信息。
* `ARP` 用于根据 IP 地址查询相应的以太网 MAC 地址。



## 一些名词

**NAPI机制**

​	NAPI即New API，它是混合「中断和轮询」的方式来接收网络包，它的核心概念就是**不采用中断的方式读取数据**，而是首先采用中断唤醒数据接收的服务程序，然后 `poll` 的方法来轮询数据。当有中断来了，驱动关闭中断，通知内核收包，内核软中断轮询当前网卡，在规定时间尽可能多的收包。时间用尽或者没有数据可收，内核再次开启中断，准备下一次收包。



**HTTP**

​	HTTP 是超文本传输协议，也就是**H**yperText **T**ransfer **P**rotocol。是一个双向协议，是一个在计算机世界里专门在「两点」之间「传输」文字、图片、音频、视频等「超文本」数据的「约定和规范」。两点意味着可以在客户端与服务器、服务器与服务器等进行传输。

**Host**

​	Host是HTTP中一个常见的字段。通过Host，客户端可以指定自己想访问的http服务器的域名/IP 地址和端口号。因为一个服务器上可能会有www.A.com、www.B.com等多个域名，有了Host字段，就可以将请求发往「同一台」服务器上的不同网站。

​	**Host** 请求头指明了请求将要发送到的服务器主机名和端口号。如果没有包含端口号，会自动使用被请求服务的默认端口（比如 HTTPS URL 使用 443 端口，HTTP URL 使用 80 端口）。所有 HTTP/1.1 请求报文中必须包含一个Host头字段。对于缺少Host头或者含有超过一个Host头的 HTTP/1.1 请求，可能会收到400（Bad Request）状态码。



GET 和 POST

​	在客户机和服务器之间进行请求-响应时，两种最常被用到的方法是：GET 和 POST。

* **GET** - 从指定的资源请求数据。
  根据 RFC 规范，**GET 的语义是从服务器获取指定的资源**，这个资源可以是静态的文本、页面、图片视频等。GET 请求的参数位置一般是写在 URL 中，URL 规定只能支持 ASCII，所以 GET 请求的参数只允许 ASCII 字符 ，而且浏览器会对  URL 的长度有限制（HTTP协议本身对 URL长度并没有做任何规定）。

* **POST** - 向指定的资源提交要被处理的数据。 

  根据 RFC 规范，**POST 的语义是根据请求负荷（报文body）对指定的资源做出处理**，具体的处理方式视资源类型而不同。POST 请求携带数据的位置一般是写在报文 body 中，body 中的数据可以是任意格式的数据，只要客户端与服务端协商好即可，而且浏览器不会对 body 大小做限制。

  比如，你在我文章底部，敲入了留言后点击「提交」（**暗示你们留言**），浏览器就会执行一次 POST 请求，把你的留言文字放进了报文 body 里，然后拼接好 POST 请求头，通过 TCP 协议发送给服务器。

![img](E:\MarkDown\picture\get-post.png)

![GET 请求](E:\MarkDown\picture\12-Get请求.png)



![POST 请求](E:\MarkDown\picture\13-Post请求.png)





Cookie

Cookie，有时也用其复数形式 Cookies。类型为“**小型文本文件**”，是某些网站为了辨别用户身份，进行Session跟踪而储存在用户本地终端上的数据（通常经过加密），由用户客户端计算机暂时或永久保存的信息。

常在HTTP/1.1中使用，因为此协议[无状态]，服务器没有记忆能力，它在完成有关联性的操作时会非常麻烦。例如登录->添加购物车->下单->结算->支付，这系列操作都要知道用户的身份才行。但服务器不知道这些请求是有关联的，每次都要问一遍身份信息。

这时就可以使用 **Cookie** 技术。Cookie通过在请求和响应报文中写入 Cookie 信息来控制客户端的状态。相当于，**在客户端第一次请求后，服务器会下发一个装有客户信息的「小贴纸」，后续客户端请求服务器的时候，带上「小贴纸」，服务器就能认得了了**，

![Cookie 技术](E:\MarkDown\picture\14-cookie技术.png)





**SSL/TLS协议**

SSL和TLS是一个东西，可视为同一个东西的不同阶段，现在一般都叫TLS

在HTTPS中使用，来解决HTTP存在的安全问题，添加到了HTTP与TCP层之间。可以实现信息加密、校验机制、身份证书

基本流程：

* 客户端向服务器索要并验证服务器的公钥。
* 双方协商生产「会话秘钥」。
* 双方采用「会话秘钥」进行加密通信。

首先，通过**混合加密**可以保证信息的机密性，混合加密即在通信建立前采用非对称加密的方式交换会话密匙，在通信中使用对称加密的会话密匙加密明文数据。这里先简单介绍下密钥交换算法，因为考虑到性能的问题，所以双方在加密应用信息时使用的是对称加密密钥，而对称加密密钥是不能被泄漏的，为了保证对称加密密钥的安全性，所以使用非对称加密的方式来保护对称加密密钥的协商，这个工作就是密钥交换算法负责的。下面将对密匙交换算法进行介绍。

其次，通过**摘要算法+数字签名**来保证传输的内容不被篡改。摘要算法（哈希函数）计算出内容的哈希值，与内容一起发送，接收方收到内容后进行哈希运算，得到的哈希值B与传来的哈希值A比对，相同则说明消息是完整的。

但[哈希值 + 内容]可能会被替换，因此需要证明接收到的内容是来自发送端的，这通过数字签名算法实现。首先使用非对称加密算法，发送端使用私钥对哈希值加密，接收端使用公钥对哈希值解密。私钥是由服务端保管，然后服务端会向客户端颁发对应的公钥。如果客户端收到的信息，能被公钥解密，就说明该消息是由服务器发送的。

但公钥可能是被伪造的，比如坏人伪造出一对公私钥，将客户端的公钥给替换了，然后用假的私钥加密，客户端用假的公钥发现可以解密，就当作是真消息了。所以需要**数字证书**，即 CA （数字证书认证机构）用他们的私钥，对服务端的公钥做了个数字签名，将服务端的[信息+公钥+数字签名]打包成数字证书。这样客户端再收到消息时，会先去CA用CA的公钥解密，验证数字证书是否合法，这就能保证公钥是真的来源于服务端了。

![数子证书工作流程](E:\MarkDown\picture\22-数字证书工作流程.png)





























































