> 距离上一次利用高并发技术实现360度行车记录仪功能已经过去半年了。开始写一系列关于系统编程和网络编程内容进行总结。  
>  温故而知新，欢迎大家讨论学习。

> 第一次复习时间2021-09-02  
>  补充点： 虚拟地址空间的概念  
>  僵尸进程和孤儿进程更为具体的描述

### 文章目录

  * 1 进程相关概念
  *     * 1.1 程序和进程
    * 1.2 并发
    * 1.3 虚拟内存和虚拟地址空间
    * 1.4 进程控制块PCB
    * 1.5 进程状态
  * 2 进程控制
  *     * 2.1 fork/getpid/getppid
    *       * 2.1.1 函数原型
      * 2.1.2 案例一 打印父子进程id
      * 2.1.3 案例二 顺序打印5个进程
    * 2.2 进程共享
    *       * 2.2.1 读时共享写时复制原则
      * 2.2.2 案例一：非共享变量数据
    * 2.3 exec函数族
    * 2.4 孤儿进程 僵尸进程
    *       * 2.4.1 概念+实例
    * 2.5 wait/waitpid
    *       * 2.5.1 wait函数原型及作用
      * 2.5.2 宏函数判断终止原因
      * 2.5.3 wait和宏函数配套使用实例
      * 2.5.4 waitpid函数原型及作用
      * 2.5.6 waitpid回收指定子进程
      * 2.5.7 waitpid回收指定子进程（错误写法）
      * 2.5.8 waitpid 回收全部子进程
      * 2.5.9 暴力回收子进程
  * 总结

# 1 进程相关概念

> 一些比较基本的概念，如果学过操作系统，这些内容都很常见，可以适当跳过…
>
>   * 单道程序设计
>   * 多道程序设计
>   * 微观串行、宏观并行 时间片
>   * 同步 和 异步
>   * 并发和并行
>

