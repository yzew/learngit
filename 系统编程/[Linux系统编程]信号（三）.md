> 距离上一次利用高并发技术实现360度行车记录仪功能已经过去半年了。开始写一系列关于系统编程和网络编程内容进行总结。  
>  温故而知新，欢迎大家讨论学习。  
>  `10-06` 复习时间

### 文章目录

  * 1 信号的概念
  * 2 信号的共性
  * 3 信号的机制
  * 4 与信号相关的事件和状态
  * 5 信号四要素
  * 6 Linux常规信号汇总
  * 7 信号的产生
  *     * 7.1 终端按键产生信号
    * 7.2 硬件异常产生信号
    * 7.3 kill 函数/命令产生信号
    *       * 7.3.1 函数原型
      * 7.3.2 使用kill函数终止任意进程
      * 7.3.3 kill 命令
    * 7.4 软件条件产生信号
    *       * 7.4.1 alarm函数原型
      * 7.4.2 测试你使用的计算机 1 秒钟能数多少个数
      * 7.4.3 使用 time 命令查看程序执行的时间（优化）
      * 7.3.5 setitimer函数原型
      * 7.3.6 测试你使用的计算机 1 秒钟能数多少个数（方式二）
  * 8 信号集操作函数+原理（重）
  *     * 8.1 sigprocmask 函数
    * 8.2 sigpending 函数
    * 8.3 屏蔽ctrl+c信号，打印未决信号集
  * 9 信号捕捉(重)
  *     * 9.1 signal 函数
    * 9.2 signal 函数修改ctrl+c触发事件
    * 9.3 sigaction 函数
    * 9.4 信号捕捉特性(重)
    * 9.5 验证信号捕捉特性二
    * 9.6 内核实现信号捕捉过程
  * 8 SIGCHLD信号（面试常问）
  *     * 8.1 SIGCHLD的产生条件
    * 8.2 借助 SIGCHLD 信号回收子进程(重)
  * 总结

# 1 信号的概念

信号是软件中断，很多比较重要的应用程序都需要处理信号。信号是一种进程之间或者内核与进程间异步通信的一种机制，例如：用户在终端键入中断键，会通过信号机制停止一个程序.

# 2 信号的共性

  1. 简单
  2. 不能携带大量信息
  3. 满足某个特设条件才发送

# 3 信号的机制

A 给 B 发送信号，B 收到信号之前执行自己的代码，收到信号后，不管执行到程序的什么位置，都要暂停运行，  
去处理信号，处理完毕再继续执行。与硬件中断类似——异步模式。但信号是软件层面上实现的中断，早期常被称  
为“软中断”。

**信号的特质** ：由于信号是通过软件方法实现，其实现手段导致信号有很强的延时性。但对于用户来说，这个延  
迟时间非常短，不易察觉。  
每个进程收到的所有信号，都是由内核负责发送的，内核处理。

# 4 与信号相关的事件和状态

**产生信号:**

  1. 按键产生，如：Ctrl+c、Ctrl+z、Ctrl+\
  2. 系统调用产生，如：kill、raise、abort
  3. 软件条件产生，如：定时器 alarm
  4. 硬件异常产生，如：非法访问内存(段错误)、除 0(浮点数例外)、内存对齐出错(总线错误)
  5. 命令产生，如：kill 命令

**递达** :递送并且到达进程。

**未决** :产生和递达之间的状态。主要由于阻塞(屏蔽)导致该状态。

**信号的处理方式:**

  1. 执行默认动作(每一个信号都有默认的处理事件)
  2. 忽略(丢弃)
  3. 捕捉(调用户处理函数)

Linux 内核的进程控制块 PCB 是一个结构体，task_struct, 除了包含进程 id，状态，工作目录，用户 id，组
id，文件描述符表，还包含了信号相关的信息，主要指阻塞信号集和未决信号集。

**阻塞信号集(信号屏蔽字)** ：将某些信号加入集合，对他们设置屏蔽，当屏蔽 x 信号后，再收到该信号，该信号的处理将推后(解除屏蔽后)

