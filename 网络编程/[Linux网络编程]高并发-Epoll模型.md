**概要介绍**

>
> epoll是Linux下多路复用IO借口select/poll的`增强版本`，它能显著提高程序在大量并发连接但是只有少数活跃的情况下的系统CPU利用率，因为他会复用文件描述符几何来传递结果。另一点原因是获取事件的时候，它无需遍历整个被监听的描述符集，只要遍历哪些被内核IO事件异步唤醒而加入Ready队列的描述符集合就行了

> `2021-09-08`  
>  重新复习了一下代码，有几个注意点区别于selec，这边的结构体是一开始创建两个，重复使用（没有清空），event也是重复使用，每次epoll_ctl
> 添加拷贝了一次。注意epoll_ctl添加 和 删除注意区别 删除只需要fd 不需要结构体。

### 文章目录

  * 1 Epoll和Select区别对比
  * 2 Epoll模型的三个函数
  *     * 2.1 函数原型+功能说明
    * 2.2 epoll_create()参数使用
    * 2.3 epoll_ctl()参数使用
    * 2.4 epoll_wait()参数使用
  * 3 Epoll模型封装成类
  * 4 Epoll简单实现打印 代码+解释
  * 总结

# 1 Epoll和Select区别对比

  1. epoll不存在集合的覆盖 epoll_create会返回一个fd，指向空间包含全部的事件（结构体）

  2. epoll把要监听的每一个fd都包装成一个事件，并把这个事件记入epollfd 让epollfd来监听

  3. select产生动静是吧fd放入集合 但是epoll通过epoll_wait 把产生动静的fd所包装好的事件放入结构体数组

  4. select需要备份，需要重新创建数组放fd循环比对，epoll直接通过包装好的事件（结构体）就能获得fd，效率也更快（差别主要体现在这）

  5. 两者的区别是的select适合用户客服端不多的情况，而epoll没有客户端的上限

# 2 Epoll模型的三个函数

## 2.1 函数原型+功能说明

    
    
    #include <sys/epoll.h>
    
    int epoll_create(int size);
    作用：创建一个epoll句柄，告诉他需要监听的数目（也可以理解成申请一片空间，用于存放监听的套接字）
    
    int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
    作用：控制某个epoll监控的文件描述符上的事件：注册，修改、删除（也就是增添 删除 修改一个事件）
    
    int epoll_wait(int epfd,struct epoll_event * events,int maxevents,int timeout)
    作用：监听红黑树上的事件，将产生动静的事件放在event这个数组内，
    

## 2.2 epoll_create()参数使用

  * `int epoll_create(int size);`
  * 参数一：通知内核监听size个fd，只是个建议值并与硬件有关系。（从 Linux 内核 2.6.8 版本起，size 这个参数就被忽略了，只要求 size 大于 0 即可）  
返回值：返回epoll句柄（fd）

## 2.3 epoll_ctl()参数使用

  * `int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；`

  * 参数一：int epfd:epoll_create()的返回值

  * 参数二：int op： 表示动作，用三个宏来表示

    * EPOLL_CTL_ADD(注册新的fd到epfd)
    * EPOLL_CTL_MOD(修改已经注册的fd监听事件)
    * EPOLL_CTL_DEL(从epfd删除一个fd)
  * 参数三：int fd 操作对象（socket）

参数四：struct epoll_evevt* evevt; 告诉内核需要监听的事件

    
    
    结构体如下：
    struct epoll_event {
    __uint32_t events; 宏定义读和写EPOLLIN读EPOLLOUT写
    epoll_data_t data; 联合体
    };
    联合体如下：
    
    typedef union epoll_data {
    void *ptr;
    int fd;
    __uint32_t u32;
    __uint64_t u64;
    } epoll_data_t;
    

  * 返回值：成功返回0,不成功返回-1

特别注意参数四的使用

## 2.4 epoll_wait()参数使用

  * `int epoll_wait(int epfd,struct epoll_event * events,int maxevents,int timeout)`

  * 参数一：int epfd:epoll_create()函数返回值

  * 参数二：struct epoll_events* events用于回传代处理事件的数组（也就是存放产生动静的事件）

  * 参数三:int maxevents 同时最多产生多少事件，告诉内核events有多大，该值必须大于0

  * 参数四:int timeout表示 -1相当于阻塞，0相当于非阻塞，超时时间(单位：毫秒)

  * 返回值：成功返回产生动静事件的个数