[相关术语参考链接](https://zhuanlan.zhihu.com/p/79886251)（重）

## 1.1 程序和进程

“程序(Program)”是一个静态的概念，一般对应于操作系统中的一个可执行文件  
执行中的程序叫做进程(Process)，是一个动态的概念。现代的操作系统都可以同时启动多个进程。比如：我们在用酷狗听音乐，也可以使用eclipse写代码，也可以同时用浏览器查看网页。

## 1.2 并发

并发是多个任务交替执行的，多个任务之间可能还是串行的。所有的并发处理都有排队等候，唤醒和执行这三个步骤。所以并发是宏观的观念，在微观上他们都是序列被处理的，只不过资源不会在某一个上被阻塞（一般是通过时间片轮转），所以在宏观上多个几乎同时到达的请求同时在被处理。如果是同一时刻到达的请求也会根据优先级的不同，先后进入队列排队等候执行。并发针对的是多个请求，比如：一个CPU，一个web服务，同时涌入多个请求，CPU需要交替切换的执行多个请求，而不是一个请求执行完成之后再执行下一个请求。并发的实质是一个物理CPU（也可以是多个物理CPU）在若干个程序之间多路复用，并发性是对有限物理资源强制行使
多用户共享以提高效率。

## 1.3 虚拟内存和虚拟地址空间

[参考链接](https://blog.csdn.net/u014379540/article/details/52263114?utm_medium=distribute.pc_relevant.none-
task-blog-baidujs_utm_term-0&spm=1001.2101.3001.4242)  
**虚拟内存** ：虚拟内存是一种逻辑上扩充物理内存的技术。基本思想是用软、硬件技术把内存与外存这两级存储器当做一级存储器来用。虚拟内存技术的实现利用了
**自动覆盖和交换技术** 。简单的说就是 **将硬盘的一部分作为内存来使用** 。

**虚拟地址空间** ：在32位的i386
CPU的地址总线的是32位的，也就是说可以寻找到4G的地址空间。我们的程序被CPU执行，就是0x000000000xFFFFFFFF这一段地址中。高1G的空间为内核空间，由操作系统调用，低3G的空间为用户空间，由用户使用。  
CPU在寻址的时候，是按照虚拟地址来寻址，然后通过 **MMU**
(内存管理单元)将虚拟地址转换为物理地址。因为只有程序的一部分加入到内存中，所以会出现所寻找的地址不在内存中的情况（CPU产生缺页异常），如果在内存不足的情况下，就会通过页面调度算法来将内存中的页面置换出来，然后将在外存中的页面加入到内存中，使程序继续正常运行。  
[原图链接](https://blog.csdn.net/dyq1991/article/details/106318021?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161844963416780274128863%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=161844963416780274128863&biz_id=0&utm_medium=distribute.pc_search_result.none-
task-blog-2~blog~first_rank_v2~rank_v29-8-106318021.nonecase&utm_term=mmu)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210415092147129.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

0902补充一个比较好的总结

>
> 为了在多进程环境下，使得进程之间的内存地址不受影响，相互隔离，于是操作系统就为每个进程独立分配一套`虚拟地址空间`，每个程序只关心自己的虚拟地址就可以，实际上大家的虚拟地址都是一样的，但分布到物理地址内存是不一样的。作为程序，也不用关心物理地址的事情。  
>  
>
> 每个进程都有自己的虚拟空间，而物理内存只有一个，所以当启用了大量的进程，物理内存必然会很紧张，于是操作系统会通过`内存交换技术`，把不常使用的内存暂时存放到硬盘（换出）(`这个就是虚拟内存)`，在需要的时候再装载回物理内存（换入）。  
>  
>  那既然有了虚拟地址空间，那必然要把虚拟地址「映射」到物理地址，这个事情通常由操作系统来维护。  
>  
>  那么对于虚拟地址与物理地址的映射关系，可以有`分段和分页`的方式，同时两者结合都是可以的。  
>  
>
> 内存分段是根据程序的逻辑角度，分成了栈段、堆段、数据段、代码段等，这样可以分离出不同属性的段，同时是一块连续的空间。但是每个段的大小都不是统一的，这就会导致内存碎片和内存交换效率低的问题。  
>  
>  于是，就出现了内`存分页，把虚拟空间和物理空间分成大小固定的页`，如在 Linux 系统中，每一页的大小为
> 4KB。由于分了页后，就不会产生细小的内存碎片。同时在内存交换的时候，写入硬盘也就一个页或几个页，这就大大提高了内存交换的效率。  
>  
>  再来，为了解决简单分页产生的页表过大的问题，就有了`多级页表`，它解决了空间上的问题，但这就会导致 CPU
> 在寻址的过程中，需要有很多层表参与，加大了时间上的开销。于是根据程序的局部性原理，在 CPU 芯片中加入了
> TLB，负责缓存最近常被访问的页表项，大大提高了地址的转换速度。  
>  
>  L`inux 系统主要采用了分页管理，但是由于 Intel 处理器的发展史，Linux 系统无法避免分段管理。`于是 Linux
> 就把所有段的基地址设为 0，也就意味着所有程序的地址空间都是线性地址空间（虚拟地址），相当于屏蔽了 CPU
> 逻辑地址的概念，所以段只被用于访问控制和内存保护。另外，Linxu
> 系统中虚拟空间分布可分为用户态和内核态两部分，其中用户态的分布：代码段、全局变量、BSS、函数栈、堆内存、映射区。

## 1.4 进程控制块PCB

我们知道，每个进程在内核中都有一个进程控制块（PCB）来维护进程相关的信息，Linux内核的进程控制块是 task_struct 结构体。  
`/usr/src/linux-headers-3.16.0-30/include/linux/sched.h` 文件中可以查看 struct
task_struct 结构体定义。其内部成员有很多，我们重点掌握以下部分即可：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210408222627564.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)  
补充

> 文件id查看指令：`ps aux`

## 1.5 进程状态

[参考链接](https://www.zhihu.com/people/chris-46-9)

进程基本的状态有 5
种。分别为新建态，就绪态，运行态，阻塞态与终止态。其中新建态为进程准备阶段，常与就绪态结合来看。![在这里插入图片描述](https://img-
blog.csdnimg.cn/20210415092334768.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

> 引起进程状态转换的具体原因如下：  
>  ​

  * NULL→新建态：执行一个程序，创建一个子进程。  
​

  * 新建态→就绪态：当操作系统完成了进程创建的必要操作，并且当前系统的性能和虚拟内存的容量均允许。  
​

  * 运行态→终止态：当一个进程到达了自然结束点，或是出现了无法克服的错误，或是被操作系统所终结，或是被其他有终止权的进程所终结。  
​

  * 运行态→就绪态：运行时间片到；出现有更高优先权进程。  
​

  * 运行态→等待态：等待使用资源；如等待外设传输；等待人工干预。  
​

  * 就绪态→终止态：未在状态转换图中显示，但某些操作系统允许父进程终结子进程。  
​

  * 等待态→终止态：未在状态转换图中显示，但某些操作系统允许父进程终结子进程。  
​

  * 终止态→NULL：完成善后操作。

# 2 进程控制

## 2.1 fork/getpid/getppid

### 2.1.1 函数原型

  * `pid_t fork(void);`失败返回-1；成功返回：① 父进程返回子进程的 ID(非负) ②子进程返回 0
  * `pid_t getpid(void);`获取当前进程 ID
  * `pid_t getppid(void);`获取当前进程的父进程 ID

补充：初学者常常有个问题，就是子进程的执行范围：其实子进程只会继续执行fork函数之后的部分…

### 2.1.2 案例一 打印父子进程id

**fork getpid getppid的使用**

    
    
    # include<stdio.h>
    # include<stdlib.h>
    # include<unistd.h>
    int main()
    {
      pid_t pid ;
      pid = fork();
      if(pid<0)
      {
      	perror("fork error");
      	
      }
      else if(pid==0)
      {
      	printf("I am child pid =%d my father pid=%d\n",getpid(),getppid());
      }
      else
      {
      	printf("I am father pid =%d my father pid =%d\n",getpid(),getppid());
      }
      return 0;
    }
    

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210409111939113.png)

### 2.1.3 案例二 顺序打印5个进程

**通过命令行参数指定创建进程的个数，每个进程休眠 1S 打印自己是第几个被创建的进程。**

    
    
    # include<stdio.h>
    # include<stdlib.h>
    # include<unistd.h>
    int main()
    {
      pid_t pid;
      int i;
      for(i=0;i<5;i++)
      {
      	pid =fork();
      	if(pid==0)
      	{
      	 break;
      	}
      }
      if(5==i)//父进程 等待for结束自己台跳出
      {
      	sleep(5);
      	printf("I am father\n");
      
      }
      else//子进程 提前break跳出 打印
      {
      	sleep(i);
      	printf("I am %d child \n",i+1);
      }
    
      return 0;
    }
    

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210409111137142.png)

## 2.2 进程共享

### 2.2.1 读时共享写时复制原则

**相同部分** ：全局变量、.data、.text、栈、堆、环境变量、用户 ID、宿主目录、进程工  
作目录、信号处理方式…  
**不同部分** ：1.进程 ID 2.fork 返回值 3.父进程 ID 4.进程运行时间 5.闹钟(定时器) 6.未决信号集

**值得注意的是** ：  
父子进程间遵循 **读时共享写时复制** 的原则。  
这样设计，无论子进程执行父进程的逻辑还是执行自己的逻辑都能节省内存开销。  
而不是简单的复制0-3G用户空间内容。

### 2.2.2 案例一：非共享变量数据

    
    
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <unistd.h>
    #include <pthread.h>
    
    int main(int argc, char *argv[])
    {
        int a = 10;
        pid_t pid = fork();             // 创建子进程
        if (pid == -1) {
            perror("fork error");
            exit(1);
    
        } else if (pid == 0) {          // 子进程
            
            a=a+10;
            printf("---child is created\n");
            printf("a=%d\n",a);
    
        } else if (pid > 0) {           // 父进程
    
            printf("---parent process: my child is %d\n", pid);
            printf("a=%d\n",a);
        }
        
        printf("===================end of file\n");  // 父子进程各自执行一次.
    
        return 0;
    }
    

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210409111239904.png)

