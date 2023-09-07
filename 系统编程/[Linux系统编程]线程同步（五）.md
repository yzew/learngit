> 距离上一次利用高并发技术实现360度行车记录仪功能已经过去半年了。开始写一系列关于系统编程和网络编程内容进行总结。  
>  温故而知新，欢迎大家讨论学习。

> `2021-09-08 复习内容`：
>
>   * 复习代码
>   * 1 man 1 man 2 man 3 分别是标准命令 系统调用 和 库函数
>   * 编译需要 -l pthread 加载第三方库
>   *
> 什么叫第三方库呢：除本地类库、系统类库以外的类库，需要后来安装，才能调用的类库。C++常见第三方库[参考](https://blog.csdn.net/luckywqf/article/details/19215741?ops_request_misc=&request_id=&biz_id=&utm_medium=distribute.pc_search_result.none-
> task-
> blog-2~all~es_rank~default-2-19215741.pc_search_all_es&utm_term=c%2b%2b%20%E4%BB%80%E4%B9%88%E5%8F%AB%E7%AC%AC%E4%B8%89%E6%96%B9%E5%BA%93&spm=1018.2226.3001.4187)
>   * 信号量那边的代码 如果是多生产者 都消费者就是错误的，因为没有处理生产者和生产者 或者
> 消费者和消费者之间互斥访问的问题。可能存在同时进入临界区。
>   * sem_init 初始化 分线程间 参数为0 进程间 参数为1 注意
>

>
> `2021-10-06 补充内容`
>
>   * 条件变量虚假唤醒的问题
>   * 条件变量也叫做管程
>   * 生产者生产 上锁 生产 解锁 唤醒 ；消费者消费 上锁 while 判断阻塞 消费 释放锁。
>

### 文章目录

  * 1 同步的概念
  * 2 线程同步的概念
  * 3 为什么要有线程同步
  * 4 互斥量 mutex
  *     * 4.1 基本概念
    * 4.2 函数使用
    *       * 4.2.1 pthread_mutex_t 类型
      * 4.2.2 pthread_mutex_init 函数
      *         * 4.2.2.1静态初始化和动态初始化
      * 4.2.3 pthread_mutex_destroy 函数
      * 4.2.4 pthread_mutex_lock 函数
      * 4.2.5 pthread_mutex_unlock 函数
      * 4.2.6 pthread_mutex_trylock 函数
    * 4.3 加锁和解锁
    *       * 4.3.1 lock 与 unlock：
      * 4.3.2lock 与 trylock：
  * 5 读写锁
  * 6 条件变量
  *     * 6.1 基本概念
    * 6.2 为什么使用条件变量
    * 6.3 函数使用
    *       * 6.3.1 pthread_cond_init 函数
      *         * 6.3.1.1 静态初始化和动态初始化
      * 6.3.2 pthread_cond_destroy 函数
      * 6.3.3 pthread_cond_wait 函数（重）
      * 6.3.4 pthread_cond_timedwait 函数
      * 6.3.5 pthread_cond_signal 函数
      * 6.3.6 pthread_cond_broadcast 函数
    * 6.4 [实例]生产者消费者模型
    * 6.5 条件变量的有优点
  * 7 信号量
  *     * 7.1 基本概念
    * 7.2 函数使用
    *       * 7.2.1 sem_init 函数
      * 7.2.2 sem_destroy 函数
      * 7.2.3 sem_wait 函数
      * 7.2.4 sem_post 函数
      * 7.2.5 sem_trywait 函数
      * 7.2.6 sem_timedwait 函数
    * 7.3 [实例]生产者消费者模型
  * 补充-异步
  * 补充 两套信号量函数的区别！！
  * 总结

# 1 同步的概念

  * 所谓同步，即同时起步，协调一致。不同的对象，对“同步”的理解方式略有不同。
  * 设备同步，是指在两设备之间规定一个共同的时间参考；
  * 数据库同步，是指让两个或多个数据库内容保持一致，或者按需要部分保持  
一致；

  * 文件同步，是指让两个或多个文件夹里的文件保持一致。等等

# 2 线程同步的概念

  * 线程同步简单说就是线程排队
  * 线程同步是一种制约关系，一个线程的执行依赖另一个线程消息。  
当它没有得到另一个线程的消息时应等待，得到消息被唤醒

  * 线程同步使得多个线程协调工作从而带到一致性

# 3 为什么要有线程同步

  * 共享资源，多个线程都可对共享资源操作 [容易产生冲突]
  * 线程操作共享资源的先后顺序不确定
  * cpu处理器对存储器的操作一般不是原子操作