**未决信号集:**

  1. 信号产生，未决信号集中描述该信号的位立刻翻转为 1，表信号处于未决状态。当信号被处理对应位翻转回为 0。这一时刻往往非常短暂。
  2. 信号产生后由于某些原因(主要是阻塞)不能抵达。这类信号的集合称之为未决信号集。在屏蔽解除前，信号一直处于未决状态。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210411170409293.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210411170820551.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

# 5 信号四要素

> 在信号使用之前，要确定其四要素

  1. 编号
  2. 名称
  3. 事件
  4. 默认处理动作

**指令**

  * `man 7 signal` 查看信号和默认事件
  * `kill –l` 查看所有信号  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210411171523784.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210411171530575.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)  
**默认动作**

  1. Term：终止进程
  2. Ign： 忽略信号 (默认即时对该种信号忽略操作)
  3. Core：终止进程，生成 Core 文件。(查验进程死亡原因， 用于 gdb 调试)
  4. Stop：停止（暂停）进程
  5. Cont：继续运行进程

# 6 Linux常规信号汇总

  1. SIGHUP: 当用户退出 shell 时，由该 shell 启动的所有进程将收到这个信号，默认动作为终止进程
  2.  **SIGINT** ：当用户按下了<Ctrl+C>组合键时，用户终端向正在运行中的由该终端启动的程序发出此信号。默认动  
作为终止进程。

  3. SIGQUIT：当用户按下<ctrl+>组合键时产生该信号，用户终端向正在运行中的由该终端启动的程序发出些信  
号。默认动作为终止进程。

  4. SIGILL：CPU 检测到某进程执行了非法指令。默认动作为终止进程并产生 core 文件
  5. SIGTRAP：该信号由断点指令或其他 trap 指令产生。默认动作为终止里程 并产生 core 文件。
  6. SIGABRT: 调用 abort 函数时产生该信号。默认动作为终止进程并产生 core 文件。
  7. SIGBUS：非法访问内存地址，包括内存对齐出错，默认动作为终止进程并产生 core 文件。
  8. SIGFPE：在发生致命的运算错误时发出。不仅包括浮点运算错误，还包括溢出及除数为 0 等所有的算法错误。  
默认动作为终止进程并产生 core 文件。

  9.  **SIGKILL** ：无条件终止进程。本信号不能被忽略，处理和阻塞。默认动作为终止进程。它向系统管理员提供了  
可以杀死任何进程的方法。

  10. SIGUSE1：用户定义 的信号。即程序员可以在程序中定义并使用该信号。默认动作为终止进程。
  11. SIGSEGV：指示进程进行了无效内存访问。默认动作为终止进程并产生 core 文件。
  12. SIGUSR2：另外一个用户自定义信号，程序员可以在程序中定义并使用该信号。默认动作为终止进程。
  13. SIGPIPE：Broken pipe 向一个没有读端的管道写数据。默认动作为终止进程。
  14. SIGALRM: 定时器超时，超时的时间 由系统调用 alarm 设置。默认动作为终止进程。
  15. SIGTERM：程序结束信号，与 SIGKILL 不同的是，该信号可以被阻塞和终止。通常用来要示程序正常退出。  
执行 shell 命令 Kill 时，缺省产生这个信号。默认动作为终止进程。

  16. SIGSTKFLT：Linux 早期版本出现的信号，现仍保留向后兼容。默认动作为终止进程。
  17.  **SIGCHLD** ：子进程状态发生变化时，父进程会收到这个信号。默认动作为忽略这个信号。
  18. SIGCONT：如果进程已停止，则使其继续运行。默认动作为继续/忽略。
  19. SIGSTOP：停止进程的执行。信号不能被忽略，处理和阻塞。默认动作为暂停进程。
  20.  **SIGTSTP** ：停止终端交互进程的运行。按下<ctrl+z>组合键时发出这个信号。默认动作为暂停进程。
  21. SIGTTIN：后台进程读终端控制台。默认动作为暂停进程。
  22. SIGTTOU: 该信号类似于 SIGTTIN，在后台进程要向终端输出数据时发生。默认动作为暂停进程。
  23. SIGURG：套接字上有紧急数据时，向当前正在运行的进程发出些信号，报告有紧急数据到达。如网络带外  