## 2.3 exec函数族

> 在项目中也没有用到，也不做重点记录了，需要的时候补充。

fork 创建子进程后执行的是和父进程相同的程序（但有可能执行不同的代码分支），子进程往往要调用一种 exec 函数以执行另一个程序。当进程调用一种
exec 函数时，该进程的用户空间代码和数据完全被新程序替换，从新程序的启动例程开始执行。调用 exec 并不创建新进程，所以调用 exec 前后该进程的
id 并未改变。

将当前进程的.text、.data 替换为所要加载的程序的.text、.data，然后让进程从新的.text第一条指令开始执行，但进程 ID
不变，换核不换壳。

  * `int execl(const char *path, const char *arg, ...);`
  * `int execlp(const char *file, const char *arg, ...);`
  * `int execle(const char *path, const char *arg, ..., char *const envp[]);`
  * `int execv(const char *path, char *const argv[])`
  * `int execvp(const char *file, char *const argv[]);`
  * `int execve(const char *path, char *const argv[], char *const envp[]);`

## 2.4 孤儿进程 僵尸进程

### 2.4.1 概念+实例

**孤儿进程** : 父进程先于子进程结束，则子进程成为孤儿进程，子进程的父进程成为 init进程，称为 init 进程领养孤儿进程。

  * `ps ajx`

    
    
    
    # include<stdio.h>
    # include<stdlib.h>
    # include<unistd.h>
    int main()
    {
      pid_t pid ;
      pid = fork();
      if(pid<0)
      {
      	perror("fork error");
      	
      }
      else if(pid==0)
      {
      	while(1)
      	{
      	  printf("I am child pid =%d my father pid=%d\n",getpid(),getppid());
      	  sleep(1);
    	}
      }
      else
      {
      	printf("I am father pid =%d \n",getpid());
      	sleep(4);
      	printf("I am died\n");
      	return 0;
      }
      return 0;
    }
    
    

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210409143835805.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

