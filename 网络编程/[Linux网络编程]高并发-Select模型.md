**概要介绍**

>
> 一般情况下，处理`socket通信`一个线程或者进程在处理读写的时候要么阻塞在那一直等要么非阻塞然后过会查看是否可读可写，这样会浪费大量的资源，假如需要对多个套接字进行处理读写那么得开很多个线程或者进程，`IO复用技术`就是解决这个问题。本节详细讲解IO复用模型
> select。  
>
> 解决1024以下客户端时使用select是很合适的，但如果链接客户端过多，select采用的是`轮询模型`，会大大降低服务器响应效率，不应在select上投入更多精力

> `2021-09-07` 复习发现上代码的错误，直接的结构是if else{for} 应该修改为 if
> for。因为可能多个文件描述符产生动静，如果是一个sockfd 和 几个
> 客户端的fd，那么要同时处理新客户端的连接和旧客户端的读事件。第二个就是注意代码的注释，注意代码中两个if（–nready）的判断

### 文章目录

  * 1 三种IO模型（重要）
  * 2 select模型函数使用
  *     * 2.1 select（）函数
    * 2.2 select（）函数配套使用的四个宏
  * 3 多个客户端连接实现简单的服务器回射打印（附详细解释！！！）
  * 4 伪代码
  * 5 select模型的缺点
  * 总结

# 1 三种IO模型（重要）

> 举个常用的例子，去超市买货，
>
>   * 如果我们要的货物没到，那我们就在超市一直等，不做其他的事情这就是阻塞IO
>   * 我们每隔一段时间去超市看看货到了没有，这就是 非阻塞IO
>   * 这次我们学聪明了，让进货员货物到了就通知我去取，这就是多路IO复用
>

**阻塞**

  * 在同步阻塞的IO模型中，在第一阶段“等待数据被写入Socket的缓冲区中”，操作系统会把当前的进程设置为阻塞状态，直到缓冲区被写入数据这个进程才被唤醒。这也就造成了一个问题，当操作系统把这个进程置为阻塞态的时候，这个进程就什么事都做不了了。

  * 例如常见的`fork`多线程进行的`read/write`

**非阻塞**

  * 为了解决“同步阻塞”进程可能无期限阻塞的情况，于是产生了“同步非阻塞”  
在这种IO模型中，如果发现这个Socket里面没有准备好的数据就返回一个错误，而不是把这个进程设置为阻塞状态。也就是说我们可以轮询这个Socket查看有无准备好的数据。

造成的问题

  1. 假设此时我们的服务器需要管理很多的IO请求，如果给每一个IO都分配一个进程/线程，自旋的等待有无数据到来，无疑是很浪费资源的。
  2. 如果我们用一个进程，轮询所有的IO请求，又会使IO的响应变得很慢，所以引入多路io复用

**多路复用**

  * 多路复用IO就是用一条线程，同时监听多个IO请求，并且在有IO请求产生的时候返回。注意，虽然我们的IO多路复用也会阻塞，但是这里的阻塞是应用层面的，也就是说在多路复用的方法上进行阻塞，而不是在操作系统层面去阻塞。

