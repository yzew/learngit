> 在上一篇的文章中总结了线程池的设计使用，本篇文章总结了环形共享内存。两者相辅相成，缺一不可，都是为了后面前置后置服务器做铺垫。  
>  完整的代码文中已经给出，如需整个测试项目，私信发。  
>  [目录链接](https://blog.csdn.net/weixin_44972997/article/details/115869213)  
>  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425111021975.png?x-oss-
> process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

> `2021-09-09`复习内容
>
>   * 注意头结构体 和 整体结构体。头结构体作为整体结构体的成员，整体结构体和几个操作共享内存的函数作为类的方法和属性，
>   * 头结构体包括：块大小 总块数 读写索引等基本信息
>   * 整体结构体包括头部指针
> 实际负载指针，共享内存的句柄，`4个信号量，区别于单个生产者和消费者2个信号量即可，这边还需要多出两个控制多个生产者之间和多个消费者之间的互斥，代码中注释都写的很详细。`
>   * 类中函数 包含 `ShmFifo`构造函数初始化
> 共享内存和信号量，`shmfifo_get`配和信号量和读索引从格子拷贝数据。`shmfifo_put`配合信号量和写索引从格子考入数据，还包括销毁和判断的函数。
>   * 注意到环形共享队列的好处，可以同时读和写。
>   *
> 注意shmget的第三个参数，如果要创建新的共享内存，需要使用IPC_CREAT，IPC_EXCL，如果是已经存在的，可以使用IPC_CREAT或直接传0，我们常常传入0来判断是否存在了
>   * shmat 参数注意
>

>
> `10-03修改内容`:
>
>   * 外层读写锁 内层互斥锁 那是不是 可以保存索引 和 索引后移 及时释放互斥锁 再进行拷贝 。这样应该会更快 可以多个生产者 或者消费者一起工作。
>

### 文章目录

  * 1基本介绍
  * 2 环形共享内存的设计
  *     *       * 2.1 结构图展示
      * 2.2 头结构设计
      * 2.3 整体结构设计
      * 2.4 类的设计（声明）
      * 2.5 类的定义+注释
  * 3 补充-信号量函数的封装
  * 总结

# 1基本介绍

共享队列是一台主机通信的最快方式，它的好处区别于消息队列及其他通信它可以自定义存储结构。在高并发的应用场景中，多个客户端同时发送信息，信息发送的速度远远大于处理速度，所以我们常常设计一个共享内存预先把发送来的信息进行存储。

# 2 环形共享内存的设计

### 2.1 结构图展示

>
> 从下图中我们可以很清楚的看出这个环形共享内存的大致信息，他包含了一个头部的结构体用于存放一些基本的信息（块大小、块总数、读索引、写索引）。后半部分由一块一块相同的信息存储结构构成，用于存放发送来的实际信息。  
>  0,1,2,3表示块的索引号。以及两个用于实际读写的索引下标。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425095239140.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

### 2.2 头结构设计

> 这里是引用
    
    
    typedef struct shmhead
    {
        unsigned int blksize;		// 块大小
        unsigned int blocks;		// 总块数
        unsigned int rd_index;		// 读索引
        unsigned int wr_index;		// 写索引
        int total;
    }shmhead_t;
    

### 2.3 整体结构设计

    
    
    typedef struct shmfifo
    {
        shmhead_t *p_shm;			// 共享内存头部指针
        char *p_payload;			// 有效负载的起始地址
    
        int shmid;					// 共享内存ID 操作的句柄
        int sem_mutex;				// 用来互斥用的信号量 用于同步读的线程（非读和写间）
        int sem_mutex1;				// 用来互斥用的信号量 用于同步写的线程 
        int sem_full;				// 用来控制共享内存是否满的信号量
        int sem_empty;				// 用来控制共享内存是否空的信号量
    }shmfifo_t;
    
    

### 2.4 类的设计（声明）

    
    
    class ShmFifo
    {
        public:
        	//构造函数用于初始化，申请共享内存空间，挂载映射,信号量的初始化，头结构变量的确定......
            ShmFifo(int key, int blksize,int blocks );
    
    		//数据放入共享内存
            void shmfifo_put(const void *buf);
    		//从共享内存取出数据 buf表示的是传出参数
            void shmfifo_get(void *buf);
    		//删除共享内存 包括删除信号量 取消映射 释放内存空间
            void shmfifo_destroy();
            int shmfifo_isempty();
    
        private:
            int key_;//共享内存的键值,是一个整数，可以理解成暗号
            int blksize_;//块的数量
            int blocks_;//块的大小
            shmfifo_t *fifo;//结构体指针
    
    };
    
    

### 2.5 类的定义+注释

> 代码总体来说难度不大，配合注释比较容易理解，由于对信号量进行了封装所以可能不习惯。  
>  一个需要注意的是这边四个信号量的使用，和多个`生产者消费者模型`是一致的。（比较容易错），可以对比互斥量加条件变量实现方式。  
>  **封装的一些函数**  
>  `int sem_d(int semid)`// 删除信号量  
>  `int sem_v(int semid)`//信号量++  
>  `int sem_p(int semid)`//信号量-- 为负数阻塞  
>  `int sem_create(int semid)`//创建信号量  
>  `int sem_setval(int semid, int val)`;//设置信号量的初始值
    
    
    ShmFifo::ShmFifo(int key, int blksize,int blocks )
            :key_(key),blksize_(blksize),blocks_(blocks)
    {
            fifo = (shmfifo_t *)malloc(sizeof(shmfifo_t));//共享内存信息的结构体 由于定义为shmfifo_t * fifo指针 所以开辟空间
            /*一个正确程序必须保证这个boolean表达式的值为true；如果该值为false，说明程序已经处于不正确的状态下，系统将给出警告并且退出*/
            assert(fifo != NULL);//
            memset(fifo, 0, sizeof(shmfifo_t));
    
    
            int shmid;
            //`如果要创建新的共享内存，需要使用IPC_CREAT，IPC_EXCL，如果是已经存在的，可以使用IPC_CREAT或直接传0。
            //这边通过把第三个参数设置为0来判断该共享内存是否存在
            shmid = shmget(key, 0, 0);
            int size = sizeof(shmhead_t) + blksize*blocks;//头+每个格子的长度+格子数 计算出总长
            if (shmid == -1)//判断一下这个键值的共享内存不存在 那就去创建
            {
                fifo->shmid = shmget(key, size, IPC_CREAT | 0666);
                if (fifo->shmid == -1)
                    ERR_EXIT("shmget");
                /*shmat 将这个内存映射到本进程的虚拟地址空间 （移到用户操作地段）强制转换 */
                fifo->p_shm = (shmhead_t*)shmat(fifo->shmid, NULL, 0);
                if (fifo->p_shm == (shmhead_t*)-1)
                    ERR_EXIT("shmat");
    
                fifo->p_payload = (char*)(fifo->p_shm + 1);//这个估计是从头结构体的头移到尾
    
                fifo->p_shm->blksize = blksize;
                fifo->p_shm->blocks = blocks;
                fifo->p_shm->rd_index = 0;//读的索引
                fifo->p_shm->wr_index = 0;//写的索引
                fifo->p_shm->total = 0;
                //初始化创建操作    
                fifo->sem_mutex = sem_create(key);//控制同为put的线程 不要你放入了我还放入 初始1
                fifo->sem_full = sem_create(key+1);//可写（空的格子数）初始为格子数
                fifo->sem_empty = sem_create(key+2);//可读 填满的位置 初始为0
                fifo->sem_mutex1 = sem_create(key+3);//控制同为get的线程 不要你读出来了我还放入 初始1
    
                sem_setval(fifo->sem_mutex, 1);
                sem_setval(fifo->sem_mutex1, 1);
                sem_setval(fifo->sem_full, blocks);//是否可写数据 空格数 想想生产者 消费者
                sem_setval(fifo->sem_empty, 0);
            }
            else//若一开始就存在 那就走着 初始化这些信号量
            {
                fifo->shmid = shmid;
                fifo->p_shm = (shmhead_t*)shmat(fifo->shmid, NULL, 0);
                if (fifo->p_shm == (shmhead_t*)-1)
                    ERR_EXIT("shmat");
    
                fifo->p_payload = (char*)(fifo->p_shm + 1);
                 //就是调用sem_get 获取或者创建一个信号量 并返回操作的句柄    
                fifo->sem_mutex = sem_open(key);//控制同为put的线程 不要你放入了我还放入 初始1
                fifo->sem_full = sem_open(key+1);//可写（空的格子数）初始为格子数
                fifo->sem_empty = sem_open(key+2);//可读 填满的位置 初始为0
                fifo->sem_mutex1 = sem_open(key+3);//控制同为get的线程 不要你读出来了我还放入 初始1
            }
    
    
    
    }
    
    void ShmFifo::shmfifo_put(const void *buf)
    {
            /*
            互斥和条件变量的方式是我消费 你不能生产。 
            而两个信号量是你生产 我可以消费。
            毕竟环形的不影响，最多你生产了，我不知道。
            就怕多个生产者 ，你格子放入了我还放入。这就是sem_mutex的作用 同类间的互斥
            */
            sem_p(fifo->sem_full);//可写信号量-1
            sem_p(fifo->sem_mutex);//就是防止我操作的时候其他线程不要和我一样一起放（做同样的事情）但是
            //源  复制部分  复制大小
            //把数据放入指定位置 ， 不断后移 拷贝 首地址位置  buf 长度 
            memcpy(fifo->p_payload+fifo->p_shm->blksize*fifo->p_shm->wr_index,
                buf, fifo->p_shm->blksize);
            //共享内存不断后移
            fifo->p_shm->wr_index = (fifo->p_shm->wr_index + 1) % fifo->p_shm->blocks;
            sem_v(fifo->sem_mutex);//释放锁
            sem_v(fifo->sem_empty);//可读信号量+1
    }
    
    void ShmFifo::shmfifo_get(void *buf)
    {
    
            sem_p(fifo->sem_empty);//判断是否有可读的
    
            sem_p(fifo->sem_mutex1);//防止我取你还取 同类之间
    
            //把指定共享内存读出 buf为传出参数
            memcpy(buf, fifo->p_payload+fifo->p_shm->blksize*fifo->p_shm->rd_index, fifo->p_shm->blksize);
            //读的指针便宜
            fifo->p_shm->rd_index = (fifo->p_shm->rd_index + 1) % fifo->p_shm->blocks;
    
            sem_v(fifo->sem_mutex1);//释放锁
            sem_v(fifo->sem_full);//可写信号量+1
    
    }
    
    void ShmFifo::shmfifo_destroy()
    {
            sem_d(fifo->sem_mutex);
            sem_d(fifo->sem_full);
            sem_d(fifo->sem_empty);
            sem_d(fifo->sem_mutex1);
    
            shmdt(fifo->p_shm);
            shmctl(fifo->shmid, IPC_RMID, 0);
            free(fifo);
    }
    
    int ShmFifo::shmfifo_isempty()
    {
        if (fifo->p_shm->total <= 0)
                return 1;
        else
                return 2;
    
    }
    

# 3 补充-信号量函数的封装

    
    
      int sem_create(key_t key)
        {
            int semid = semget(key, 1, 0666 | IPC_CREAT | IPC_EXCL);
            if (semid == -1)
                ERR_EXIT("semget");
    
            return semid;
        }
    
        int sem_open(key_t key)
        {
            int semid = semget(key, 0, 0);
            if (semid == -1)
                ERR_EXIT("semget");
    
            return semid;
        }
    
        int sem_p(int semid)
        {
            struct sembuf sb = {0, -1, 0};
            int ret = semop(semid, &sb, 1);
            if (ret == -1)
                ERR_EXIT("semop");
    
            return ret;
        }
    
        int sem_v(int semid)
        {
            struct sembuf sb = {0, 1, 0};
            int ret = semop(semid, &sb, 1);
            if (ret == -1)
                ERR_EXIT("semop");
    
            return ret;
        }
    
        int sem_d(int semid)
        {
            int ret = semctl(semid, 0, IPC_RMID, 0);
            /*
            if (ret == -1)
                ERR_EXIT("semctl");
            */
            return ret;
        }
    
        int sem_setval(int semid, int val)
        {
            union semun su;
            su.val = val;
            int ret = semctl(semid, 0, SETVAL, su);
            if (ret == -1)
                ERR_EXIT("semctl");
    
            //printf("value updated...\n");
            return ret;
        }
    
        int sem_getval(int semid)
        {
            int ret = semctl(semid, 0, GETVAL, 0);
            if (ret == -1)
                ERR_EXIT("semctl");
    
            //printf("current val is %d\n", ret);
            return ret;
        }
    
        int sem_getmode(int semid)
        {
                union semun su;
                struct semid_ds sem;
                su.buf = &sem;
                int ret = semctl(semid, 0, IPC_STAT, su);
                if (ret == -1)
                        ERR_EXIT("semctl");
    
                printf("current permissions is %o\n",su.buf->sem_perm.mode);
                return ret;
        }
    
        int sem_setmode(int semid,char* mode)
        {
                union semun su;
                struct semid_ds sem;
                su.buf = &sem;
    
                int ret = semctl(semid, 0, IPC_STAT, su);
                if (ret == -1)
                        ERR_EXIT("semctl");
    
                printf("current permissions is %o\n",su.buf->sem_perm.mode);
                sscanf(mode, "%o", (unsigned int*)&su.buf->sem_perm.mode);
                ret = semctl(semid, 0, IPC_SET, su);
                if (ret == -1)
                        ERR_EXIT("semctl");
    
                printf("permissions updated...\n");
    
                return ret;
        }
    

# 总结

> 如有错误，欢迎指出。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210419170705247.jpg?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