**僵尸进程** : 子进程终止，父进程没有及时回收，子进程残留资源（PCB）存放于内核中，变成僵尸（Zombie）进程。

  * 如何解决问题呢，此时杀死父进程，父进程转变成init。init发现子进程是僵尸，自动回收。

补充 细致描述

>
> 什么是僵尸进程？Unix进程模型中，进程是按照父进程产生子进程，子进程产生子子进程这样的方式创建出完成各项相互协作功能的进程的。`当一个进程完成它的工作终止之后，它的父进程需要调用wait()或者waitpid()系统调用取得子进程的终止状态。`如果父进程没有这么做的话，会产生什么后果呢？`此时，子进程虽然已经退出了，但是在系统进程表中还为它保留了一些退出状态的信息，如果父进程一直不取得这些退出信息的话，这些进程表项就将一直被占用，此时，这些占着茅坑不拉屎的子进程就成为“僵尸进程”（zombie）。`系统进程表是一项有限资源，如果系统进程表被僵尸进程耗尽的话，系统就可能无法创建新的进程。  
>  
>
> 那么，孤儿进程又是怎么回事呢？孤儿进程是指这样一类进程：在进程还未退出之前，它的父进程就已经退出了，一个没有了父进程的子进程就是一个孤儿进程（orphan）。既然所有进程都必须在退出之后被wait()或waitpid()以释放其遗留在系统中的一些资源，那么应该由谁来处理孤儿进程的善后事宜呢？这个重任就落到了init进程身上，`init进程就好像是一个民政局，专门负责处理孤儿进程的善后工作。每当出现一个孤儿进程的时候，内核就把孤儿进程的父进程设置为init，而init进程会循环地wait()它的已经退出的子进程。这样，当一个孤儿进程“凄凉地”结束了其生命周期的时候，init进程就会代表党和政府出面处理它的一切善后工作。`  
>  
>
> 这样来看，孤儿进程并不会有什么危害，真正会对系统构成威胁的是僵尸进程。那么，什么情况下僵尸进程会威胁系统的稳定呢？设想有这样一个父进程：它定期的产生一个子进程，这个子进程需要做的事情很少，做完它该做的事情之后就退出了，因此这个子进程的生命周期很短，但是，父进程只管生成新的子进程，至于子进程退出之后的事情，则一概不闻不问，这样，系统运行上一段时间之后，系统中就会存在很多的僵尸进程，倘若用ps命令查看的话，就会看到很多状态为Z的进程。严格地来说，僵尸进程并不是问题的根源，罪魁祸首是产生出大量僵尸进程的那个父进程。因此，当`我们寻求如何消灭系统中大量的僵尸进程时，答案就是把产生大量僵尸进程的那个元凶枪毙掉（通过kill发送SIGTERM或者SIGKILL信号）。枪毙了元凶进程之后，它产生的僵尸进程就变成了孤儿进程，这些孤儿进程会被init进程接管，init进程会wait()这些孤儿进程，释放它们占用的系统进程表中的资源，这样，这些已经“僵尸”的孤儿进程就能瞑目而去了`。

