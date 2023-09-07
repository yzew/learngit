> 距离上一次利用高并发技术实现360度行车记录仪功能已经过去半年了。开始写一系列关于系统编程和网络编程内容进行总结。  
>  温故而知新，欢迎大家讨论学习。

> 2021-09-05 补充  
>
> 1、dup2与dup区别是dup2可以用参数newfd指定新文件描述符的数值。若参数newfd已经被程序使用，则系统就会将newfd所指的文件关闭，若newfd等于oldfd，则返回newfd,而不关闭newfd所指的文件。dup2所复制的文件描述符与原来的文件描述符共享各种文件状态。共享所有的锁定，读写位置和各项权限或flags等.  
>  返回值：  
>  若dup2调用成功则返回新的文件描述符，出错则返回-1.  
>  2 主控线程不能用return 退出main空间，子线程可以。exit退出进程

### 文章目录

  * 1 守护进程
  *     * 1.1 什么是守护进程
    * 1.2 守护进程创建步骤
    * 1.3 守护进程代码实现（重点）
  * 2 线程
  *     * 2.1 什么是线程
    * 2.2 线程共享资源
    * 2.3 线程间非共享资源
    * 2.4 线程的优缺点
    * 2.5 线程控制原语
    *       * 2.5.1 pthread_self 函数
      * 2.5.2 pthread_create 函数
      *         * 2.5.2.1 创建一个新线程，打印线程 ID（i值传递方式）
        * 2.5.2.2创建一个新线程，打印线程 ID（i地址传递方式）（错）
      * 2.5.3 pthread_exit 函数
      *         * 2.5.3.1 pthread_exit和exit return比较（重）
        * 2.5.3.2 pthread_exit 替代 主线程sleep+return （重）
      * 2.5.4 pthread_join 函数（重）
      *         * 2.5.4.1 pthread_join 举例使用
      * 2.5.5 pthread_detach 函数
      *         * 2.5.5.1 detach分离后 join出现的情况
      * 2.5.6 pthread_cancel 函数
    * 2.6 线程进程控制原语比对
    * 2.7 线程属性注意事项（重）

# 1 守护进程

## 1.1 什么是守护进程

  1. 在linux系统中，我们会发现在系统启动的时候有很多的进程就已经开始跑了，也称为服务，这也是我们所说的守护进程。
  2. 守护进程（daemon）是生存期长的一种进程，没有控制终端。
  3. 它们常常在系统引导装入时启动，仅在系统关闭时才终止。
  4. UNIX系统有很多守护进程，守护进程程序的名称通常以字母“d”结尾：例如，syslogd 就是指管理系统日志的守护进程
  5. 通过ps进程查看器 `ps -efj` 的输出实例，内核守护进程的名字出现在方括号中，大致输出如下：  
