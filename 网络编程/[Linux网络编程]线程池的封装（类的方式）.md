> 文章总结了另外一种线程池的封装方式，基本的思想和上一篇是一致的，只是做了一些细节上的升级或者说提升。
>
>
> [[Linux网络编程]线程池的封装（结构体方式）](https://blog.csdn.net/weixin_44972997/article/details/116033564)

> `2021-09-08`
>
>   * 我怎么觉的之前的代码 do_task 中间加了一个for循环有误呢，其实没有问题，就是比较白痴的在for循环最后加了一个break
>   * 注意下面改进的方向部分  
>  我们把任务类串成链表 ，链表作为线程池类的成员
>

### 文章目录

  * 改进的方向
  * 需要注意的东西
  * 代码+解释
  *     * pool.h
    * pool.cpp
    * task.h
    * task.cpp
  * 总结

# 改进的方向

  * 我们把任务结构体和线程池结构体修改成类的形式
  * 结构体方式中`thread_routine`对应的是`do_task`
  * 结构体方式中`thread_add_task`分解成`add_task`和`pthread_pool_dispatch`
  * 结构体中`thread_init`对应的是`pthread_pool`构造函数
  * 结构体中`thread_destroy`对应的是`pthread_pool_destory`
  * 我们把任务放入stl列表替代普通的链表方式
  * 每个任务设计对应两种方法（一个是取 一个是放）并做成了基类，因为任务对应的是函数的封装，对于
  * 新增的`pthread_pool_create`代替原本每次只是创建一个线程，可以指定创建个数的线程
  * 我们将线程池区分读和写`两种类型`，通过创建类实例对象的时候传入参数 `1和0`控制，一种只能取出任务执行，一种只能放入任务。

# 需要注意的东西

  * `do_task` 是每个线程都要调用的函数 ，对于10个线程 要有10个函数空间，如果把`do_task`做成类成员方法 那就只有一个空间在类的内部 10个线程调用这个空间那是不合适的，所以我们做成友元函数 这样属于类的成员方法 可以去调用使用类 但是地址空间不在类内部，很好解决了我们的问题 还能调用类这个对象 观察参数四 直接传入this 虽然破坏了封装性
  * 我们根据`pool_type_`这个类属性，判断这个线程是做读操作还是做写操作从而调用`task`的两种不同的方法。

# 代码+解释

> 代码不难，整体结构和上一篇一致。

## pool.h

    
    
    #ifndef _POOL_H_
    #define _POOL_H_
    
    #include <iostream>
    #include <vector>
    #include <queue>
    #include <list>
    #include <string>
    #include "condition.h"
    #include "task.h"
    using namespace std;
    
    class pthread_pool
    {
    public:
        pthread_pool(int num,int ptype);//多了一个类型//0 代表读 1 代表写
        bool add_task(task_base* task);
        void pthread_pool_create(int mun);
        friend void *do_task(void* );//友元函数 在类中定义 但是空间不在这边 而且函数不只有一个 参数为this表示把线程池传入 在do_task能使用到线程池的属性方法
        friend void *do_task1(void* );
    
        void pthread_pool_dispatch();
        void pthread_pool_destroy();
        ~pthread_pool();
    private:
        condition_t ready;		//声明  互斥锁+信号量对象（封装号的）
    
        list<task_base*> task_vector_;
        int counter_;			//线程池中当前线程数
        int idle_;				//线程池中当前正在等待任务的线程数
        int max_threads_;		//线程池中最大允许的线程数
        int quit_;				//销毁线程池的时候置1
        int task_count_;        //任务数量
        int pool_type_;         //
        int pos;
    };
    
    #endif//_POOL_H_
    
    

## pool.cpp

    
    
    #include "pool.h"
    #include "shmfifo.h"
    /*做任务 从线程池取出任务 执行 其实就是从动态数组取出任务地址执行它的方法（任务自带执行方法 两种（取和放（其实也只是调用共享内存对象的函数方法 取和放）））*/
    /*参数 data 就是线程池对象 需要类型强转*/
    /*这是一个友元函数*/
    void *do_task(void* data)//创建线程就是执行函数
     {
        pthread_pool *pp = (pthread_pool*)data;
        struct timespec abstime;
        int timeout = 0;
        list<task_base*>::iterator it;//迭代器
        task_base* tb;
        while(1)
        {
            //pp->idle_++;//线程数增加 好像没啥用
            condition_lock(&pp->ready);//注意  一个上锁 下面的每个判断都对应一个解锁 或者是阻塞wait放锁
    
            while(pp->task_count_ == 0)
            {	printf("thread 0x%x is waiting\n", (int)pthread_self());
                clock_gettime(CLOCK_REALTIME, &abstime); //获取当前时间
                abstime.tv_sec += 1;
                int status = condition_timedwait(&pp->ready, &abstime);//超时等待
                if (status == ETIMEDOUT)
                {
                    printf("thread 0x%x is wait timed out\n", (int)pthread_self());
                    timeout = 1;//通过一个标志未 再加一个判断 break 跳出两个while
                    break;
                }
            }
    
    
            for (it=pp->task_vector_.begin();it!=pp->task_vector_.end();++it)
                {
                        pp->task_count_--;//任务数量-1
                        tb=pp->task_vector_.front();//从链表的头取出一个任务
                        printf("thread 0x%x is working\n", (int)pthread_self());
                        pp->task_vector_.erase(it);//取出后删除相关位置
    
                        condition_unlock(&pp->ready);
                        printf("pp->pool_type_=%d\n",pp->pool_type_);
                        if(pp->pool_type_ == 0)//类型为0 放入 判断线程池的类型
                                tb->task_get();
                        if(pp->pool_type_ == 1)
                                tb->task_put();//类型为1 取出
                        condition_lock(&pp->ready);
    
    
                    break;
                }
    
            if (timeout == 1)
            {
                pp->counter_--;//线程数减少
                condition_unlock(&pp->ready);
                break;
            }
                condition_unlock(&pp->ready);
        }
    
    
    
        printf("thread 0x%x is exiting\n", (int)pthread_self());
        return NULL;
    
    }
    /*构造函数初始化  */
    pthread_pool::pthread_pool(int num,int ptype):max_threads_(num),pool_type_(ptype)//注意这边也有初始化
    {
        //创建num个线程 cond_wait()
        //	pthread_pool_create(max_threads_);
        this->counter_ = 0;//线程池内的线程数
        this->idle_ = 0;//线程池内正在等待的数量
        this->quit_ = 0;//销毁值改为1
        condition_init(&ready);// 初始化信号量和互斥锁
        task_count_=0;//开始的任务数量   add_task增加  do_task减少
    
    }
    
    /*往线程池里面添加任务 其实就是往链表（数组）中添加任务对象*/
    bool pthread_pool::add_task(task_base* task)
    {
        condition_lock(&ready);//上锁
        task_vector_.push_back(task);//添加到队列
        task_count_++;//任务数量增加
        condition_unlock(&ready);
        pthread_pool_dispatch();//分发任务
        return true;
    }
    /*线程的创建 在dispatch中使用到*/
    void pthread_pool::pthread_pool_create(int num)
    {
        //创建num个线程之后  signal
        //	pthread_pool_dispatch();
    
        pthread_t tid[num];
        int i;
        for (i = 0; i< num; i++)
        {       
                //do_task 是每个线程都要调用的函数 
                //对于10个线程 要有10个函数空间
                //如果把do_task做成类成员方法 那就只有一个空间在类的内部 10个线程调用这个空间那是不合适的
                //所以我们做成友元函数 这样属于类的成员方法 可以去调用使用类 但是地址空间不在类内部
                //很好解决了我们的问题 还能调用类这个对象 观察参数四 直接传入this 虽然破坏了封装性 
                //我们也可以把函数做成普通的函数 
                pthread_create(&tid[i], NULL, do_task, this);
    
        }
        this->counter_+=num;//线程数量+num
    
    }
    
    /*每次添加完任务 都要调用一次*/
    /*分发任务 */
    void pthread_pool::pthread_pool_dispatch()
    {
    
    //判断任务队列有没有数据 有的话 判断
    //有没有空闲线程 有的话就唤醒 没有就创建
    //其实这边可以多加几个判断 比如说任务太多了 多创建一个
    
                condition_lock(&ready);//分发任务也要上锁
                //估计是涉及到数量的判断
                if (task_count_ > 0 )//任务数量大于0唤醒线程
                {
                    condition_signal(&ready);//通知 做do_task
    
                }
                if (task_count_ > 0 && counter_ <= 0)//任务数量大于0 但是没有线程 则创建3个线程
                {
                    pthread_pool_create(3);//调用封装好的函数线程池创建线程
                }
                if (task_count_ <= 0)
                {
    
                    return;
                }
                condition_unlock(&ready);
    
    
    }
    
    void pthread_pool::pthread_pool_destroy( pthread_pool* pool;)
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
    
    pthread_pool::~pthread_pool()
    {
    
    }
    

## task.h

    
    
    #ifndef _TASK_H_
    #define _TASK_H_
    
    #include <iostream>
    #include <vector>
    #include <string>
    #include "shmfifo.h"
    
    using namespace std;
    extern int mytime;
    extern int myno;
    typedef struct chongzhi
    {
        char pno[12];
        char money[10];
    }chongzhi_t;
    //我们把任务做成基类是因为 任务有很多种 例如登入 消费 充值 
    class task_base
    {
    public:
        task_base(){}
        //任务有两种状态 就是 存和取
        virtual bool task_get()  = 0;//取信息
        virtual bool task_put()  = 0;//放信息
    
        virtual ~task_base()
        {
    
        }
    private:
    
    };
    
    
    
    class task_charge: public task_base  //充值任务
    {
    public:
        task_charge(char* data):pdata_(data)
        {
          gdata_ = new char;
        }
        bool task_get() ;
        bool task_put() ;
        ~task_charge()
        {
            cout <<"~task_charge..."<<endl;
        }
    
    private:
    
        char* pdata_;
        char* gdata_;
    
    };
    
    // 简单工厂模式
    class TaskFactory
    {
    public:
        //为什么设为静态  直接通过类名：：调用  一个传字符串 一个传字符地址
        static task_base* CreateTask(const string& name,char* &data)//这边是引用
        {
            task_base* tb = 0;
            if (name == "charge")
            {
                tb = new task_charge(data);
    
            }
                return tb;
        }
    };
    
    /*
    void do_all_task(vector<task_base*> &v)
    {
        vector<task_base*>::iterator it;
        for (it = v.begin(); it != v.end();++it)
        {
            (*it)->task_run();
        }
    }
    
    void del_all_task(vector<task_base*> &v)
    {
        vector<task_base*>::iterator it;
        for (it = v.begin(); it != v.end();++it)
        {
            delete (*it) ;
        }
    }*/
    
    
    #endif //_TASK_H_
    
    
    
    
    
    

## task.cpp

    
    
    #include "task.h"
    /*基类的任务类 两个纯虚的方法都要实现*/
    /*目的 从共享内存放入信息*/
    /*本质还还调用共享内存的方法*/
    bool task_charge::task_put()
    {
        cout<<"put:"<<pdata_<<endl;
        g_fifo.shmfifo_put(pdata_);//
        return true;
    }
    /*目的 从共享内存取出信息*/
    bool task_charge::task_get()
    {
    
        g_fifo.shmfifo_get(gdata_);//
        cout<<"get:"<<gdata_<<"->"<<mytime++<<endl;//mytime是次数
        return true;
    
    }
    
    
    

# 总结

> 如有错误，欢迎指出。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210419170705247.jpg?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

