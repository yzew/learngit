>
> 线程池在实际的服务器开发是非常重要的一环，他涉及的概念也比较多，例如线程的使用，互斥锁，条件变量，信号量的创建使用时机等等。同时你还要知道它如何自动销毁和创建，实现一个较为智能的模式。  
>  本文对线程池的一种构建方式进行详细分解解读注释，但也有许多需要改进的地方。  
>  完整的代码文中已经给出，如需整个测试项目，私信发。  
>  [目录链接](https://blog.csdn.net/weixin_44972997/article/details/115869213)  
>  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210424132009230.png?x-oss-
> process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

> `2021-09-08`：
>
>   * 可以思考几个问题，为什么创建线程池，内部设计。
>   *
> 可以想象成一个链表放包装好的函数叫做任务，许多独立的线程在等待（有时间限制，超过时间就退出线程了），不断判断有没有任务，如果有了就去取。有一个添加的函数，专门把任务挂起来，然后看看有没有空闲的线程，如果有啥事不用做，没有就创建线程，执行`thread_routine`这个不断取任务函数执行。当然也有`第二种策略`，线程不要超时等待退出和循环判断有没有任务，就一直wait阻塞等待，如果执行添加的函数，就去看看有没有空闲的线程，如果有就去唤醒一个线程执行任务。
>

### 文章目录

  * 1 为什么要epoll创建一个线程池
  * 2 线程池的实现流程
  * 3 准备工作，封装互斥锁和条件变量
  * 4 线程池的实现
  *     * 4.1 结构说明(重)
    *       * 4.1.1 任务结构体
      * 4.1.2 线程池结构体
      * 4.1.3 四个函数
    * 4.2 threadpool_add_task函数实现说明
    *       * 4.2.1 伪代码（详细中文说明）
      * 4.2.2 具体代码实现
    * 4.3 thread_routine函数实现说明
    *       * 4.3.1 伪代码（详细中文说明）
      * 4.3.2 具体代码实现
    * 4.4 threadpool_init函数实现说明
    * 4.5 threadpool_destroy函数实现说明
  * 5 测试代码
  *     * 5.1 main.c
    * 5.2 client.c
    * 5.3 测试结果
  * 补充
  * 总结

# 1 为什么要epoll创建一个线程池

  1. 降低资源消耗。通过重复利用已创建的线程降低线程的创建和销毁造成的消耗。
  2. 提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。
  3. 提高线程的可管理性。线程为稀缺资源，如果无限制地创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一分配、调优和监控。但是，要做到合理利用线程池，必须对其实现原理了如指掌。

# 2 线程池的实现流程

  1. 创建一个线程池，初始化其中属性，可创建一定数量线程，放入队列，或者不初始化
  2. 线程都处于阻塞等待状态，不占用cpu，当超时等待，可以自己结束线程
  3. 当需要执行函数，则把函数包装成任务，并把任务传入空闲线程进行运行
  4. 当没有空闲进程且小于最大线程数要求，则创建线程执行任务
  5. 执行完任务的线程不需要退出，阻塞等待即可（或者设置等待时间）
  6. 若有线程池销毁通知，确保任务执行完退出销毁

# 3 准备工作，封装互斥锁和条件变量

> 其实这边的封装就是为了方便使用，我们把信号量和互斥锁封装成一个结构体是有很大帮助的。  
>  在实际使用中我们都知道实现互斥访问（不懂可以参考`生产者和消费者模型`）其中一种方式就是条件变量和互斥锁的组合使用。  
>  多个线程可以理解成`多个消费者`。所以设计到很多共享资源，需要互斥锁。而条件变量是为了方便通知线程有可以执行的任务了。

**condition.h**

    
    
    #ifndef _CONDITION_H_
    #define _CONDITION_H_
    
    #include <pthread.h>
    
    //结构体内放了一个锁和条件变量
    typedef struct condition
    {
    	pthread_mutex_t pmutex;
    	pthread_cond_t pcond;
    } condition_t;
    //初始化锁和条件变量
    int condition_init(condition_t *cond);
    //上锁
    int condition_lock(condition_t *cond);
    //解锁
    int condition_unlock(condition_t *cond);
    //阻塞等待唤醒
    int condition_wait(condition_t *cond);
    //设置时间等待
    int condition_timedwait(condition_t *cond, const struct timespec *abstime);
    //随机唤醒一个阻塞的
    int condition_signal(condition_t *cond);
    //广播
    int condition_broadcast(condition_t *cond);
    //销毁条件变量和互斥锁
    int condition_destroy(condition_t *cond);
    
    #endif /* _CONDITION_H_ */
    
    

**# condition.c**

    
    
    #include "condition.h"  
    
    int condition_init(condition_t *cond)
    {
    	int status;
    	if ((status = pthread_mutex_init(&cond->pmutex, NULL)))
    		return status;
    
    	if ((status = pthread_cond_init(&cond->pcond, NULL)))
    		return status;
    
    	return 0;
    }
    
    int condition_lock(condition_t *cond)
    {
    	return pthread_mutex_lock(&cond->pmutex); 
    }
    
    int condition_unlock(condition_t *cond)
    {
    	return pthread_mutex_unlock(&cond->pmutex);
    }
    
    int condition_wait(condition_t *cond)
    {
    	return pthread_cond_wait(&cond->pcond, &cond->pmutex);
    }
    
    int condition_timedwait(condition_t *cond, const struct timespec *abstime)
    {
    	return pthread_cond_timedwait(&cond->pcond, &cond->pmutex, abstime);
    }
    
    int condition_signal(condition_t *cond)
    {
    	return pthread_cond_signal(&cond->pcond);
    }
    
    int condition_broadcast(condition_t* cond)
    {
    	return pthread_cond_broadcast(&cond->pcond);
    }
    
    int condition_destroy(condition_t* cond)
    {
    	int status;
    	if ((status = pthread_mutex_destroy(&cond->pmutex)))
    		return status;
    
    	if ((status = pthread_cond_destroy(&cond->pcond)))
    		return status;
    
    	return 0;
    }
    
    

# 4 线程池的实现

## 4.1 结构说明(重)

**线程池主要组成就三个部分**

  * task_t类型的结构体
  * threadpool_t类型结构体
  * 四个实现函数

### 4.1.1 任务结构体

> 任务结构体简单的说就是对实际执行函数的封装

> 特别注意一下第一个成员变量，其实就是`函数指针`，函数指针
> 的本质是一个指针，该指针的地址指向了一个函数，所以它是指向函数的指针。简单的说：理解成一个未初始化的`函数变量`

> 把函数指针和参数分开作为参数其实就是为了在调用pthread_create方便传入参数。第三个成员和链表的下一个指针域一个意思。我们这边吧任务串成链表。
    
    
    typedef struct task
    {
    	// 任务回调函数
    	// 简单的说就是返回值为空指针的函数指针
    	void *(*run)(void *arg);	
    	// 回调函数参数
    	void *arg;					
    	struct task *next;//指向的下一个结构体指针
    } task_t;
    

### 4.1.2 线程池结构体

> 定义那么多关于线程的变量其实都是为了方便我们去管理线程。  
>  唯一需要注意的是任务队列的`头指针和尾指针`。因为定义了这个，我们可以通过线程池对象成员很方便的放入或者取出任务。重要
    
    
    typedef struct threadpool
    {
    	condition_t ready;		//初始化了一个条件变量和互斥锁
    	task_t *first;			//任务队列头指针
    	task_t *last;			//任务队列尾指针
    	int counter;			//线程池中当前线程数
    	int idle;				//线程池中当前正在等待任务的线程数
    	int max_threads;		//线程池中最大允许的线程数
    	int quit;				//销毁线程池的时候置1
    } threadpool_t;
    

### 4.1.3 四个函数

> 大致进行了一个介绍，具体解释说明看后面。
    
    
    // 初始化线程池
    void threadpool_init(threadpool_t *pool, int threads);
    // 往线程池中添加任务，同时唤醒线程（或者创造线程）去执行
    // 具体的说应该是把函数包装成任务 把任务放到链表 然后让线程去取出来任务执行
    void threadpool_add_task(threadpool_t *pool, void *(*run)(void *arg), void *arg);
    // 销毁线程池
    void threadpool_destroy(threadpool_t *pool);
    //线程所执行的那个函数 （注意：并不是任务里面的那个函数）
    //它的任务就是去判断有没有需要执行的任务，有的话就取出执行，否则等待or退出
    void *thread_routine(void *arg);
    
    

## 4.2 threadpool_add_task函数实现说明

  * `void threadpool_add_task(threadpool_t *pool, void *(*run)(void *arg), void *arg);`

**作用** ：  
往线程池中添加任务（封装的函数）（其实就是任务链表中添加任务（函数包装成的任务），并且唤醒等待线程去执行 如果没有空闲的线程并且小于限制的最大线程数
就去创建线程）。这里的任务队列的头部指针和尾部指针在线程池都有定义 挂上去就对了。补充一点：这边的线程不再是简简单单执行一个任务 而是可以循环等待取出执行

  * 参数一 线程池对象
  * 参数二 线程执行的具体函数
  * 参数二 具体函数的参数

> 对照伪代码和具体代码看

### 4.2.1 伪代码（详细中文说明）

    
    
    void threadpool_add_task(threadpool_t *pool, void *(*run)(void *arg), void *arg);
    /*
    {
    malloc一个任务空间把传入的执行的函数 和 参数挂上去
    上锁
    if(如果头指针为空)
    {
     任务挂头
    }
       
    else
    {
        否则挂到尾巴的下一个   
        并把当前任务作为链表的最后一个
    }
    if(有空闲的线程)
    {
        就去唤醒一个
    }
    else if（没有空闲线程 并且存货线程数不多于最大值）
    {
        这部分有很大的改进空间，每次创建一个没必要
        创建一个线程，执行thread_routine
        这边的thread_routine就是自己去取任务执行
        线程数增加
    }
    解锁
    }
    */
    

### 4.2.2 具体代码实现

    
    
    // 往线程池中添加任务
    void threadpool_add_task(threadpool_t *pool, void *(*run)(void *arg), void *arg)
    {
    	// 生成新任务
    	task_t *newtask = (task_t *)malloc(sizeof(task_t));
    	newtask->run = run;
    	newtask->arg = arg;
    	newtask->next = NULL;
    
    	condition_lock(&pool->ready);
    	// 将任务添加到队列
    	if (pool->first == NULL)
    		pool->first = newtask;
    	else
    		pool->last->next = newtask;
    	pool->last = newtask;
    
    	// 如果有等待线程，则唤醒其中一个
    	if (pool->idle > 0)
    		condition_signal(&pool->ready);
    	else if (pool->counter < pool->max_threads)
    	{
    		// 没有等待线程，并且当前线程数不超过最大线程数，则创建一个新线程
    		pthread_t tid;
    		pthread_create(&tid, NULL, thread_routine, pool);
    		pool->counter++;
    	}
    	condition_unlock(&pool->ready);
    }
    
    

## 4.3 thread_routine函数实现说明

  * `void *thread_routine(void *arg);`

**作用** ：  
从链表取出（任务）（结构体）（函数）进行执行

  * 参数一 线程池对象

> 对照伪代码和具体代码看

### 4.3.1 伪代码（详细中文说明）

    
    
    void *thread_routine(void *arg);
    //线程调用的那个函数 
    //作用就是：从链表取出（任务）（结构体）（函数）进行执行
    //参数就是 传入的线程对象 因为需要线程池对象的互斥锁 条件变量 任务指针 这些东西
    //具体 while(1)大循环 
    /*
    while(1)
    {
        上锁
        while(如果没有任务 并且线程池不要求销毁)
        {
            设置时间等待
            超时退出，但是只是退出第一层，所以这边还设置了timeout标志 准备二次退出
    
        }
        if（有任务）
        {
            我们就从任务头指针去取任务,更改头指针
            解锁 
            释放结构体空间 
            加锁 （这边加锁 是应为下面有涉及线程数的加减操作）
        }
        if(没有任务了 并且 线程池要求销毁)
        {
            线程数--
            if（线程数==0）也就是说其他线程任务都结束了 那就
            {
                唤醒在等待摧毁的函数
                跳出循环
            }
            释放锁
        }
        //这一个和第二个while是关联的
        if(超时了就是从第二个while跳出了的 并且 依旧没有任务了)
        {
            线程数减少
            释放锁
        }
    }
    */
    
    

### 4.3.2 具体代码实现

    
    
    void *thread_routine(void *arg)
    {
    	struct timespec abstime;
    	int timeout;
    	printf("thread 0x%x is starting\n", (int)pthread_self());
    	threadpool_t *pool = (threadpool_t *)arg;
    	while (1)
    	{
    		timeout = 0;
    		condition_lock(&pool->ready);
    		pool->idle++;
    		// 等待队列有任务到来或者线程池销毁通知
    		while (pool->first == NULL && !pool->quit)
    		{
    			printf("thread 0x%x is waiting\n", (int)pthread_self());
    			//condition_wait(&pool->ready);
    			clock_gettime(CLOCK_REALTIME, &abstime);
    			abstime.tv_sec += 2;
    			int status = condition_timedwait(&pool->ready, &abstime);
    			if (status == ETIMEDOUT)
    			{
    				printf("thread 0x%x is wait timed out\n", (int)pthread_self());
    				timeout = 1;
    				break;
    			}
    		}
    
    		// 等待到条件，处于工作状态
    		pool->idle--;
    
    		// 等待到任务
    		if (pool->first != NULL)
    		{
    			// 从队头取出任务
    			task_t *t = pool->first;
    			pool->first = t->next;
    			// 执行任务需要一定的时间，所以要先解锁，以便生产者进程
    			// 能够往队列中添加任务，其它消费者线程能够进入等待任务
    			condition_unlock(&pool->ready);
    			t->run(t->arg);
    			free(t);
    			condition_lock(&pool->ready);
    		}
    		// 如果等待到线程池销毁通知, 且任务都执行完毕
    		if (pool->quit && pool->first == NULL)
    		{
    			pool->counter--;
    			if (pool->counter == 0)
    				condition_signal(&pool->ready);
    
    			condition_unlock(&pool->ready);
    			// 跳出循环之前要记得解锁
    			break;
    		}
    
    		if (timeout && pool->first == NULL)
    		{
    			pool->counter--;
    			condition_unlock(&pool->ready);
    			// 跳出循环之前要记得解锁
    			break;
    		}
    		condition_unlock(&pool->ready);
    	}
    
    	printf("thread 0x%x is exting\n", (int)pthread_self());
    	return NULL;
    
    }
    
    

> 一开始弄错的是线程执行的函数和实际我们要执行的函数不一样，可以理解成一个嵌套。

## 4.4 threadpool_init函数实现说明

    
    
    void threadpool_init(threadpool_t *pool, int threads)
    {
    	// 对线程池中的各个字段初始化
    	condition_init(&pool->ready);
    	pool->first = NULL;
    	pool->last = NULL;
    	pool->counter = 0;
    	pool->idle = 0;
    	pool->max_threads = threads;
    	pool->quit = 0;
    }
    

## 4.5 threadpool_destroy函数实现说明

> 简单的说就是广播通知所有线程执行完任务，然后线程退出
    
    
    // 销毁线程池
    void threadpool_destroy(threadpool_t *pool)
    {
    	if (pool->quit)
    	{
    		return;
    	}
    	condition_lock(&pool->ready);
    	pool->quit = 1;
    	if (pool->counter > 0)
    	{
    		if (pool->idle > 0)
    			condition_broadcast(&pool->ready);
    
    		// 处于执行任务状态中的线程，不会收到广播
    		// 线程池需要等待执行任务状态中的线程全部退出
    
    		while (pool->counter > 0)
    			condition_wait(&pool->ready);
    	}
    	condition_unlock(&pool->ready);
    	condition_destroy(&pool->ready);
    }
    
    

# 5 测试代码

> 这边通信方式用的是共享内存，设计了一个简单的结构体。test
> 函数是我们具体要做的事，这边做的就是简单的打印而已，实际上我们可能对客户端发送来的信息做更多的处理。

## 5.1 main.c

    
    
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <netinet/in.h>
    #include <arpa/inet.h>
    #include <sys/epoll.h>
    #include <errno.h>
    #include <unistd.h>
    #include <ctype.h>
    #include <fcntl.h>
    #include <sys/shm.h>
    #include <sys/types.h>
    #include <sys/sem.h>
    #include <signal.h>
    #include <pthread.h>
    #include "threadpool.h"
    
    #define PORT 8000
    #define OPEN_MAX 1024
    
    struct shared{
    	int written;    // 作为一个标志，非0：表示可读，0：表示可写
    	char text[OPEN_MAX];      // 记录写入 和 读取 的文本
    
    };
    pthread_mutex_t mutex;
    pthread_cond_t cond;
    pthread_t pid[5];
    
    
    void *Test(void* arg)
    {
        struct shared* shm_shared = (struct shared*)arg; 
    		while(1)
    		{
    			if(shm_shared->written == 1)
    			{
            		printf("read: %s\n",shm_shared->text);
    			//	write(5,shm_shared->text,);
           
    				shm_shared->written = 0;
    				break;
               	}	
    		}
    	return NULL;
    }
    
    
    int main()
    {
    	int i;
    	
    	struct shared *shm_shared;
       
       //创建一个线程池
       	threadpool_t pool;
    	threadpool_init(&pool, 3);
    	
    	//创建共享内存
    	int shmid;
    	void* shmadd;
    	//struct shared *shm_shared;
    	if((shmid = shmget(IPC_PRIVATE,10,IPC_CREAT|0666)) < 0)
    	{
    		perror("shmget\n");
    		exit(-1);
    	}
    	printf("创建的共享内存为：%d\n",shmid);
    
    	//挂载共享内存到进程内，成功返回共享内存的起始地址
    	if((shmadd = shmat(shmid,NULL,0)) < (char*)0)
    	{
    		perror("shmat");
    		exit(-1);
    	}
    
    	shm_shared = (struct shared*)shmadd;
    	shm_shared->written = 0;
    
    	//创建套接字，监听文件描述符
    	int sockfd;
    	struct sockaddr_in seraddr;	
    	sockfd = socket(AF_INET,SOCK_STREAM,0);
    	//设置非阻塞
    	int flags = fcntl(sockfd,F_GETFL);
    	fcntl(sockfd,F_SETFL,flags | O_NONBLOCK);
    	//初始化端口和IP
    	bzero(&seraddr,sizeof(seraddr));
    	seraddr.sin_family = AF_INET;
    	seraddr.sin_addr.s_addr = htonl(INADDR_ANY);
    	seraddr.sin_port = htons(PORT);
    
    	int ret;
    	int on;
    	ret = setsockopt(sockfd,SOL_SOCKET,SO_REUSEADDR,(const char*)&on,sizeof(on));
    	if (ret == -1)
    		perror("bind");
    	//绑定服务器
    	ret = bind(sockfd,(struct sockaddr*)&seraddr,sizeof(seraddr));
    	if(ret == -1){
    		perror("bind");
    	}
    
    	//监听套接字
    	if(listen(sockfd,SOMAXCONN) < 0)
    	{
    		perror("listen");
    	}
    
    	int client[OPEN_MAX];
    	int efd;
    	struct epoll_event event,events[OPEN_MAX];
    	//将客户端标识初始化为-1
    	for(i = 0; i<OPEN_MAX; i++)
    	{
    		client[i] = -1;
    	}
    	
    	//监听事件个数
    	efd = epoll_create(OPEN_MAX);
    	if(efd == -1)
    	{
    		perror("epoll_create");
    	}
    		
    	event.events = EPOLLIN;   //监听文件描述符的可读事件
    	event.data.fd = sockfd;     //设置监听的文件描述符
    
    	ret = epoll_ctl(efd,EPOLL_CTL_ADD,sockfd,&event);
    	if(ret == -1)
    	{
    		perror("epoll_ctl");
    	} 
    
    	socklen_t len;
    	int confd,nready;
    	struct sockaddr_in cliaddr;
    	char  buf[OPEN_MAX] = {0};
    	while(1)
    	{
    		printf("wait....\n");
    		nready = epoll_wait(efd,events,OPEN_MAX,-1);
    		for(i = 0; i<nready;i++)
    		{
    			if(events[i].data.fd == sockfd)
    			{
    				len = sizeof(cliaddr);
    				confd = accept(sockfd,(struct sockaddr*)&cliaddr,&len);
    				printf("ip = %s;port = %d\n",inet_ntoa(cliaddr.sin_addr),ntohs(cliaddr.sin_port));
    				
    				event.data.fd = confd;
    				event.events = EPOLLIN | EPOLLET;
    				epoll_ctl(efd,EPOLL_CTL_ADD,confd,&event);
    
    				//设置为非阻塞模式
    				//flags = fcntl(confd,F_GETFL);
    				//fcntl(confd, F_SETFL, flags | O_NONBLOCK);
    			}
    			else
    			{
    				confd = events[i].data.fd;
    				printf("connfd=%d\n",confd);
    		
    				
    			
    					int nread = read(confd,buf,sizeof(buf));
    			
    					if(nread == 0)
    					{
    						close(confd);
    						printf("client close\n");
    						event = events[i];
    						epoll_ctl(efd,EPOLL_CTL_DEL,confd,&event);
    						//break;
    					}			
    				 	else{
    						
    						//将数据写入共享内存中
    					
    						strcpy(shm_shared->text,buf);
    						shm_shared->written = 1;
    					
    						//添加一个任务
    						threadpool_add_task(&pool, Test, shm_shared);
    						memset(buf,0x0,OPEN_MAX);
    						
    					}
    				
    			}
    		}	
    	}
    
    
    	
    	//释放共享内存
    	shmdt(shmadd);
    	close(sockfd);
    	close(efd);
    	return 0;
    }
    

## 5.2 client.c

    
    
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <unistd.h>
    #include <sys/epoll.h>
    #include <errno.h>
    #include <netinet/in.h>
    #include <arpa/inet.h>
    
    
    #define SERV_PORT 8000
    #define OPEN_MAX 1024
    
    
    int main(int arg,char* argv[])
    {
    	struct sockaddr_in addr;
    	int sockfd ,id;
    
    	sockfd = socket(AF_INET,SOCK_STREAM,0);
    
    	bzero(&addr,sizeof(addr));
    	addr.sin_family = AF_INET;
    	inet_pton(AF_INET,"127.0.0.1",&addr.sin_addr);
    	addr.sin_port = htons(SERV_PORT);
    
    	//连接服务器
    	connect(sockfd,(struct sockaddr*)&addr,sizeof(addr));
    
    
    	char wbuf[OPEN_MAX] = {0};
    	char rbuf[OPEN_MAX] = {0};
    
    	while(fgets(wbuf,sizeof(wbuf),stdin) != NULL)
    	{
    		//通过sockfd给服务器发送数据
    		write(sockfd,wbuf,strlen(wbuf));
    		
    		/*
    		id = read(sockfd,rbuf,sizeof(rbuf));
    		if(id == 0)
    		{
    			printf("the other side has been closed\n");
    		}
    		*/
    		fputs(rbuf,stdout);
    		memset(wbuf,0,1024);
    		memset(rbuf,0,1024);
    	}
    	close(sockfd);
    	return 0;
    }
    

## 5.3 测试结果

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021042413062090.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210424130638675.png)

# 补充

  * 一个容易弄错的就是实际执行的函数和线程执行的函数是不同的，二者是嵌套关系。

# 总结

> 如有错误，欢迎指出。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210419170705247.jpg?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