## 2.5 wait/waitpid

> 一个进程在终止时会关闭所有文件描述符，释放在用户空间分配的内存，但它的
> PCB还保留着，内核在其中保存了一些信息：如果是正常终止则保存着退出状态，如果是异常终止则保存着导致该进程终止的信号是哪个。这个进程的父进程可以调用
> wait 或 waitpid 获取这些信息，然后彻底清除掉这个进程。我们知道一个进程的退出状态可以在 Shell 中用特殊变量$?查看，因为 Shell
> 是它的父进程，当它终止时 Shell 调用 wait 或 waitpid 得到它的退出状态同时彻底清除掉这个进程。

### 2.5.1 wait函数原型及作用

**作用：**

  1. 阻塞等待子进程退出
  2. 回收子进程残留资源
  3. 获取子进程结束状态(退出原因)。

**原型：**

  * `pid_t wait(int *status);`
  * 成功：清理掉的子进程 ID；失败：-1 (没有子进程)

### 2.5.2 宏函数判断终止原因

  * `WIFEXITED（status）`宏判断为真 表示程序正常退出
  * `WEXITSTATUS(status)`上一个宏判断为真 则返回状态值
  * `WIFSIGNALED(status)` 宏判断为真 表示程序异常退出
  * `WTERMSIG(status)` 上一个判断为真，则返回状态值

### 2.5.3 wait和宏函数配套使用实例

    
    
    #include <sys/types.h>
    #include <sys/wait.h>
    #include <unistd.h>
    #include <stdio.h>
    #include <stdlib.h>
    
    int main()
    {
    	pid_t pid,wpid;
    	int status;
    	pid =fork();
    	if(pid==0)
    	{
    		printf(" i am child,my id is%d\n",getpid());
    		printf("child die\n");
    		return 73;
    	}
    	else if(pid>0)
    	{
    		wpid=wait(NULL);//不关心怎么结束的
    		wpid = wait(&status);//等待子进程结束
    		if(wpid==-1)
    		{
    			perror("wait error");
    			exit(1);
    		}
    		if(WIFEXITED(status))//判断 子进程正常退出判断
    		{
    			printf("child exit with%d\n",WEXITSTATUS(status));
    			printf("------parent  finish\n");
    		}
    		if(WIFSIGNALED(status))//判断 子进程异常退出判断
    		{
    			printf("child exit with%d\n",WTERMSIG(status));
    		}
    	}
    	else
    	{
    		perror("fork");
    		return 1;
    	}
    	
    }
    

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210411104848888.png)

### 2.5.4 waitpid函数原型及作用

**作用** ：  
作用同 wait，但可指定 pid 进程清理，可以不阻塞。

**原型：**

  * `pid_t waitpid(pid_t pid, int *status, in options);` 成功：返回清理掉的子进程 ID；失  
败：-1(无子进程)

**参数pid**

  * > 0 回收指定 ID 的子进程

  * -1 回收任意子进程（相当于 wait）
  * 0 回收和当前调用 waitpid 一个组的所有子进程
  * < -1 回收指定进程组内的任意子进程

**参数三**

  * 使用WNOHANG，子进程正在运行，直接返回0。设置阻塞或者非阻塞

**注意** ：

  * 一次 wait 或 waitpid 调用只能清理一个子进程，清理多个子进程应使用循环。

