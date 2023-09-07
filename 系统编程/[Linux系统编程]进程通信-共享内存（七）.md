> 文章主要对共享内存的函数使用和案例实现进行总结，用于个人复习

> `2021-09-09`
>
>   *
> `shmget的参数三`如果要创建新的共享内存，需要使用IPC_CREAT，IPC_EXCL，如果是已经存在的，可以使用IPC_CREAT或直接传0。`所有我们常常传入0用来判断这个暗号的共享内存是否存在再来分别处理
> if else 存在和不存在的情况`
>

### 文章目录

  * 1 共享内存的概念
  * 2 共享内存的优点
  * 3 共享内存使用流程图
  * 4 函数使用
  *     * 4.1 shmget()函数
    * 4.2 shmat()函数
    * 4.3 shmdt()函数
    * 4.4 shmctl()函数
  * 5 共享内存实现消息传递
  * 总结

# 1 共享内存的概念

共享内存（Shared
Memory）就是允许多个进程访问同一个内存空间，是在多个进程之间共享和传递数据最高效的方式。操作系统将不同进程之间共享内存安排为同一段物理内存，进程可以将共享内存连接到它们自己的地址空间中，如果某个进程修改了共享内存中的数据，其它的进程读到的数据也将会改变。

共享内存并未提供锁机制，也就是说，在某一个进程对共享内存的进 行读写的时候，不会阻止其它的进程对它的读写。如果要对共享内存的读/写加锁，可以使用信号灯。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021041820532663.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)  
[引用原图链接](https://blog.csdn.net/An_Mo/article/details/104414157?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161874769516780357264356%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=161874769516780357264356&biz_id=0&utm_medium=distribute.pc_search_result.none-
task-
blog-2~all~top_click~default-2-104414157.first_rank_v2_pc_rank_v29&utm_term=%E5%85%B1%E4%BA%AB%E5%86%85%E5%AD%98)

# 2 共享内存的优点

  * 可以自定义数据类型
  * 共享内存是进程间共享数据的一种最快的方法，一个进程向共享的内存区域写入了数据，共享这个内存区域的所有进程就可以立刻看到其中的内容。

# 3 共享内存使用流程图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210418200223899.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

# 4 函数使用

    
    
    //头文件
    #include <sys/ipc.h>
    #include <sys/shm.h>
    

## 4.1 shmget()函数

**作用**  
用来获取或创建共享内存

  * `int shmget(key_t key, size_t size, int shmflg);`
  * 参数一:共享内存的键值,是一个整数，可以理解成暗号
  * 参数二：待创建的共享内存的大小，以字节为单位
  * 参数三：共享内存的访问权限，0666|IPC_CREAT表示全部用户对它可读写，如果共享内存不存在，就创建一个共享内存。补充：`如果要创建新的共享内存，需要使用IPC_CREAT，IPC_EXCL，如果是已经存在的，可以使用IPC_CREAT或直接传0。`
  * 返回值：成功创建返回共享内存的标识符，失败返回-1

## 4.2 shmat()函数

**作用**  
把共享内存连接到当前进程的地址空间（映射）

  * `void *shmat(int shm_id, const void *shm_addr, int shmflg);`
  * 参数一：参数shm_id是由shmget函数返回的共享内存标识
  * 参数shm_addr指定共享内存连接到当前进程中的地址位置，通常为空，表示让系统来选择共享内存的地址。
  * 参数shm_flg是一组标志位，通常为0。
  * 调用成功时返回一个指向共享内存第一个字节的指针，如果调用失败返回-1.

## 4.3 shmdt()函数

**作用**  
从本进程中去掉这块内存，也就是解除映射，也可以理解成分离

  * `int shmdt(const void *shmaddr);`
  * 参数shmaddr是shmat函数返回的地址。
  * 调用成功时返回0，失败时返回-1.

## 4.4 shmctl()函数

**作用**  
主要功能是删除共享内存，其实有其他功能，和消息队列的msgctl()类似。但是由于不重要，就不多介绍

  * `int shmctl(int shm_id, int command, struct shmid_ds *buf);`
  * 参数shm_id是shmget函数返回的共享内存标识符。
  * 参数command填IPC_RMID。
  * 参数buf填0。

# 5 共享内存实现消息传递