数据到达，默认动作为忽略该信号。

  24. SIGXCPU：进程执行时间超过了分配给该进程的 CPU 时间 ，系统产生该信号并发送给该进程。默认动作为  
终止进程。

  25. SIGXFSZ：超过文件的最大长度设置。默认动作为终止进程。
  26. SIGVTALRM：虚拟时钟超时时产生该信号。类似于 SIGALRM，但是该信号只计算该进程占用 CPU 的使用时  
间。默认动作为终止进程。

  27. SGIPROF：类似于 SIGVTALRM，它不公包括该进程占用 CPU 时间还包括执行系统调用时间。默认动作为终止  
进程。

  28. SIGWINCH：窗口变化大小时发出。默认动作为忽略该信号。
  29. SIGIO：此信号向进程指示发出了一个异步 IO 事件。默认动作为忽略。
  30. SIGPWR：关机。默认动作为终止进程。
  31. SIGSYS：无效的系统调用。默认动作为终止进程并产生 core 文件。
  32. SIGRTMIN ～ (64) SIGRTMAX：LINUX 的实时信号，它们没有固定的含义（可以由用户自定义）。所有的实时  
信号的默认动作都为终止进程。

# 7 信号的产生

## 7.1 终端按键产生信号

  1. Ctrl + c → 2) SIGINT（终止/中断） “INT” ----Interrupt
  2. Ctrl + z → 20) SIGTSTP（暂停/停止） “T” ----Terminal 终端。
  3. Ctrl + \ → 3) SIGQUIT（退出）

## 7.2 硬件异常产生信号

  * 除 0 操作 → 8) SIGFPE (浮点数例外) “F” -----float 浮点数。
  * 非法访问内存 → 11) SIGSEGV (段错误)  
总线错误 → 7) SIGBUS

## 7.3 kill 函数/命令产生信号

### 7.3.1 函数原型

  * `int kill(pid_t pid, int sig);` 成功：0；失败：-1 (ID 非法，信号非法，普通用户杀 init 进程等权级问题)，设置 errno
  * `sig`：不推荐直接使用数字，应使用宏名，因为不同操作系统信号编号可能不同，但名称一致
  * `pid > 0`: 发送信号给指定的进程。
  * `pid = 0`: 发送信号给 与调用 kill 函数进程属于同一进程组的所有进程。
  * `pid < 0`: 取|pid|发给对应进程组。
  * `pid = -1`：发送给进程有权限发送的系统中所有进程。

### 7.3.2 使用kill函数终止任意进程

    
    
    #include <stdio.h>
    #include <unistd.h>
    #include <stdlib.h>
    #include <signal.h>
    #define N 5
    int main(void)
    {
        int i;
        pid_t pid, q;	
        for (i = 0; i < N; i++) 
    	{
            pid = fork();
            if (pid == 0)
            {
    			break;
            }	
            if (i == 2)
                q = pid;
        }
        if (i < 5) 
        {            //子进程
    
                printf("I'm child %d, getpid = %u\n", i, getpid());
                sleep(10);	
        } 
    	else 
    	{                //父进程
            sleep(3);
            kill(q, SIGKILL);
            printf("------------kill %d child %u finish\n", 2, q);
            while (1);
        }	
        return 0;
    }
    
    

### 7.3.3 kill 命令

  * kill 命令产生信号：`kill -SIGKILL pid`

## 7.4 软件条件产生信号

### 7.4.1 alarm函数原型

设置定时器(闹钟)。在指定 seconds 后，内核会给当前进程发送 14）SIGALRM 信号。进程收到该信号，默认动作终止。

每个进程都有且只有唯一个定时器

  * `unsigned int alarm(unsigned int seconds);` 返回 0 或剩余的秒数，无失败。
  * 常用：取消定时器 alarm(0)，返回旧闹钟余下秒数。