### 2.5.6 waitpid回收指定子进程

    
    
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <unistd.h>
    #include <sys/wait.h>
    #include <pthread.h>
    int main(int argc, char *argv[])
    {
        int i;
        pid_t pid, wpid, tmpid;
        for (i = 0; i < 5; i++) {       
            pid = fork();
            if (pid == 0) {       // 循环期间, 子进程不 fork 
                break;
            }
            if (i == 2) {
                tmpid = pid;	//子进程pid
                printf("--------pid = %d\n", tmpid);
            }
        }
    
        if (5 == i) {       // 父进程, 从 表达式 2 跳出
    //      sleep(5);
            //wait(NULL);                           // 一次wait/waitpid函数调用,只能回收一个子进程.
            //wpid = waitpid(-1, NULL, WNOHANG);    //回收任意子进程,没有结束的子进程,父进程直接返回0 
            //wpid = waitpid(tmpid, NULL, 0);       //指定一个进程回收, 阻塞等待
            printf("i am parent , before waitpid, pid = %d\n", tmpid);
            //wpid = waitpid(tmpid, NULL, WNOHANG);   //指定一个进程回收, 不阻塞
            wpid = waitpid(tmpid, NULL, 0);         //指定一个进程回收, 阻塞回收
            if (wpid == -1) {
                perror("waitpid error");
                exit(1);
            }
            printf("I'm parent, wait a child finish : %d \n", wpid);
        } else {            // 子进程, 从 break 跳出
            sleep(i);
            printf("I'm %dth child, pid= %d\n", i+1, getpid());
        }
        return 0;
    }
    

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210411142805843.png)

### 2.5.7 waitpid回收指定子进程（错误写法）

> 这段代码从结果上可以看出做不到回收指定的进程。两段代码的区别在于子进程id的存放的方式。  
>  在这段代码中，我们在子进程中做了`pid = getpid();`父子进程资源不共享导致的是父进程不知道这个`pid`的值。  
>  而在上一段代码中，我们直接通过 `pid = fork();tmpid = pid;`这样的方式在父进程中成功保存子进程的进程号。
    
    
    //指定回收一个子进程错误示例
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <unistd.h>
    #include <sys/wait.h>
    #include <pthread.h>
    int main(int argc, char *argv[])
    {
        int i;
        pid_t pid, wpid;
        for (i = 0; i < 5; i++) {       
            if (fork() == 0) {       // 循环期间, 子进程不 fork 
                if (i == 2) {
                     pid = getpid();
                     printf("------pid = %d\n", pid);
                }
                break;
            }
        }
        if (5 == i) {       // 父进程, 从 表达式 2 跳出
            sleep(5);      
            printf("------in parent , before waitpid, pid= %d\n", pid);
            wpid = waitpid(pid, NULL, 0);  //指定一个进程回收
            if (wpid == -1) {
                perror("waitpid error");
                exit(1);
            }
            printf("I'm parent, wait a child finish : %d \n", wpid);
        } else {            // 子进程, 从 break 跳出
            sleep(i);
            printf("I'm %dth child, pid= %d\n", i+1, getpid());
        }
        return 0;
    }
    

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210411142720686.png)

### 2.5.8 waitpid 回收全部子进程

    
    
    // 回收多个子进程
    
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <unistd.h>
    #include <sys/wait.h>
    #include <pthread.h>
    
    
    int main(int argc, char *argv[])
    {
        int i;
        pid_t pid, wpid;
    
        for (i = 0; i < 5; i++) {       
            pid = fork();
            if (pid == 0) {       // 循环期间, 子进程不 fork 
                break;
            }
        }
    
        if (5 == i) {       // 父进程, 从 表达式 2 跳出
        
      	while((wpid=waitpid(-1,NULL,0))>0)
      	{
      		printf("I'm parent,wait a child finish:%d\n",wpid);
      	}
      	
        } else {            // 子进程, 从 break 跳出
            sleep(i);
            printf("I'm %dth child, pid= %d\n", i+1, getpid());
        }
    
        return 0;
    }
    

### 2.5.9 暴力回收子进程

  * `kill -9 父进程号`

# 总结

> 如有错误欢迎指出…

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406091722343.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