**shm_write**

    
    
    #include <unistd.h>
    #include <stdlib.h>
    #include <stdio.h>
    #include <string.h>
    #include <sys/shm.h>
    
    #define TEXT_SZ 2048
    struct shared_use_st
    {  
        int written;              //作为一个标志，非0：表示可读，0表示可写 
        char text[TEXT_SZ];       //记录写入和读取的文本
    };
    int main()
    {  
        int flat = 1;   
        void *shm = NULL;  
        struct shared_use_st *shared = NULL;      // 定义一个结构体指针指向shm 
        char buffer[BUFSIZ + 1];                  //用于保存输入的文本
        int shmid;                                //共享内存id 
        
        //创建一个共享内存对象 存在则打开 IPC_PRIVATE  不同的进程需要用key，(父子进程，同个进程内，IPC_PRIVATE)
        shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666|IPC_CREAT);
        if(shmid == -1)
        {  
            perror("shmget error");
            exit(-1);
        }   
        //挂载共享内存到进程中,成功返回共享内存的起始地址
        shm = shmat(shmid, (void*)0, 0);   
        if(shm == (void*)-1)
        {  
            perror("shmat error");
            exit(-1);
        }  
        printf("Memory attached at %X\n", (int)shm);     
        shared = (struct shared_use_st*)shm;   //设置共享内存 （挂钩）   
        while(flat)//向共享内存中写数据  
        {       //数据还没有被读取，则等待数据被读取,不能向共享内存中写入文本       
            while(shared->written == 1)       //信号量 互斥访问 
            {          
                sleep(1);      
                printf("Waiting read\n");
            }       //向共享内存中写入数据       
            printf("Enter some text: ");       
            fgets(buffer, BUFSIZ, stdin);      
            strncpy(shared->text, buffer, TEXT_SZ);      //写完数据，设置written使共享内存段可读       
            shared->written = 1;                         //写完可读 
            if(strncmp(buffer, "end", 3) == 0)          //输入了end，退出循环（程序）     
                flat = 0;   
        }   
        //卸载共享内存
        if(shmdt(shm) == -1)   
        {      
            perror("shmdt error");    
            exit(-1);
        }  
        sleep(2);  
        exit(-1);
    }
    
    

**shm _read**

    
    
    #include <unistd.h>
    #include <stdlib.h>
    #include <stdio.h>
    #include <sys/shm.h>
    
    #define TEXT_SZ 2048
    struct shared_use_st
    {  
        int written;        //作为一个标志，非0：表示可读，0表示可写 1
        char text[TEXT_SZ];//记录写入和读取的文本
    };
    
    
    
    int main(int argc, char const *argv[])
    {  
        int flat = 1;                      //程序是否继续运行的标志  
        void *shm = NULL;                 //分配的共享内存的原始首地址   
        struct shared_use_st *shared;     //定义了一个指向始首地址shm的结构体指针    
        int shmid;                        //共享内存标识符    
        
         //创建一个共享内存对象  IPC_PRIVATE  不同的进程需要用key，(父子进程，同个进程内，IPC_PRIVATE)
        shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666|IPC_CREAT);
        if(shmid == -1)
        {      
            perror("shmget error");
            exit(-1);
        }   
        //挂载共享内存到进程中,成功返回共享内存的起始地址
        shm = shmat(shmid, 0, 0);
        if(shm == (void*)-1)   
        {  
            perror("shmat error");
            exit(-1);
        }  
        
        shared = (struct shared_use_st*)shm; //设置共享内存 （挂钩）   
        shared->written = 0;                 //0为可写 
        while(flat)                          //循环读取数据 
        {         
            if(shared->written != 0)
            {      
                printf("write text: %s", shared->text);      
                sleep(rand() % 3);          //读取完数据，设置written使共享内存段可写
                shared->written = 0;           
                if(strncmp(shared->text, "end", 3) == 0)  //输入了end，退出循环（程序）  
                    flat = 0;       
            }      
            else//有其他进程在写数据，不能读取数据     
                sleep(1);  
        }   
         //卸载共享内存
        if(shmdt(shm) == -1)   
        {      
            perror("shmdt error");    
            exit(-1);
        }  
         //完成对共享内存的控制,释放共享内存
        if(shmctl(shmid, IPC_RMID, 0) == -1)   
        {  
            perror("shmctl error"); 
            exit(-1);
        }  
        exit(-1);
    }
    
    

# 总结

> 如有错误，欢迎指出。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406091722343.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