补充说明： 产生动静是指

  1. 有新的客户端需要连接 或者
  2. 已连接的客户端发送了信息

# 3 Epoll模型封装成类

**头文件**

    
    
    const int  MAXEPOLLSIZE = 10;
    
    class Epoll
    {
       public:
           Epoll();
           bool Add(int fd,int eventsOption);//添加事件
    
           //Returns the number of triggered events
           int Wait();//等待事件触发
    	   bool Del(int fd);
           //bool Delete(const int eventIndex);//删除事件
           int  GetEventOccurfd(const int eventIndex) const;//得到事件数组某个值的fd
           int  GetEvents(const int eventIndex) const;//得到事件数组的宏//可读可写，触发方式
    
       private:
            int epollfd;//epoll专用文件描述符
            int fdNumber;//epollfd里面客户端有多少
            
            struct epoll_event event;//事件
            struct epoll_event events[MAXEPOLLSIZE];
            struct rlimit  rt;
    
    };
    

**源文件**

    
    
    #include "Epoll.h"
    #include <stdio.h>
    #include <stdlib.h>
    #include <iostream>
    
    Epoll::Epoll():fdNumber(0)
    {
        rt.rlim_max= rt.rlim_cur = MAXEPOLLSIZE; 
        if(::setrlimit(RLIMIT_NOFILE,&rt) == -1){
            std::cout << "setlimit failed\n";
            exit(1);
        }
    
        epollfd = epoll_create(MAXEPOLLSIZE);
        //创建epoll句柄，上限为MAXEPOLLSIZE
    }
    
    bool Epoll::Add(int fd,int eventsOption)//把监听到的fd放到epoll句柄里面
    {
         event.data.fd = fd;
         event.events = eventsOption; //EPOLLIN | EPOLLET
         if(epoll_ctl(epollfd,EPOLL_CTL_ADD,fd,&event) < 0){
             return false;
         }
    
         fdNumber++;//文件描述符的数量+1
         return true;
    }
    
    
    bool Epoll::Del(int fd)
    {
    	event.data.fd = fd;
    	event.events = EPOLLIN | EPOLLET;
    	int ret = epoll_ctl(epollfd, EPOLL_CTL_DEL,fd, &event);
    	if (ret<0)
    	{
    		return false;
         }
    	close(event.data.fd);
    	printf("client is close ,fd = %d\n", event.data.fd);
    	return true;
    }
    int Epoll::Wait()
    {
        int eventNumber;//初始化事件响应数量
        eventNumber = epoll_wait(epollfd,events,fdNumber,-1);
        if(eventNumber < 0){
            std::cout << "epoll_wait  failed \n";
            exit(1);
        }
        return eventNumber;
    }
    
    
    int Epoll::GetEventOccurfd(const int eventIndex) const
    {
        return events[eventIndex].data.fd;
    }
    
    int Epoll::GetEvents(const int eventIndex) const
    {
        return events[eventIndex].events;
    }
    
    

# 4 Epoll简单实现打印 代码+解释