> **举个例子** ：  
>  两个线程都把全局变量增加1，这个操作平台需要三条指令完成  
>  从内存读到寄存器 →寄存器的值加1 →将寄存器的值写会内存。  
>
> 如果此时线程A取值在寄存器修改还未写入内存，线程B就从内存取值就会导致两次操作实际上只修改过一次。或者说后一次线程做的事情覆盖前一次线程做的事情。实际上就执行过一次线程。

# 4 互斥量 mutex

## 4.1 基本概念

  * Linux 中提供一把互斥锁 mutex（也称之为互斥量）。
  * 每个线程在对资源操作前都尝试先加锁，成功加锁才能操作，操作结束解锁。
  * 资源还是共享的，线程间也还是竞争的，但通过“锁”就将资源的访问变成互斥操作，而后与时间有关的错误也不会再产生了。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210414004141745.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

## 4.2 函数使用

### 4.2.1 pthread_mutex_t 类型

  * 其本质是一个结构体。为简化理解，应用时可忽略其实现细节，简单当成整数看待。
  * 变量 mutex 只有两种取值 1、0。

### 4.2.2 pthread_mutex_init 函数

**作用** ：  
初始化一个互斥锁—> 初值可看作 1

  * `int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr)`
  * 参 1：传出参数，调用时应传 &mutex
  * 参 2：互斥量属性。是一个传入参数，通常传 NULL，选用默认属性(线程间共享)。

#### 4.2.2.1静态初始化和动态初始化

  1. 静态初始化：如果互斥锁 mutex 是静态分配的（定义在全局，或加了 static 关键字修饰），可以直接使用宏进行初始化。e.g. `pthead_mutex_t muetx = PTHREAD_MUTEX_INITIALIZER;`
  2. 动态初始化：局部变量应采用动态初始化。e.g. pthread_mutex_init(&mutex, NULL)

### 4.2.3 pthread_mutex_destroy 函数

**作用** ：  
销毁一个互斥锁

  * `int pthread_mutex_destroy(pthread_mutex_t *mutex)`

### 4.2.4 pthread_mutex_lock 函数

**作用** ：  
**加锁** 。可理解为将 mutex–（或 -1），操作后 mutex 的值为 0。

  * `int pthread_mutex_lock(pthread_mutex_t *mutex);`

### 4.2.5 pthread_mutex_unlock 函数

**作用** ：  
**解锁** 。可理解为将 mutex ++（或 +1），操作后 mutex 的值为 1。

  * `int pthread_mutex_unlock(pthread_mutex_t *mutex);`

### 4.2.6 pthread_mutex_trylock 函数

**作用** ：  
尝试加锁

  * `int pthread_mutex_trylock(pthread_mutex_t *mutex);`

## 4.3 加锁和解锁

### 4.3.1 lock 与 unlock：

  * lock 尝试加锁，如果加锁不成功，线程阻塞，阻塞到持有该互斥量的其他线程解锁为止。
  * unlock 主动解锁函数，同时将阻塞在该锁上的所有线程全部唤醒，至于哪个线程先被唤醒，取决于优先级、调度。默认：先阻塞、先唤醒。
  * 例如：T1 T2 T3 T4 使用一把 mutex 锁。T1 加锁成功，其他线程均阻塞，直至 T1 解锁。T1 解锁后，T2 T3 T4 均被唤醒，并自动再次尝试加锁。
  * 可假想 mutex 锁 init 成功初值为 1。lock 功能是将 `mutex--`。而 unlock 则将 `mutex++`。

### 4.3.2lock 与 trylock：

  * lock 加锁失败会阻塞，等待锁释放。
  * trylock 加锁失败直接返回错误号（如：EBUSY），不阻塞

# 5 读写锁

> 这部分内容在项目中没有使用过，如有需要，下次补充…

# 6 条件变量

## 6.1 基本概念

  * 条件变量本身不是锁！但它也可以造成线程阻塞。通常与互斥锁配合使用。给多线程提供一个会合的场所。

## 6.2 为什么使用条件变量

  * 线程抢占互斥锁时，线程A抢到了互斥锁，但是条件不满足，线程A就会让出互斥锁让给其他线程，然后等待其他线程唤醒他；一旦条件满足，线程就可以被唤醒，并且拿互斥锁去访问共享区。经过这中设计能让进程运行更稳定。

## 6.3 函数使用

### 6.3.1 pthread_cond_init 函数

**作用** ：  
初始化一个条件变量

  * `int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict` attr);
  * 参 2：attr 表条件变量属性，通常为默认值，传 NULL 即可