[概念参考](https://zhuanlan.zhihu.com/p/341371484)

# 2 select模型函数使用

## 2.1 select（）函数

    
    
    #include <sys/select.h>
    #include <sys/time.h>
    #include <sys/types.h>
    #include <unistd.h>
    
    int select(int nfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval *timeout);
    

**参数说明：**

  * `nfds：` 被监听的文件描述符总数，它会比文件描述符表集合中的文件描述符表的最大值大1，因为文件描述符从0开始计数

  * `readfds`：需要监听的可读事件的文件描述符集合`（如果有动静则该集合只会保留产生可读事件）看下面实例`

  * `writefds`：需要监听的可写事件的文件描述符集合

  * `exceptfds`：需要监听的异常事件的文件描述符集合

  * `timeout`：即告诉内核select等待多长时间之后就放弃等待。一般设为NULL 表示无动静就阻塞等待

  * `返回值：`表示产生动静的文件描述符个数`

**函数作用：**  
监听等待参数集合中的`两类`文件描述符产生动静，产生动静后集合中只会`剩下`产生动静的文件描述符  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814153739997.png#pic_center)

  * sockfd：专门检查有没有客服端连接的监听套接字 ------数量 1
  * connfd：已经连接成功的客服端通信套接字 ------数量 n

**实例补充说明：**

    
    
    select(maxfd+1,&rset,NULL,NULL,NULL);
    

由于参数二放入集合，参数一填入数字，其余位置皆为NULL,  
表示我们监听的是`rset文件描述符集合`是否有可读事件（动静），如果有则该集合只会保留产生可读事件（动静）的文件描述符，否则就阻塞等待。  
`可读事件分为以下三种`

  1. 新客服端连接（此时唯一`sockfd`放入集合）
  2. 旧客服端发消息过来（此时对应的`connfd`放入集合）
  3. 旧客户端断开连接（此时对应的`connfd`放入集合）

## 2.2 select（）函数配套使用的四个宏

  * `FD_ZERO(fd_set* set)`  
理解为初始化文件描述符几何 (既把所有位都设置为0)

  * `FD_SET(fd,fd_set* set)`  
理解为把文件描述符加入集合(本质为对应fd位设置为1)

  * `FD_CLR(fd,fd_set* set)`  
理解为把文件描述符从集合取出（本质为对应fd位设置为0）

  * `FD_ISSET(fd,fd_set* set)`  
理解为检测改文件描述符是否在集合（本质为对应的fd位是否设置为1）

# 3 多个客户端连接实现简单的服务器回射打印（附详细解释！！！）

**客户端代码如下**

    
    
    #include <unistd.h>
    #include <stdlib.h>
    #include <string.h>
    #include <stdio.h>
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <fcntl.h>
    #include <errno.h>
    #include <termios.h>
    #include <signal.h>
    #include <sys/types.h>          /* See NOTES */
    #include <sys/socket.h>
    #include <arpa/inet.h>
    #define SET_PORT 8000
    int main(int argc, char *argv[])
    {
        int sockfd,connfd;//监听套接字 和连接套接字
        struct sockaddr_in serveraddr;
        int i;//主要用于各种for循环的i
        int on = 1;//只有下方设置可重复性使用的端口用到
         //1.创建监听套接字
        sockfd = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP); 
        setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on)) ;//设置为可重复使用的端口
        
        //2.bind（通信需要套接字 我把我家的地址 门牌号绑上去 ip和端口）
        serveraddr.sin_family = AF_INET;
        serveraddr.sin_port = htons(SET_PORT);
        serveraddr.sin_addr.s_addr = htonl(INADDR_ANY);
        bind(sockfd, (struct sockaddr *)&serveraddr,sizeof(serveraddr));
        //3.监听 和服务器连接的总和
        listen(sockfd,128) ;  
        int maxfd = sockfd;
        //初始化两个集合 一个数组
        int client[FD_SETSIZE];//数组用于存放客服端fd 循环查询使用 FD_SETSIZE 是一个宏 默认已经最大1024
        fd_set rset;//放在select中 集合中都是有动静的fd 一般就一个
        fd_set allset;//只做添加fd
        FD_ZERO(&rset);//清空的动作
        FD_ZERO(&allset);//清空的动作
        
        //先将监听套接字放入
        FD_SET(sockfd,&allset);
        int nready;
        
        //初始化数组都为-1 应为标识符从0开始
        for(i = 0; i < FD_SETSIZE; i++)
            client[i] = -1;
        while(1)
        {
        	//非常关键的一步
            rset = allset;//保证每次循环 select能监听所有的文件描述符 因为rset只会剩下有动静的
            nready = select(maxfd+1,&rset,NULL,NULL,NULL);//参数一
            
            //新客户端
            if(FD_ISSET(sockfd,&rset))
            {
                struct sockaddr_in clientaddr;  
                memset(&clientaddr,0,sizeof(clientaddr));
                int len = sizeof(clientaddr);
                
                connfd = accept(sockfd, (struct sockaddr*)&clientaddr,&len);
                
                char ipstr[128];//打印用到
                printf("client ip%s ,port %d\n",inet_ntop(AF_INET,&clientaddr.sin_addr.s_addr,ipstr,sizeof(ipstr)),
                       ntohs(clientaddr.sin_port));
                
                //做的事情一：文件df放入数组
                for(i = 0; i < FD_SETSIZE; i++)
                {	if(client[i] < 0)
                    {
                        client[i] = connfd;
                        break;//一定要记得及时跳出 易错点
                    }
                }
                //做的事情二：放入集合
                FD_SET(connfd,&allset);//放入集合
                //防止超出范围//select的第一个参数必须是监视的文件描述符+1 如果不断有新的客户连接 最大值不断变大 超出就赋值
                if(connfd > maxfd)
                    maxfd = connfd;
                
                //下方表示 如果同一时刻只有 一个动静 就无需进入下方的else判断处理 如果不止一个 nready-1 再进入下方判断
                if(--nready <= 0)  
                	continue;
                
            }
            else
            {
                //已连接FD产生可读事件
                for(i = 0; i < FD_SETSIZE; i++)//FD_SEISIZE 是宏 1024  //循环从数组取出元素比对
                {		
                    
                    if(FD_ISSET(client[i],&rset))
                    {
                        connfd = client[i];
                        char buf[1024] = {0};
                        int nread ;
                        nread = read(connfd, buf, sizeof(buf));
                        if(nread == 0)//表示客服端断开链接
                        {
                            //四步处理 打印说明 从集合中删除  从数组中删除 关闭客服端
                            printf("client is close..\n");
                            FD_CLR(connfd, &allset);
                            client[i] = -1;
                            close(connfd);
                        }
                        else//正常读到处理
                        {
                            write(connfd,buf,nread);
                            memset(buf,0,1024); 
                            
                        }
                       //下方表示如果同意时刻如果可读事件只有一个 无需再将数组元素进行循环比对 直接跳出
                        //不必让循环走完 浪费时间
                        if(--nready <= 0)
                            break;
                    }
                }
            }
        }    
        return 0;
    }
    
    

**服务器代码如下**

    
    
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
    	int sockfd;
    	struct sockaddr_in sfdaddr; //指定要连接服务器的结构体 ip 端口
    	int len;
    	//char buf[1024];
    	char wbuf[1024];
    	char rbuf[1024];
    	//1.socket  通信用套接字，创建一个socket
    	sockfd = socket(AF_INET,SOCK_STREAM,0);         
    	
    	char ipstr[] = "127.0.0.1";           //本机测试ip
    	//初始化地址
    	bzero(&sfdaddr,sizeof(sfdaddr));
    	
    	sfdaddr.sin_family = AF_INET;
    	sfdaddr.sin_port = htons(8000);
    	
    	inet_pton(AF_INET,ipstr,&sfdaddr.sin_addr.s_addr); //转换ip 保存到结构体内 
    	
    //2.connect  主动连接服务器
    	connect(sockfd,(struct sockaddr *)&sfdaddr,sizeof(sfdaddr));	
    	//3.读写
    #if 1
    	
        while(1)
    	{        
            memset(wbuf,0,1024);
            memset(rbuf,0,1024);
            scanf("%s",wbuf);
            write(sockfd,wbuf,strlen(wbuf));
    		len=read(sockfd,rbuf,sizeof(rbuf));
    		//write(STDOUT_FILENO,buf,len);
            printf("%s\n",rbuf);		
    	}
    	
    #endif	
    	//4.close
    	close(sockfd);
    	return 0;
    }
    

**效果图**  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814164353801.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70#pic_center)

# 4 伪代码

    
    
    #include<stdio.h>
    int main()
    {
    	int sockfd=sockfd();
    	bind(socket);
    	listen(socket);	
    	fd_set rest;
    	fd_set allset;//allset 只做添加fd 不在select进行改变
    	int allfd[1024]={0};//把客服端同时也放入数组 
    	int count =0;
    		
    	//把socket（把服务器文件描述符）放入集合
          FD_SET(socket,&rset);
    	FD_SET(socket,&allset);
    	while(1)
    	{
    		reset=allset;//保证每次循环 select能监听所有的文件描述符
    		//在这里等 等rset里面有动静
    		select(100,&rest,NULL,NULL,NULL); 
    		if(FD_ISSET(socket,&rset))   //可能1 产生新链接
    		{
    			int confd=accept(socket);//accept你做等待
    			//connfd 是用来通信 read write
    			FD_SET(confd,&allset);
    			allfd[count]=connfd;
    			count++;
    		}
    		else                         //可能2 旧客服端发来消息
    		{
    			for(int i=0;i<=count;i++)//把数组元素循环取出集合是否存在
    			{
    				if(FD_ISSET(allfd[i],&rset))
    				{
    					if conn=allfd[i];//找到了产生动静的fd
    					break；
    						
    				}
    			}			
    		}		
    	}		
    }
    

# 5 select模型的缺点

  1. 最大并发数限制，因为一个进程所打开的 fd（文件描述符）是有限制的，由 FD_SETSIZE 设置，默认值是 1024，并且集合描述符最大也只能为1024，因此 select 模型的最大并发数就被相应限制了。

  2. 效率问题，采用循环的方式匹配数组内的fd是否在产生的动静集合中，如果连接的客户端数量很多，那么效率可想而知。

  3. 内核 / 用户空间 内存拷贝问题，如何让内核把 FD 消息通知给用户空间呢？在这个问题上 select 采取了内存拷贝方法，在FD非常多的时候，非常的耗费时间。  
[第三点参考链接](https://www.zhihu.com/search?type=content&q=select%E6%A8%A1%E5%9E%8B)  
`总结：`前面两点缺陷在代码中都有体现。

# 总结

> `结束语：`凡心所向，素履所往，生如逆旅，一苇以航。

> 如有错误，欢迎指出批评…