**服务器**

    
    
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <netinet/in.h>
    #include <arpa/inet.h>
    #include <sys/epoll.h>
    #include <errno.h>
    #include <string.h>
    #include <stdio.h>
    #include <unistd.h>
    #define MAXLINE 80
    #define SERV_PORT 8000 //端口号
    #define OPEN_MAX 1024  //最多连接数
    
    
    
    int main(int argc,char *argv[])
    {	
    	int maxi,sockfd,connfd;//sockfd 用来监听 connfd用来连接的· 监听套接字要自己创建  连接套接字等待返回就行不需要socket
    	int nready,efd; //efd是 epoll模型标识符 nready 产生的动静数
    	struct sockaddr_in clientaddr, serveraddr;
    	struct epoll_event event, events[OPEN_MAX]; //一个果篮 一个结构体
    	
        sockfd = socket(AF_INET, SOCK_STREAM, 0);
        bzero(&serveraddr, sizeof(serveraddr));
    	bzero(&clientaddr, sizeof(clientaddr));
        serveraddr.sin_family = AF_INET;
        serveraddr.sin_addr.s_addr = htonl(INADDR_ANY);
        serveraddr.sin_port = htons(SERV_PORT);
    	int on =1;
    	setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on)) ;//设置为可重复使用的端口
        bind(sockfd, (struct sockaddr *) &serveraddr, sizeof(serveraddr));
        listen(sockfd, OPEN_MAX);//服务器套接字
    	efd=epoll_create(OPEN_MAX);
    	/*包装服务器fd为事件*/
    	event.events=EPOLLIN|EPOLLET;
    	event.data.fd=sockfd;
        epoll_ctl(efd,EPOLL_CTL_ADD,sockfd,&event);//把服务器fd包装成事件放在红黑树上	
        while(1)
    	{
    		//参数一 epollfd 参数二 产生动静的结构体数组 参数三 同时最多产生多少动静 参数四 超市时间
    		nready=epoll_wait(efd,events,OPEN_MAX,-1);//返回值为动静数量
    		for(int i=0;i<nready;i++)
    		{        	
    			if(!(events[i].events & EPOLLIN))//判断为可读事件 不是立刻返回循环 与select不同
    				continue;
    			if (events[i].data.fd==sockfd)//表示有新的连接
    			{
    				int len=sizeof(clientaddr);
    				char ipstr[128];//打印用到
    				connfd=accept(sockfd,(struct sockaddr *)&clientaddr,&len);
    				printf("client ip%s ,port %d\n",inet_ntop(AF_INET,&clientaddr.sin_addr.s_addr,ipstr,sizeof(ipstr)),
    					ntohs(clientaddr.sin_port));
    				event.events = EPOLLIN|EPOLLET; 
    				event.data.fd = connfd;  
    				epoll_ctl(efd, EPOLL_CTL_ADD, connfd, &event);			
    			}
    			else//表示旧的数据产生可读事件（1 客户端发来数据 2 客户端断开链接）
    			{				
         			connfd=events[i].data.fd;
    				char buf [1024];
    				memset(buf,0,1024);
    				int nread;			nread=read(connfd,buf,sizeof(buf));
    				
    				if(nread==0)
    				{
    					printf("client is close..\n"); //打印
    					epoll_ctl(efd, EPOLL_CTL_DEL, connfd, NULL);//删除果子 select是从集合 和 数组 删除
    					close(connfd);//关闭客服端 select一样
                    }
    				else
    				{
    					printf("%s",buf);
    				}			
    			}		
    		}	
    	}	
    }
    

**客服端**

    
    
    #include <sys/types.h>
    #include <sys/socket.h>
    #include <netinet/in.h>
    #include <arpa/inet.h>
    #include <stdio.h>
    #include <string.h>
    #include <stdlib.h>
    #include <unistd.h>
    
    int main(int argc,char* argv[])
    {
    	int sfd;
    	struct sockaddr_in sfdaddr; //指定要连接服务器的结构体 ip 端口
    	int len;	
    	char wbuf[1024];	
    	//1.socket  通信用套接字，创建一个socket
    	sfd = socket(AF_INET,SOCK_STREAM,0);         	
    	char ipstr[] = "127.0.0.1";           //要连上的ip地址
    	//初始化地址
    	bzero(&sfdaddr,sizeof(sfdaddr));	
    	sfdaddr.sin_family = AF_INET;
    	sfdaddr.sin_port = htons(8000);	
    	int on =1;
    	setsockopt(sfd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on)) ;//设置为可重复使用的端口
    	inet_pton(AF_INET,ipstr,&sfdaddr.sin_addr.s_addr); //转换ip 保存到结构体内 
    	
    	//2.connect  主动连接服务器
    	connect(sfd,(struct sockaddr *)&sfdaddr,sizeof(sfdaddr));
    	memset(wbuf,0,1024);
    	printf("please input\n"); 
    	while(1)
    	{			   		  
    		fgets(wbuf, 100, stdin);//stdin 意思是键盘输入
    		write(sfd,wbuf,sizeof(wbuf));	
    		memset(wbuf,0,1024);					 
    	}	
    	close(sfd);
    	return 0;
    }
    
    

# 总结

> 如有错误，欢迎指出

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210419170705247.jpg?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

