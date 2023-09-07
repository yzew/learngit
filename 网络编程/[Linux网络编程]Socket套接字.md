> socket 的原意是“插座”，在计算机通信领域，socket 被翻译为“套接字”，它是计算机之间进行通信的一种约定或一种方式。 
>  通过 socket 这种约定，一台计算机可以接收其他计算机的数据，也可以向其他计算机发送数据。  
>  这个数据报格式套接字可以达到高质量的数据传输。这是因为它使用了 TCP 协议

> 09-05补充  
>  1 网络字节序 主机字节序 和点分十进制之间的转换  
>  2 长连接和短链接  
>  3 三次握手 注意同步确认序号 或者说请求和应答（总而言之三件事 同步 协商 确认）以及SYN ACK  
>  [之前写的参考1](https://blog.csdn.net/weixin_44972997/article/details/107694504)  
>  [三次握手2 ](https://blog.csdn.net/weixin_44972997/article/details/116228437)  
>
> [listen第二个参数数字的含义[随手笔记]](https://blog.csdn.net/weixin_44972997/article/details/120144100)  
>
> [为什么是三次握手和四次挥手[随手笔记]](https://blog.csdn.net/weixin_44972997/article/details/120143055)

### 文章目录

  * 1 套接字的概念
  * 2 通信方式有哪几种，Socket有什么区别
  * 3 如何利用套接字进行读写交流
  * 4 Socket模式流程图
  * 5 网络套接字函数详解
  *     * 5.1 Socket( )函数
    * 5.2 bind( )函数
    * 5.3 listen( )函数
    * 5.4 accept( )函数
    * 5.5 connect( )函数
  * 6 代码实例+详细注释（重）
  *     * 6.1 实现客户端与服务器消息互发
    * 6.2 回射服务器
    *       * 6.2.1 前提须知
  * 补充-为什么客户端不用bind绑定
  * 总结

# 1 套接字的概念

Linux当中的一种文件类型，伪文件，不占用存储空间，可进行IO操作，可间接看做文件描述符使用。

# 2 通信方式有哪几种，Socket有什么区别

**通信方式** ：信号量 管道 消息队列 共享内存 套接字  
**区别** ：套接字支持网络上两台以上的设备进行通信，其他其中只能在一台设备上  
**原因：** Socket有双个缓冲区

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730180122614.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

# 3 如何利用套接字进行读写交流

  1. 服务器通过accept函数返回值可以获得客服端的套接字，我们就可以对客服端进行IO(读写)操作
  2. 客服端通过connect函数的第一个参数绑定服务器，我们就可以对服务器进行IO(读写)操作
  3. 将套接字看做文件描述符使用，更好理解


​    
​    int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
​    int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);


# 4 Socket模式流程图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210416111458985.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

# 5 网络套接字函数详解

> Linux查看手册 `man 2 函数名称`

## 5.1 Socket()函数

**作用** :  
用于服务器和客户端，创建套接字，返回一个可操作的文件描述符  
**参数使用：**

  * `int socket（int domain,int type,int protocal）;`

  * 参数一：表示ip地址类型，常用的有两种

    * 其中`AF_INET`表示IPv4地址，比如127.0.0.1，这是一个本机测试ip
    * 其中`AF_INET6`表示IPv6地址，比如2001:3CA1:10F:1A:121B:0:0:10
  * 参数二：表示数据传输方式/套接字类型，常见两种

    * `SOCK_STREAM`（流格式套接字/面向连接的套接字）
    * `SOCK_DGRAM` （数据报套接字/无连接的套接字）
  * 参数三：表示传输协议，理论上前两个参数已经可以推演出采用哪种协议主要是为了解决，两种不同的协议支持同一种地址类型和数据型。如果我们不指明使用哪种协议，操作系统是没办法自动推演的。如果两种情况只有一个协议满足条件，可以将`protocol` 的值设为 `0`，系统自动推演出采用哪种协议

  * 返回值：返回一个套接字（文件描述符fd）

## 5.2 bind( )函数

**作用：**  
用于服务器，给sockfd套接字绑上本机地址和使用端口，确定了服务器的身份  
**参数使用：**

  * `Int bind（int sockfd，const struct sockaddr*addr,socklen_t addrlen）；`
  * 参数一：套接字的fd（文件描述符），socket（）函数的返回值
  * 参数二：结构体 ip+port（端口）

    
    
    struct sockaddr_in{ （涉及强制转换sockaddr_in ->sockaddr  参考）
    	short int sin_family;              //地址族
    	unsigned short int sin_port;       //端口号
    	struct in_addr sin_addr;           //IP地址
    }
    struct in_addr {
         __be32 s_addr;
    };
    
  * 参数三：结构体的字节长度
  * 返回值：判断成功失败

## 5.3 listen( )函数

**函数作用：**  
用于服务器，使socket处于监听模式，监听时候有客户端连接，并放入队列(也可以说，设置同时与服务器建立连接的上限)（同时进行3次握手连接的客户端数量）  
**参数使用：**

  * `int listen（int sockfd，int backlog）;`
  * 参数一：bind绑定ip和端口的套接字
  * 参数二：请求链接客户端队列的最大存放数目
  * 返回值：判断成功失败

## 5.4 accept( )函数

**函数作用：**  
用于服务器，接收一个客户端的连接请求，并返回连接客户端的套接字便于IO操作，如果没有客户连接会阻塞等待。  
**参数使用：**

  * `int accept(int sockfd,struct sockaddr*addr,socklen_t*addrlen)`
  * 参数一：服务器的套接字（也叫监听套接字），表明了自己的身份
  * 参数二：传出参数，跟我建立连接的客户端的结构体（内含客户端ip+端口）
  * 参数三: 结构体长度的指针 &sizeof()
  * 返回值：连接客户端的套接字

## 5.5 connect( )函数

**函数作用：**  
用于客户端，函数可以和自动与远端服务器建立连接  
**参数使用：**

  * `int connect(int sockfd,struct sockaddr*serv_addr,int addrlen)`
  * 参数一：传入参数，文件描述符绑定连接成功的服务器套接字便于在客户端对服务器进行IO操作
  * 参数二：绑定我要链接服务器的结构体（需要初始化绑上ip和断口），表明目的
  * 参数三：结构体的长度

# 6 代码实例+详细注释（重）

> 代码部分要求是能够自主写出，不查看，比较重要。

## 6.1 实现客户端与服务器消息互发

**服务器**


​    
​    #include <sys/types.h>
​    #include <sys/socket.h>
​    #include <netinet/in.h>
​    #include <arpa/inet.h>
​    #include <stdio.h>
​    #include <string.h>
​    #include <stdlib.h>
​    #include <unistd.h>
​    #define SER_PORT 8000
​    int main(void)
​    {
​    	int sockfd,connfd;//
​    	int len;
​    	char wbuf[1024];
​    	char rbuf[1024];
​    	struct sockaddr_in serveraddr,clientaddr; //两个结构体 一个用于绑定身份到套接字  一个用于接收客服端的结构体
​    	//1.创建监听套接字
​    	sockfd = socket(AF_INET,SOCK_STREAM,0);
​    	//2.bind（通信需要套接字 把家的地址 门牌号绑上去 ip和端口）
​    	bzero(&serveraddr,sizeof(serveraddr)); //类似memset 清空结构体
​    	//地址族协议，选择IPV4
​    	serveraddr.sin_family = AF_INET;     //属于ipv4还是ipv6
​    	//IP地址 本机任意可用ip地址
​    	serveraddr.sin_addr.s_addr = htonl(INADDR_ANY);
​    	serveraddr.sin_port = htons(SER_PORT);//端口号	
​    	bind(sockfd,(struct sockaddr *)&serveraddr,sizeof(serveraddr));
​    	//3.监听 和服务器连接的总和
​    	listen(sockfd,128);	
​    	int size = sizeof(clientaddr);
​    	//4.accept 阻塞监听 客服端链接的请求
​    	connfd = accept(sockfd,(struct sockaddr *)&clientaddr,&size); 	
​    	//输出客服端的ip和端口
​    	char ipstr[128];
​    	printf("client ip%s ,port %d\n",inet_ntop(AF_INET,&clientaddr.sin_addr.s_addr,ipstr,sizeof(ipstr)),
​    		ntohs(clientaddr.sin_port));
​    	//5.处理客户端请求
​    	//读和写
​    	while(1)
​    	{
​    		memset(wbuf,0,sizeof(wbuf));//清空
​    		memset(rbuf,0,sizeof(wbuf));	
​    		//接收消息	
​    		int len = read(connfd,rbuf,sizeof(rbuf));
​    		if(len==0)//表示断开连接
​    		{
​    			printf("client is close....\n");
​    		}
​    		printf("receive from client:%s",rbuf);
​    		//发送消息
​    		printf("send to client:");
​    		fgets(wbuf,sizeof(wbuf),stdin);
​    		write(connfd,wbuf,strlen(wbuf));	
​    	}
​    	close(connfd);
​    	close(sockfd);
​    	return 0;
​    }


**客户端**


​    
​    #include <sys/types.h>          /* See NOTES */
​    #include <sys/socket.h>
​    #include <netinet/in.h>
​    #include <arpa/inet.h>
​    #include <stdio.h>
​    #include <string.h>
​    #include <stdlib.h>
​    #include <unistd.h>
​    #define SER_PORT 8000
​    int main(void)
​    {
​    	int sockfd;
​    	struct sockaddr_in serveraddr;
​    	int len;
​    	char wbuf[1024],rbuf[1024];
​    	//1、socket 通信用套接字，创建一个sockfd
​    	sockfd = socket(AF_INET,SOCK_STREAM,0);
​        char ipstr[]="127.0.0.1";
​    	//2、编辑要连接的服务器地址，并绑定
​    	bzero(&serveraddr,sizeof(serveraddr));
​    	serveraddr.sin_family = AF_INET;            //设置地址族协议
​    	serveraddr.sin_port = htons(SER_PORT);      //设置端口号
​    	inet_pton(AF_INET,ipstr,&serveraddr.sin_addr.s_addr);//设置ip地址   点分十进制转成网络字节序
​    	//2、connect 连接服务器 sockfd传出服务器套接字
​    	connect(sockfd,(struct sockaddr*)&serveraddr,sizeof(serveraddr));
​    	//3、读写
​    	while(1)
​    	{
​    		memset(wbuf,0,sizeof(wbuf));
​    		memset(rbuf,0,sizeof(rbuf));
​    		//发送消息
​    		printf("send to server:");
​    		fgets(wbuf,sizeof(wbuf),stdin);
​    		write(sockfd,wbuf,strlen(wbuf));
​    		
    		//接收消息
    		len=read(sockfd,rbuf,sizeof(rbuf));
    		if(len==0)//表示断开连接
    		{
    			printf("server is close....\n");
    		}
    		printf("receive from server:%s",rbuf);


​    		
​    	}
​    	//4、close
​    	close(sockfd);
​    	return 0;
​    }


**显示结果**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210416110527872.png)

## 6.2 回射服务器

### 6.2.1 前提须知

  * 简单的说就是即从客户端收到什么数据，就发送什么数据回去

**前提须知** ：read读取不到信息会阻塞等待！  
**执行过程** ： 1 -》2-》5-》6-》3-》4


​    
​             /* 客服端部分 */
​          1  scanf("%s",wbuf);//等待键盘输入 
​          2  write(sfd,wbuf,strlen(wbuf));//写入服务器
​    	  3  read(sfd,rbuf,sizeof(rbuf));//等待客服端写会
​          4  printf("%s\n",rbuf);//打印内容
​            ----------------------分割线------------------------------
​            /*服务器部分*/
​          5  read(confd,buf,sizeof(buf)); //阻塞等待
​    	  6  write(confd,buf,len); //写会客服端


​    

**流程**

  * 首先客服端与服务器建立连接
  * 客服端等待键盘输入（scanf 或者 fget）
  * 如果此时键盘输入回车确认 客服端write写入服务器 服务器read读到了信息
  * 服务器再将read读到的信息write写回客服端 客服端read接受到信息并打印

**服务器**


​    
​    #include <sys/types.h>
​    #include <sys/socket.h>
​    #include <netinet/in.h>
​    #include <arpa/inet.h>
​    #include <stdio.h>
​    #include <string.h>
​    #include <stdlib.h>
​    #include <unistd.h>
​    
    int main(void)
    {
    	int sockfd,confd;
    	char ipstr[128];
    	int size;
    	char buf[1024];
    	int i;
    	int len;
    	//两个结构体 一个用于绑定身份到套接字  一个用于接收客服端的结构体
    	struct sockaddr_in serveraddr,clientaddr; 
    	//1.创建监听套接字
    	sockfd = socket(AF_INET,SOCK_STREAM,0);
    	//2.bind（通信需要套接字 我把我家的地址 门牌号绑上去 ip和端口）
    	bzero(&serveraddr,sizeof(serveraddr)); //类似memset 清空结构体
    	//地址族协议，选择IPV4
    	serveraddr.sin_family = AF_INET;     //属于ipv4还是ipv6
    	//IP地址 本机任意可用ip地址
    	serveraddr.sin_addr.s_addr = htonl(INADDR_ANY);
    	serveraddr.sin_port = htons(8002);//端口号
    	
    	bind(sockfd,(struct sockaddr *)&serveraddr,sizeof(serveraddr));
    	
    	//3.监听 128为服务器连接的总和
    	listen(sockfd,128);
    	
    	size = sizeof(clientaddr);
    	//4.accept 阻塞监听 客服端链接的请求
    	//参数二结构体的转换
    	confd = accept(sockfd,(struct sockaddr *)&clientaddr,&size); 
    	
    	//输出客服端的ip和端口
    	printf("client ip%s ,port %d\n",inet_ntop(AF_INET,&clientaddr.sin_addr.s_addr,ipstr,sizeof(ipstr)),
    		ntohs(clientaddr.sin_port));
    	
    	//5.处理客户端请求
    	//读和谐
         
    	while(1)
    	{
    		len=read(confd,buf,sizeof(buf)); //接受不到 阻塞等待
    	#if 0  //if 0 endif   之间的大小写转换代码已经屏蔽
    		i=0;
    		while(i<len)
    		{
    			buf[i]=toupper(buf[i]);//小写转大写的操作
    			i++;	
    		}
    	#endif
    		write(confd,buf,len);
    		memset(buf,0,1024);
    		
    	}
    	close(confd);
    	close(sockfd);
    	return 0;
    }


**客户端**


​    
​    #include <sys/types.h>
​    #include <sys/socket.h>
​    #include <netinet/in.h>
​    #include <arpa/inet.h>
​    #include <stdio.h>
​    #include <string.h>
​    #include <stdlib.h>
​    #include <unistd.h>
​    int main(int argc,char* argv[])
​    {
​    	int sfd;
​    	struct sockaddr_in sfdaddr; //指定要连接服务器的结构体 ip 端口
​    	int len;
​    	//char buf[1024];
​    	char wbuf[1024];
​    	char rbuf[1024];
​    	//1.socket  通信用套接字，创建一个socket
​    	sfd = socket(AF_INET,SOCK_STREAM,0);         
​    	char ipstr[] = "127.0.0.1";                  //或者本机测试ip
​    	//char ipstr[] = "192.168.3.106";           //要连上的ip地址
​    	//初始化地址
​    	bzero(&sfdaddr,sizeof(sfdaddr));
​    	
    	sfdaddr.sin_family = AF_INET;
    	sfdaddr.sin_port = htons(8002);
    	
    	inet_pton(AF_INET,ipstr,&sfdaddr.sin_addr.s_addr); //转换ip 保存到结构体内 
    	
    	//2.connect  主动连接服务器  sfd返回客服端的套接字（文件描述符）
    	connect(sfd,(struct sockaddr *)&sfdaddr,sizeof(sfdaddr));	
    	//3.读写	
        while(1)
    	{        
            memset(wbuf,0,1024);
            memset(rbuf,0,1024);
            scanf("%s",wbuf);
            write(sfd,wbuf,strlen(wbuf));
    		len=read(sfd,rbuf,sizeof(rbuf));
    		//write(STDOUT_FILENO,buf,len);
            printf("%s\n",rbuf);
    			
    	}	
    	//4.close
    	close(sfd);
    	
    	return 0;
    }


**执行结果**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730123912262.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

# 补充-为什么客户端不用bind绑定

  * 如果不适用bind绑定客户端地址结果，采用“隐式绑定”。系统自动分配
  * 可以bind 也可以 不 bind 都是可以的

# 总结

> 如有错误，欢迎指出。

> 看到一篇概念问题解释的比较详细的文章，感兴趣的可以看看下面链接。文中比较关注具体的实现，概念内容放的比较少。  
>  [文章链接](https://zhuanlan.zhihu.com/p/230800627?utm_source=wechat_session)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406091722343.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