![#](https://img-blog.csdnimg.cn/2021041308235428.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

## 1.2 守护进程创建步骤

  1. fork子进程，让父进程终止。
  2. 子进程调用，setsid()创建会话。
  3. 通常根据需要，改变工作目录chdir
  4. 通常根据需要，重设umask文件权限掩码
  5. 通常根据需要，关闭/重定向文件描述符
  6. 守护进程，业务逻辑。while（）

## 1.3 守护进程代码实现（重点）

> 主要是理解一些概念，重点参考一下文献。

> 参考文献（非常重要）  
>  [1 掩码+进程组+会话的描述](https://zhuanlan.zhihu.com/p/25118420)  
>  [2
> dev/null是什么](https://blog.csdn.net/matthewei6/article/details/50561590?ops_request_misc=&request_id=&biz_id=102&utm_term=./dev/null&utm_medium=distribute.pc_search_result.none-
> task-blog-2~all~sobaiduweb~default-2-50561590.first_rank_v2_pc_rank_v29)  
>  [3标准输入 标准输出 标准错误](https://www.cnblogs.com/any91/p/7072553.html)
    
    
    #include <stdio.h>
    #include <sys/stat.h>
    #include <fcntl.h>
    #include <stdlib.h>
    #include <string.h>
    #include <unistd.h>
    #include <errno.h>
    #include <pthread.h>
    void sys_err(const char *str)
    {
    	perror(str);
    	exit(1);
    }
    int main(int argc, char *argv[])
    {
        pid_t pid;
        int ret, fd;
        pid = fork();
        if (pid > 0)                // 父进程终止
            exit(0);
        pid = setsid();           //创建新会话
        if (pid == -1)
            sys_err("setsid error");
        ret = chdir("/home/itcast/28_Linux");       // 改变工作目录位置
        if (ret == -1)
            sys_err("chdir error");
    
        umask(0022);            // 改变文件访问权限掩码
    
        close(STDIN_FILENO);    // 关闭文件描述符 0标准输入 //因为0 1 2 都是进程启动默认启动的
    
        fd = open("/dev/null", O_RDWR);  //  fd --> 0
        if (fd == -1)
            sys_err("open error");
    
        dup2(fd, STDOUT_FILENO); // 重定向 stdout和stderr
        dup2(fd, STDERR_FILENO);
    
        while (1);              // 模拟 守护进程业务.
    	return 0;
    }
    

# 2 线程

## 2.1 什么是线程

  1. 轻量级进程(light-weight process)，也有 PCB，创建线程使用的底层函数和进程一样，都是 clone

  2. 从内核里看进程和线程是一样的，都有各自不同的 PCB，但是 PCB 中指向内存资源的三级页表是相同的（下图区别进程）  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210413102044140.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

  3. 进程可以蜕变成线程

  4. 线程可看做寄存器和栈的集合

  5. 在 linux 下，线程最是小的执行单位；进程是最小的分配资源单位

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021041310202748.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

> 参考：《Linux 内核源代码情景分析》  
>
> 对于进程来说，相同的地址(同一个虚拟地址)在不同的进程中，反复使用而不冲突。原因是他们虽虚拟址一样，但，页目录、页表、物理页面各不相同。相同的虚拟址，映射到不同的物理页面内存单元，最终访问不同的物理页  
>  面。  
>  但！线程不同！两个线程具有各自独立的 PCB，但共享同一个页目录，也就共享同一个页表和物理页面。所以两个 PCB 共享一个地址空间。  
>  实际上，无论是创建进程的 fork，还是创建线程的 pthread_create，底层实现都是调用同一个内核函数 clone。  
>  如果复制对方的地址空间，那么就产出一个“进程”；如果共享对方的地址空间，就产生一个“线程”。  
>  因此：Linux 内核是不区分进程和线程的。只在用户层面上进行区分。所以，线程所有操作函数 pthread_* 是库函数，而非系统调用

## 2.2 线程共享资源

  1. 文件描述符表
  2. 每种信号的处理方式
  3. 当前工作目录
  4. 用户 ID 和组 ID
  5.  **内存地址空间** (.text/.data/.bss/heap/共享库)

## 2.3 线程间非共享资源

  1. 线程 id
  2. 处理器现场和栈指针(内核栈)
  3.  **独立的栈空间** (用户空间栈)
  4. errno 变量
  5.  **信号屏蔽字**
  6. 调度优先级

## 2.4 线程的优缺点

**优点：**

  1. 提高程序并发性
  2. 开销小
  3. 数据通信、共享数据方便

**缺点：**

  1. 线程不稳定（第三方库函数实现）
  2. 线程调试困难
  3. 等待使用共享资源时造成程序运行速度变慢，主要是一些独占性的资源
  4. 线程的死锁，较长时间的等待或者资源竞争造成死锁

## 2.5 线程控制原语

> 编译的时候记得后面 `-l pthread` 毕竟第三方库实现

### 2.5.1 pthread_self 函数

**作用** ：  
获取线程 ID。其作用对应进程中 getpid() 函数。

  * `pthread_t pthread_self(void);`//成功返回本线程id

### 2.5.2 pthread_create 函数

创建一个新线程，起作用，对应进程中fork()函数。

  * `int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg)`; 成功返回0，失败返回errno
  * 参数一：表示传出参数，表示创建的子线程id
  * 参数二：线程属性，传NILL表使用默认属性
  * 参数三：函数指针，指向线程主函数(线程体)，该函数运行结束，则线程结束。
  * 参数四：参数三函数的参数，空传NULL

#### 2.5.2.1 创建一个新线程，打印线程 ID（i值传递方式）

> 主线程结束，但是进程地址空间还在，其他子线程正常执行.
> 如果main中return，由于其他子线程函数空间是在main函数里面，所以main不能提前结束。sleep就是让子进程中的函数执行完。（重）

> 注意 pthread_create 第四个参数的传递 ，以及后面的强制转换。可以先去体会一下错误写法（下下方的例子）
    
    
    # include<stdio.h>
    # include<stdlib.h>
    # include<string.h>
    # include<unistd.h>
    # include<errno.h>
    # include<pthread.h>
    void* fun(void* arg)
    {
    	int i=(int)arg;
    	sleep(i);
    	printf("I am %dth thread: pid=%d,tid=%lu\n",i+1,getpid(),pthread_self());
    	return NULL;
    }
    void sys_err(const char*str)
    {
    	perror(str);
    	exit(1);
    }
    int main()
    {
    	int i;
    	int ret;
    	pthread_t tid;
    	for(i=0;i<5;i++)
    	{
    		ret = pthread_create(&tid,NULL,fun,(void*)i);
    		if(ret!=0)
    		{
    			sys_err("pthread_creat error");
    		}
    	}
    	sleep(i);
    	return 0;
    }
    

#### 2.5.2.2创建一个新线程，打印线程 ID（i地址传递方式）（错）

>
> 从结果很容易看出i的打印出现的问题，那是因为当我们从子进程（函数内）通过i的地址取值的时候。这个i的地址是在父进程栈区。这片地址存放的值i可能已经发生了变化。毕竟父进程也一直在执行
>
>   * %lu表示输出无符号长整型整数
>   * 编译指令 `gcc test.c -o test -l pthread`(注意加后面的)
>   * 需要注意的是，不要让 **进程** 先与子线程结束，毕竟共享一片内存空间（代码中的sleep）  
>  ![在这里插入图片描述](https://img-blog.csdnimg.cn/2021041318130740.png)
>

    
    
    # include<stdio.h>
    # include<stdlib.h>
    # include<string.h>
    # include<unistd.h>
    # include<errno.h>
    # include<pthread.h>
    void* fun(void* arg)
    {
    	int i=*((int*)arg);
    	sleep(i);
    	printf("I am %dth thread: pid=%d,tid=%lu\n",i+1,getpid(),pthread_self());
    	return NULL;
    }
    void sys_err(const char*str)
    {
    	perror(str);
    	exit(1);
    }
    int main()
    {
    	int i;
    	int ret;
    	pthread_t tid;
    	for(i=0;i<5;i++)
    	{
    		ret = pthread_create(&tid,NULL,fun,(void*)&i);
    		if(ret!=0)
    		{
    			sys_err("pthread_creat error");
    		}
    	}
    	sleep(i);
    	return 0;
    }
    

### 2.5.3 pthread_exit 函数

**作用** ：  
将单个线程退出

  * `void pthread_exit(void *retval);` 参数：retval 表示线程退出状态，通常传 NULL

#### 2.5.3.1 pthread_exit和exit return比较（重）

> 直接说结论：
>
>   1. 线程中，禁止使用 exit 函数，会导致进程内所有线程全部退出。
>
>   2. return 在子线程是没有问题的，他是退出到函数调用的位置，也算是线程结束。毕竟线程也就是一个函数调用罢了。
>
>   3. pthread_exit 线程退出而不是进程退出切记。  
>  举个例子放下面，注释部分自己去掉试试就懂了
>
>   4. **主控线程退出时不能 return 或 exit** 。(重)
>
>

    
    
    # include<stdio.h>
    # include<stdlib.h>
    # include<string.h>
    # include<unistd.h>
    # include<errno.h>
    # include<pthread.h>
    void* fun(void* arg)
    {
    	int i=(int)arg;
    	if(i==2)
    	{
    		//return 0;
    		//exit(0);
    		//pthread_exit(NULL);
    	}
    	sleep(i);
    	printf("I am %dth thread: pid=%d,tid=%lu\n",i+1,getpid(),pthread_self());
    	return NULL;
    }
    void sys_err(const char*str)
    {
    	perror(str);
    	exit(1);
    }
    int main()
    {
    	int i;
    	int ret;
    	pthread_t tid;
    	for(i=0;i<5;i++)
    	{
    		ret = pthread_create(&tid,NULL,fun,(void*)i);
    		if(ret!=0)
    		{
    			sys_err("pthread_creat error");
    		}
    	}
    	sleep(i);
    	return 0;
    }
    

#### 2.5.3.2 pthread_exit 替代 主线程sleep+return （重）

> 主线程结束，但是进程地址空间还在，其他子线程正常执行. 如果main中return
> ，由于其他子线程函数空间是在main函数里面，所以main不能提前结束。
    
    
    # include<stdio.h>
    # include<stdlib.h>
    # include<string.h>
    # include<unistd.h>
    # include<errno.h>
    # include<pthread.h>
    void* fun(void* arg)
    {
    	int i=(int)arg;
    	if(i==2)
    	{
    		//return 0;
    		//exit(0);
    		//pthread_exit(NULL);
    	}
    	sleep(i);
    	printf("I am %dth thread: pid=%d,tid=%lu\n",i+1,getpid(),pthread_self());
    	return NULL;
    }
    void sys_err(const char*str)
    {
    	perror(str);
    	exit(1);
    }
    int main()
    {
    	int i;
    	int ret;
    	pthread_t tid;
    	for(i=0;i<5;i++)
    	{
    		ret = pthread_create(&tid,NULL,fun,(void*)i);
    		if(ret!=0)
    		{
    			sys_err("pthread_creat error");
    		}
    	}
    	pthread_exit(NULL);
    	
    }
    

### 2.5.4 pthread_join 函数（重）

**作用** ：  
阻塞等待线程退出，获取线程退出状态 其作用，对应进程中 waitpid() 函数

补充：任意线程得到其他线程的pid都可以回收，没有父线程回收子线程的说法。而进程需要父进程回收子进程。

  * `int pthread_join(pthread_t thread, void **retval);` 成功：0；失败：错误号

#### 2.5.4.1 pthread_join 举例使用

    
    
    #include <stdio.h>
    #include <unistd.h>
    #include <pthread.h>
    #include <stdlib.h>
    
    typedef struct {
    	int a;
    	int b;
    } exit_t;
    
    void *tfn(void *arg)
    {
    	exit_t *ret;
    	ret = malloc(sizeof(exit_t)); 
    
    	ret->a = 100;
    	ret->b = 300;
    
    	pthread_exit((void *)ret);
    }
    
    int main(void)
    {
    	pthread_t tid;
    	exit_t *retval;
    
    	pthread_create(&tid, NULL, tfn, NULL);
    
    	/*调用pthread_join可以获取线程的退出状态*/
    	pthread_join(tid, (void **)&retval);      //wait(&status);
    	printf("a = %d, b = %d \n", retval->a, retval->b);
    
    	return 0;
    }
    
    

### 2.5.5 pthread_detach 函数

**作用** ：  
实现线程分离，线程结束后，自动释放资源。无需pthread_join() 回收资源。

  * `int pthread_detach(pthread_t thread);` 成功：0；失败：错误号

> 线程分离状态：指定该状态，线程主动与主控线程断开关系。线程结束后，其退出状态不由其他线程获取，而直接自己自动释放。网络、多线程服务器常用。  
>  进程若有该机制，将不会产生僵尸进程。僵尸进程的产生主要由于进程死后，大部分资源被释放，一点残留资源仍存于系统中，导致内核认为该进程仍存在。  
>  也可使用 pthread_create 函数参 2(线程属性)来设置线程分离。

#### 2.5.5.1 detach分离后 join出现的情况

> 符合pthread_detach作用，分离独立。主线程无法再等待回收子线程资源。
    
    
    # include<stdio.h>
    # include<stdlib.h>
    # include<string.h>
    # include<unistd.h>
    # include<errno.h>
    # include<pthread.h>
    void* fun(void* arg)
    {
    	int i=(int)arg;
    	
    	printf("I am %dth thread: pid=%d,tid=%lu\n",i+1,getpid(),pthread_self());
    	sleep(10);
    	return NULL;
    }
    void sys_err(const char*str)
    {
    	perror(str);
    	exit(1);
    }
    int main()
    {
    	int i=1;
    	int ret;
    	pthread_t tid;
    	ret = pthread_create(&tid,NULL,fun,(void*)i);
    	if(ret!=0)
    	{
    		sys_err("pthread_creat error");
    	}
    	ret=pthread_detach(tid);
    	if(ret==0)
    	{
    		printf("success pthread_detach\n");
    	}
    	ret = pthread_join(tid,NULL);
    	if(ret!=0)
    	{
    		printf("pthread_join error\n");
    	}	
    	pthread_exit(NULL);
    	
    }
    

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210413203238181.png)

### 2.5.6 pthread_cancel 函数

**作用** ：  
杀死(取消)线程 其作用，对应进程中 kill() 函数。

  * `int pthread_cancel(pthread_t thread);` 成功：0；失败：错误号

> 线程的取消并 **不是实时** 的，而有一定的延时。需要等待线程到达某个取消点(检查点)。  
>  取消点：是线程检查是否被取消，并按请求进行动作的一个位置。通常是一些系统调用  
>  creat，open，pause， close，read，write… 执行命令 man 7 pthreads  
>  可以查看具备这些取消点的系统调用列表。也可参阅 APUE.12.7 取消选项小节。  
>  可粗略认为一个系统调用(进入内核)即为一个取消点。如线程中没有取消点，可以通过调用`pthread_testcancel`函数自行设置一个取消点。

## 2.6 线程进程控制原语比对

进程| 线程  
---|---  
fork| pthread_create  
exit| pthread_exit  
wait| pthread_join  
kill| pthread_cancel  
getpid| pthread_self  
  
## 2.7 线程属性注意事项（重）

  1. 主线程退出其他线程不退出，主线程应调用 pthread_exit
  2. 避免僵尸线程  
pthread_join  
pthread_detach  
pthread_create 指定分离属性  
被 join 线程可能在 join 函数返回前就释放完自己的所有内存资源，所以不应当返回被回收线程栈中的值;

  3. malloc 和 mmap 申请的内存可以被其他线程释放
  4. 应避免在多线程模型中调用 fork 除非，马上 exec，子进程中只有调用 fork 的线程存在，其他线程在子进程  
中均 pthread_exit

  5. 信号的复杂语义很难和多线程共存，应避免在多线程引入信号机制

> 如有错误欢迎指出…

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406091722343.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