### 7.4.2 测试你使用的计算机 1 秒钟能数多少个数

    
    
    #include <stdio.h>
    #include <unistd.h>
    
    int main(void)
    {
    	int i;
    	alarm(1);
    	for(i = 0; ; i++)
    		printf("%d\n", i);
    	return 0;
    }
    

### 7.4.3 使用 time 命令查看程序执行的时间（优化）

  * `time ./test`

  * 结论：实际执行时间 = 系统时间 + 用户时间 + 等待时间 （如下）  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210411220854331.png)

  * `time ./test > out` (将结果输入到out) 明显提高性能

  * 结论：程序运行的瓶颈在于 IO，优化程序，首选优化 IO

### 7.3.5 setitimer函数原型

> 更具体的细节 `man setitimer`查看手册

设置定时器(闹钟)。 可代替 alarm 函数。精度微秒 us，可以实现周期定时

  * `int setitimer` (int which, const struct itimerval *new_value, struct itimerval *old_value); 成功：0；失败：-1，设置errno
  * `which`：指定定时方式（①自然定时：`ITIMER_REAL`）（②虚拟空间计时(用户空间)`ITIMER_VIRTUAL`）（③运行时计时(用户+内核)：`ITIMER_PROF`）
  * `参数二`：结构体设置间隔时间和单次时间
  * `参数三`：返回旧闹钟的剩余时间（传出参数）
  * `it_value`： **结构体内部的参数** ：第一次触发信号的时间
  * `it_interval`： **结构体内部的参数** ：用来设定两次定时任务之间间隔的时间（第二次和第一次 or 第三次和第二次之间的时间间隔）

### 7.3.6 测试你使用的计算机 1 秒钟能数多少个数（方式二）

    
    
    #include <sys/types.h>
    #include <unistd.h>
    #include <wait.h>
    #include <string.h>
    #include <signal.h>
    #include <sys/time.h>
    #include <stdio.h>
    unsigned int my_alarm(unsigned int sec)
    {
     struct itimerval it,oldit;
     int ret;
      it.it_interval.tv_sec=0;//间隔事件
      it.it_interval.tv_usec=0;
      it.it_value.tv_sec=sec;//第一次触发时间
      it.it_value.tv_usec=0;
      ret=setitimer(ITIMER_REAL,&it,&oldit);
      if(ret==-1)
      perror("setitimer");
      return oldit.it_value.tv_sec;
    }
    int main(void)
    {
     int i;
     my_alarm(1); //默认发射9号信号  默认函数为终止程序 
     for(i=0;;i++)
     printf("%d\n",i);
     return 0; 
    
    }
    
    

# 8 信号集操作函数+原理（重）

**控制原理** ：内核通过读取未决信号集来判断信号是否应被处理。信号屏蔽字 mask 可以影响未决信号集。而我们可以在应用程序中自定义 set 来改变
mask。已达到屏蔽指定信号的目的。（重点）

> 更多函数具体使用 `man ***` 的方式查看手册

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210411231631398.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

  * `sigset_t set;` // typedef unsigned long sigset_t;
  * `int sigemptyset(sigset_t *set);` 将某个信号集清 0 成功：0；失败：-1
  * `int sigfillset(sigset_t *set);` 将某个信号集置 1 成功：0；失败：-1
  * `int sigaddset(sigset_t *set, int signum);` 将某个信号加入信号集 成功：0；失败：-1
  * `int sigdelset(sigset_t *set, int signum);` 将某个信号清出信号集 成功：0；失败：-1
  * `int sigismember(const sigset_t *set, int signum);`判断某个信号是否在信号集中 返回值：在集合：1；不在：0；出错：-1

## 8.1 sigprocmask 函数