#### 6.3.1.1 静态初始化和动态初始化

**静态初始化** ：

  * `pthread_cond_t cond=PTHREAD_COND_INITIALIZER`  
**动态初始化** ：

  * `int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict` attr);

### 6.3.2 pthread_cond_destroy 函数

**作用** ：  
销毁一个条件变量

  * i`nt pthread_cond_destroy(pthread_cond_t *cond);`

### 6.3.3 pthread_cond_wait 函数（重）

**作用** ：（非常重要 三点）

  1. 阻塞等待条件变量 cond（参 1）满足
  2. 释放已掌握的互斥锁（解锁互斥量）相当于 pthread_mutex_unlock(&mutex);
  3. 当被唤醒，pthread_cond_wait 函数返回时，解除阻塞并重新申请获取互斥锁 pthread_mutex_lock(&mutex);

  * `int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);`

> 1.2.两步为一个原子操作。

### 6.3.4 pthread_cond_timedwait 函数

**作用** ：  
限时等待一个条件变量

  * `int pthread_cond_timedwait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex, const struct timespec *restrict abstime);`

### 6.3.5 pthread_cond_signal 函数

**作用** ：  
唤醒至少一个阻塞在条件变量上的线程

  * `int pthread_cond_signal(pthread_cond_t *cond);`

### 6.3.6 pthread_cond_broadcast 函数

**作用** ：  
唤醒全部阻塞在条件变量上的线程

  * `int pthread_cond_broadcast(pthread_cond_t *cond);`

## 6.4 [实例]生产者消费者模型

> 代码不难，就是实现一个生产消费。如果没有商品了，消费者需要去等待。  
>  要求能够自助完整写完。
    
    
    //链表是新生产的挂载头部 消费也从头部消费 
    //当然也可以 初始化一个哨兵头结点 新的节点挂载尾巴 一个temp（初始化为哨兵头结点） 不断指向新节点的上一个 如果temp =head 表示位空也可以。
    // head = temp
    // temp->next =新结点
    // temp =temp->next
    //每次消费 就是 temp-> 估计这边还要一个双向链表 要不找不到上一个temp 所以还是挂在头结点好一点
    # include<stdio.h>
    # include<stdlib.h>
    # include<pthread.h>
    //静态的方式定义了一个互斥锁 和 一个条件变量
    pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
    pthread_cond_t has_product=PTHREAD_COND_INITIALIZER;
    
    struct msg
    {
        struct msg* next;//指针域
        int num;//数据域
    };
    
    struct msg* head;//全局头指针
    void * producer(void* p)
    {
        struct msg* mp;//创建一个节点指针 毕竟生产 要加入到链表
    
        while(1)
        {
            mp=(struct msg*)malloc(sizeof(struct msg));s
            mp->num = rand()%1000+1;//放入一个1-1000的数
            printf("produce is %d\n",mp->num);
            pthread_mutex_lock(&lock);
            mp->next=head;//先看一一句 head作为上一个节点 当前节点挂载链表头部
            head=mp;//head变量保存头指针
            pthread_mutex_unlock(&lock);
            //唤醒消费者
            pthread_cond_signal(&has_product);
            sleep(rand()%5);
    
        }
    
    
    
    }
    
    void* consumer(void* p)
    {
        struct msg* mp;
        while(1)
        {
            pthread_mutex_lock(&lock);
            while(head==NULL)
            {
                //空的时候 阻塞等待生产者
                pthread_cond_wait(&has_product,&lock);    
            }
            mp=head; //消费 取出头指针
            head=mp->next; //头指针后移
            pthread_mutex_unlock(&lock);
            printf("Consume is %d\n",mp->num);
            free(mp);//记得即使清空
            sleep(rand()%5);
        }
    }
    int main()
    {
        pthread_t pid,cid;
        //创建两个线程 执行生产者和消费者
        pthread_create(&pid,NULL,producer,NULL);
        pthread_create(&cid,NULL,consumer,NULL);
        //等待线程结束，回收 无传出值 也是防止主线程提前退出main空间
        pthread_join(pid,NULL);
        pthread_join(cid,NULL);
       	pthread_mutex_destroy(&lock);
       	pthread_cond_destroy(&has_product);
     
        return 0;
    }
    

## 6.5 条件变量的有优点

> 再三重复，条件变量不是锁，但是常常配合锁使用。

  * 相较于 mutex 而言，条件变量可以减少竞争。
  * 如直接使用 mutex，除了生产者、消费者之间要竞争互斥量以外，消费者之间也需要竞争互斥量，但如果汇聚  