用来 **屏蔽信号** 、 **解除屏蔽** 也使用该函数。其本质，读取或修改进程的信号屏蔽字(PCB 中)

  * `int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);` 成功：0；失败：-1，设置 errno
  * `set`：传入参数，是一个位图，set 中哪位置 1，就表示当前进程屏蔽哪个信号。
  * `oldset`：传出参数，保存旧的信号屏蔽集。
  * `how` 参数取值： 假设当前的信号屏蔽字为 mask 
    1. `SIG_BLOCK`: 该进程新的信号屏蔽字是其当前信号屏蔽字和set指向信号集的并集。set包含了我们希望阻塞的附加信号。
    2. `SIG_UNBLOCK`: 该进程新的信号屏蔽字是其当前信号屏蔽字和set所指向信号集的补集的交集。set包含了我们希望解除阻塞的信号.
    3. `SIG_SETMASK`: 自定义set替换mask（mask表示系统自带的阻塞信号集）

## 8.2 sigpending 函数

读取当前进程的未决信号集

  * `int sigpending(sigset_t *set)`; set 传出参数。 返回值：成功：0；失败：-1，设置 errno

## 8.3 屏蔽ctrl+c信号，打印未决信号集

> 从结果可以很清楚的看出，当我们按下ctrl+c
> 未决信号机2号位置变成1，并且一直保持，就是因为我们利用·`sigprocmask`设置了阻塞信号集，信号无法递达。
    
    
    #include <signal.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    
    void printset(sigset_t *ped)//打印操作 
    {
    	int i;
    	for(i = 1; i < 32; i++)
    	{
    		if((sigismember(ped, i) == 1))//利用成员判断函数
    		{
    			putchar('1');
    		} 
    		else 
    		{
    			putchar('0');
    		}
    	}
    	printf("\n");
    }
    
    int main(void)
    {
    	sigset_t set, oldset, ped;
    	sigemptyset(&set);//清空
    	sigaddset(&set, SIGINT);//设置阻塞 SIGINT
    //SIG_BLOCK 表示添加阻塞信号 set表示要覆盖的信号集 oldset表示旧的信号集
    	sigprocmask(SIG_BLOCK, &set, &oldset);	
    
    	while(1)
    	{
    		sigpending(&ped);       //获取未决信号集 传出
    		printset(&ped); //打印 自定义的函数
    		sleep(1);
    	}
    
    	return 0;
    }
    

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412103326861.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

# 9 信号捕捉(重)

## 9.1 signal 函数

**作用** ：注册一个信号捕捉函数（注册而非创建）  
**原型** ：

  * `sighandler_t signal(int signum, sighandler_t handler);`
  * `typedef void (*sighandler_t)(int);`//函数指针类型 需要注意的就是需要传入一个整形参数 不管用不用

## 9.2 signal 函数修改ctrl+c触发事件

    
    
    #include <stdio.h>
    #include <unistd.h>
    #include <stdlib.h>
    #include <errno.h>
    #include <signal.h>
    void do_sig(int a)
    {
        printf("hello world!\n");
    }
    int main(void)
    {      
        if (signal(SIGINT, do_sig)== SIG_ERR)
        {
            perror("signal");
            exit(1);
        }
        while (1) {
            printf("---------------------\n");
            sleep(1);
        }
    
        return 0;
    }
    

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412110102107.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

## 9.3 sigaction 函数

> 不做过多记录，用的较少，需要再回来补充

**作用** ：修改信号处理动作（通常在 Linux 用其来注册一个信号的捕捉函数）  
**原型** ：

  * `int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);`
  * `act`：传入参数，新的处理方式。
  * `oldact`：传出参数，旧的处理方式。

## 9.4 信号捕捉特性(重)

  1. 进程正常运行时，默认 PCB 中有一个信号屏蔽字，假定为☆，它决定了进程自动屏蔽哪些信号。当注册了某个信号捕捉函数，捕捉到该信号以后，要调用该函数。而该函数有可能执行很长时间，在这期间所屏蔽的信号不由☆来指定。而是用 sa_mask 来指定。调用完信号处理函数，再恢复为☆。
  2. XXX 信号捕捉函数执行期间，XXX 信号自动被屏蔽。
  3. 阻塞的常规信号不支持排队，产生多次只记录一次。（后 32 个实时信号支持排队）

## 9.5 验证信号捕捉特性二

> 从结果很容易看出在信号处理函数执行期间，该信号多次递送，那么只在处理函数之行结束后，处理一次。
    
    
    #include <stdio.h>
    #include <unistd.h>
    #include <stdlib.h>
    #include <errno.h>
    #include <signal.h>
    void do_sig(int a)
    {
        printf("hello world!\n");
        sleep(10);//休息10秒 让函数执行久一点
    }
    int main(void)
    {      
        if (signal(SIGINT, do_sig)== SIG_ERR)
        {
            perror("signal");
            exit(1);
        }
        while (1) {
            printf("---------------------\n");
            sleep(1);
        }
        return 0;
    }
    

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412111521668.png)

## 9.6 内核实现信号捕捉过程

> 值得说明的是第四步到达第五步，因为函数处理完需要返回（类似于返回值的感觉），在函数处理之前我们实在内核空间，所以还要返回一次。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210412113155128.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

# 8 SIGCHLD信号（面试常问）

## 8.1 SIGCHLD的产生条件

  * 子进程终止时
  * 子进程接收到 SIGSTOP 信号停止时
  * 子进程处在停止态，接受到 SIGCONT 后唤醒时

## 8.2 借助 SIGCHLD 信号回收子进程(重)

> 子进程结束运行，其父进程会收到 SIGCHLD 信号。该信号的默认处理动作是忽略。可以捕捉该信号，在捕捉函数中完成子进程状态的回收。

> 有几个代码中需要注意的问题 [ 重要]
>
>   *
> 开始设置的信号阻塞，是为了防止子进程结束了，父进程还没有走到注册捕获信号函数。这样就会导致子进程结束信号还是会执行默认的事件–忽略。所以多个了阻塞和解除阻塞
>   * 自定义事件`do_sig_child`中的`while`是为了防止，同一时间多个子进程同时结束，同时发送信号，就会导致 性质二【XXX
> 信号捕捉函数执行期间，XXX 信号自动被屏蔽。】。最多两个信号事件执行，多个子进程信号被忽略了。
>   * 父进程结束的时候我们要加个循环，防止父进程先与子进程结束，导致捕捉信号执行自定义函数的功能失效。
>

    
    
    # include<stdio.h>
    # include<stdlib.h>
    # include<string.h>
    # include<errno.h>
    # include<pthread.h>
    # include<signal.h>
    # include<unistd.h>
    # include<sys/wait.h>
    # include<sys/types.h>
    void sys_err(char* str)
    {
    	perror(str);
    	exit(1);
    }
    void do_sig_child(int sig_no)
    {
    	int status;
    	pid_t pid;
    	while((pid = waitpid(0,&status,WNOHANG))>0)
    	{
    		if(WIFEXITED(status))
    		{
    			printf("child %d exit %d \n",pid,WEXITSTATUS(status));	
    		}
    		else if(WIFSIGNALED(status))
    		{
    			printf("child %d exit %d \n",pid,WTERMSIG(status));
    		}
    	}
    
    }
    
    int main()
    {
    	int i;
    	pid_t pid;
    	//阻塞
    	sigset_t set,oldset;
    	sigemptyset(&set);//清空
    	sigaddset(&set, SIGCHLD);//设置阻塞 SIGINT
    	sigprocmask(SIG_BLOCK, &set, &oldset);	
    	for ( i=0;i<10;i++)//创建多个进程
    	{
    		pid=fork();
    		if(pid==0)
    		{
    			break;
    		}
    		else if(pid<0)
    		{
    			sys_err("fork");
    		}
    	}
    	if(10==i)//  父进程中注册捕获信号函数  
    	{
    		if (signal(SIGCHLD, do_sig_child)== SIG_ERR)
       	 	{
    	        	perror("signal");
    		        exit(1);
        		}
        		//解除阻塞
        		sigprocmask(SIG_UNBLOCK, &set, &oldset);
        		while(1)//防止父进程先与
        		{
        			printf("Parent ID %d\n", getpid());
        			sleep(100);
        		}
    	}
    	else
    	{
    		printf("i am %d child,pid=%d\n",i,getpid());
    	}	
    	return 0;
    }
    

# 总结

> 如有错误欢迎指出…

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406091722343.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