（链表）中没有数据，消费者之间竞争互斥锁是无意义的。有了条件变量机制以后，只有生产者完成生产，才会引起消费者之间的竞争。提高了程序效率。

# 7 信号量

## 7.1 基本概念

  * 简单的说就是进化版的互斥锁（1~N）计数器
  * 记录当前可利用的资源数，当资源数量<=0时会阻塞，当资源数量>时才开始进行操作，另外信号量的操作均为原子操作

## 7.2 函数使用

### 7.2.1 sem_init 函数

**作用** ：  
初始化一个信号量

  * `int sem_init(sem_t *sem, int pshared, unsigned int value);`
  * 参 1：sem 信号量
  * 参 2：pshared 取 0 用于线程间；取非 0（一般为 1）用于进程间
  * 参 3：value 指定信号量初值

> 信号量的初值，决定了占用信号量的线程（进程）的个数。

### 7.2.2 sem_destroy 函数

**作用** ：

销毁一个信号量

  * `int sem_destroy (sem_t *sem);`

### 7.2.3 sem_wait 函数

**作用** ：  
给信号量加锁 –

  * `int sem_wait(sem_t *sem);`

### 7.2.4 sem_post 函数

**作用** ：  
给信号量解锁 ++

  * `int sem_post(sem_t *sem);`

### 7.2.5 sem_trywait 函数

**作用** ：  
尝试对信号量加锁 – (与 sem_wait 的区别类比 lock 和 trylock)

  * `int sem_trywait(sem_t *sem);`

### 7.2.6 sem_timedwait 函数

**作用** ：  
限时尝试对信号量加锁 –

  * `int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);`
  * 参 2：abs_timeout 采用的是绝对时间。
  * ![在这里插入图片描述](https://img-blog.csdnimg.cn/2021041420360554.png)

## 7.3 [实例]生产者消费者模型

> 根据注释，代码还是比较好理解的。两个信号量来控制，一个初始为0，表示一开始生产的数量为0.另外一个初始的数量是可以存放商品的空格数，初始化就是格子数N。  
>  要求直接写出下面代码。

补充 0425：互斥和条件变量的方式是我消费 你不能生产。 而两个信号量是你生产 我可以消费。毕竟环形的 不影响，最多你生产了，我不知道。就怕多个生产者
，你格子放入了还没移到下一个格子我又重复放入了。

    
    
    # include<stdio.h>
    # include<pthread.h>
    # include<stdlib.h>
    # include<semaphore.h>
    # define N 5
    int queue[N];
    sem_t blank_number,product_number;//两个信号量控制
    void * producer(void*arg)//生产
    {
        int p=0;
        while(1)
        {
            sem_wait(&blank_number);//空格的位置-1
            queue[p]=rand()%1000+1;//放入队列
            printf("product %d \n",queue[p]);
            sem_post(&product_number);//生产的数量+1
            p=(p+1)%N;//1-N 1-N 循环放入 模拟队列
            sleep(rand()%5);
        }
           
    }
    void * consumer(void* arg)//消费者
    {
        int c=0;
        while(1)
        {
            sem_wait(&product_number);//产品数量-1
            printf("Consume %d\n",queue[c]);
            queue[c]=0;
            sem_post(&blank_number);//空格数量+1
            c=(c+1)%N;
            sleep(rand()%5);
        }
    }
    int main()
    {
        sem_init(&blank_number,0,N);//空格的数量(剩余空间的位置)
        sem_init(&product_number,0,0);//已经生产的数量
        pthread_t pid,cid;
        pthread_create(&pid,NULL,producer,NULL);
        pthread_create(&cid,NULL,consumer,NULL);
        pthread_join(pid,NULL);
        pthread_join(cid,NULL);
    
    }
    
    

# 补充-异步

异步处理就是,你现在问我问题,我可以不回答你,等我用时间了再处理你这个问题.同步不就反之了，同步信息被立即处理 –
直到信息处理完成才返回消息句柄；异步信息收到后将在后台处理一段时间 – 而早在信息处理结束前就返回消息句柄。

# 补充 两套信号量函数的区别！！

![在这里插入图片描述](https://img-
blog.csdnimg.cn/6161fa08db39465d92c12c4bea917685.png?x-oss-
process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAV2luZGFsb3Zl,size_20,color_FFFFFF,t_70,g_se,x_16)

[参考链接](https://blog.csdn.net/u014426028/article/details/103042819/)

# 总结

> 如有错误欢迎指出…

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406091722343.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

