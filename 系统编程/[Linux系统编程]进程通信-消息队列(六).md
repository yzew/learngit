> 文章主要对消息队列的函数使用和案例实现进行总结，用于个人复习

> `09-05`  
>  注意消息队列的创建和访问命令的不同  
>  本质为链表，结构体的参数一long类型用来指定类型。可以指定发送和接受的类型。

### 文章目录

  * 1 消息队列的简介
  * 2 linux 下消息队列查看和删除指令
  * 3消息队列相关函数
  *     * 3.1 msgget函数
    * 3.2 msgsnd函数
    * 3.3 msgrcv函数
    * 3.4 msgctl函数
  * 4 通过fork创建父子进程实现简单互发消息（两个消息队列 实现通信）
  * 补充-ipcs指令
  * 补充 -小点知识
  * 总结

# 1 消息队列的简介

消息队列作为通信方式的一种，在本质上是位于内核空间的链表，每个链表的节点都是一条消息。每条消息都有自己的消息类型且消息类型必须大于0。每种消息类型都被所对应的链表所维护。![在这里插入图片描述](https://img-
blog.csdnimg.cn/20200806195010379.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)  
如图1,2,3,4表示不同的数据，消息类型为 0 的链表记录了所有消息加入队列的顺序，其中红色箭头表示消息加入的顺序。

图片及简介参考自：[  
简书博主小Q_wang](https://www.jianshu.com/p/7e3045cf1ab8)

# 2 linux 下消息队列查看和删除指令

  * `ipcs --查看进程间通信状态（包括键值、id.....如下）`
  * `ipcs -q --只查看消息队列的信息`
  * `ipcrm -q id --删除指定的消息队列（id为消息队列的标识符）`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806202139941.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

# 3消息队列相关函数

    
    
    #include <sys/types.h>
    #include <sys/ipc.h>
    #include <sys/msg.h>
    // 访问或创建消息队列 并返回消息队列的标识符
    int msgget(key_t key, int msgflg)
    // 将消息发送到消息队列
    int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg)
    // 从消息队列获取消息
    ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);
    // 查看、设置、删除消息队列
    int msgctl(int msqid, int command,struct msqid_ds *buf)
    

## 3.1 msgget函数

**函数作用：**  
访问一个消息队列或创建一个消息队列设置权限，并返回消息队列的标志符  
**参数使用：**

  * `int msgget(key_t key, int msgflg)`
  * 参数一：键值或者理解成暗号，大于0的32位整数 
    * 可以手动输入一个整形
    * 或者通过ftok返回的IPC键值
  * 参数二：IPC_CREAT|0666 表示创建并设置权限  
（后面权限数字不固定）  
IPC_EXCL 表示访问

  * 返回值：成功返回消息队列号， 失败返回0并设置 errno

## 3.2 msgsnd函数

**函数作用：**  
将消息写入到消息队列，或者说发送消息  
**参数使用：**

  * `int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg)`
  * 参数一：消息队列标识符
  * 参数二：发送到队列的信息结构体地址，可以是任意类型的结构体，只要求第一个字段必须为 **long**
  * 参数三：要发送信息的大小，不包含信息类型占用的4个字节
  * 参数四： 
    * 0：若消息队列满了，阻塞等待
    * IPC_NOWAIT：若消息队列满了，不阻塞直接返回
    * IPC_NOERROR：实际信息长于参数三设定，截取部分发送，其余抛弃，不报错
  * 返回值：0 表示成功，-1 失败并设置 errno

## 3.3 msgrcv函数

**函数作用：**  
从消息队列读取信息，每读取一条少一条，否则阻塞等待  
**参数使用：**

  * `ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);`
  * 参数一：消息队列标识符
  * 参数二：存放接受到的信息结构体地址，要求与发送类型一致。
  * 参数三：接收消息大小，不含消息类型占用的4个字节
  * 参数四： 
    * 0：接受第一个消息
    * ‘> 0:接受与参数二设置消息类型相同的第一个消息
    * <0:接收类型等于或者小于参数二设置消息类型的第一个消息
  * 参数五：0: 阻塞式接收消息，不存在则阻塞等待  
其余几种：IPC_NOWAIT IPC_EXCEPT IPC_NOERROR 不多介绍  
返回值：0 表示成功，-1 失败并设置 errno

**发送or接收的信息结构体如下：**

    
    
    struct msgbuf{
        long type; // 必须为long类型 >0
        // 消息正文,可自由设定....  
    };
    ----------------举例---------------
    struct msgbuf {
        long type;
        char text[1024];
    } ;
    

## 3.4 msgctl函数

**函数作用：**  
查看、设置、删除消息队列  
**参数使用：**

  * `int msgctl(int msqid, int command,struct msqid_ds *buf)`

  * 参数一：消息队列标识符

  * 参数二：IPC_STAT:获得msgid的消息队列头数据到buf中  
IPC_RMID：删除消息队列。  
IPC_INFO：读取消息队列基本情况。此命令等同于 ipcs 命令  
IPC_SET：设置消息队列的属性，要设置的属性需  
先存储在buf中，可设置的属性包括：  
(1)msg_perm.uid  
(2)msg_perm.gid  
(3)msg_perm.mode  
(4)msg_qbytes

  * 参数三：消息队列管理结构体

  * 返回值：0 表示成功，-1 失败并设置 errno

**举例**

    
    
    msgctl（msgqid,IPC_RMID,NULL）; 删除
    

# 4 通过fork创建父子进程实现简单互发消息（两个消息队列 实现通信）

**两个文件基本一致 差别在于进程中创建的消息队列不同**

**文件 A.c**

    
    
    #include <sys/types.h>
    #include <unistd.h>
    #include <sys/ipc.h>
    #include <sys/msg.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    
    #define MSGKEY 123//暗号
    #define MSGKEY_NEW 321//暗号
    #define TEXTSIZE 512//长度
    
    //父子进程中两个while存在区别
    //其中一个在不断等待输入发送
    //另一个把访问msgget放在循环中，若访问到了才开始接受信息，注意区别。
    
    struct msgbuf
    {
    	long msg_type;//类型
    	char msg_text[TEXTSIZE];//文本长度
    	
    };
    int main(int argc,char *argv[])
    {
    	pid_t pid;//进程号
    	int qid;//消息队列号 两个进程都有就放在最上面
    	pid=fork();
    	if(pid>0)
    	{
    		char str[500];
    		struct msgbuf msgbuf_send;
    		qid=msgget(MSGKEY,IPC_EXCL);
    		if(qid<0)  //先判断存不存在 不存在再创建
    		{
    			qid =msgget(MSGKEY,IPC_CREAT|0666);//创建一个消息队列 设置权限
    		}
    		printf("input messsage send\n");
    		while(1)//不断发送消息
    		{
    			scanf("%s",str);
    			msgbuf_send.msg_type=1;
    			strcpy(msgbuf_send.msg_text,str);
    			msgsnd(qid,&msgbuf_send,strlen(msgbuf_send.msg_text),0);	
    			
    		}
    		msgctl(qid,IPC_RMID,NULL);
    		
    	}
    	else if(pid==0)
    	{
    		struct msgbuf msgbuf_receive;
    		while(1)//获取消息队列 不在则不断循环判断等待创建出现
    		{
    			
    			if((qid=msgget(MSGKEY_NEW,IPC_EXCL))<0)
    			{
    				sleep(2);
    				continue;
    			}
    			
    			msgrcv(qid,&msgbuf_receive,sizeof(msgbuf_receive.msg_text),0,0);
    			printf("receive message: %s\n",(&msgbuf_receive)->msg_text);
    		}
    		msgctl(qid,IPC_RMID,NULL);
    		
    	}
    	else
    	{
    		perror("fork");
    	}
    	
    	
    	return 0;
    }
    
    
    

文件 B.c

    
    
    #include <sys/types.h>
    #include <unistd.h>
    #include <sys/ipc.h>
    #include <sys/msg.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    
    #define MSGKEY 123//暗号
    #define MSGKEY_NEW 321//暗号
    #define TEXTSIZE 512//长度
    
    //父子进程中两个while存在区别
    //其中一个在不断等待输入发送
    //另一个把访问msgget放在循环中，若访问到了才开始接受信息，注意区别。
    
    struct msgbuf
    {
    	long msg_type;//类型
    	char msg_text[TEXTSIZE];//文本长度
    	
    };
    int main(int argc,char *argv[])
    {
    	pid_t pid;//进程号
    	int qid;//消息队列号 两个进程都有就放在最上面
    	pid=fork();
    	if(pid>0)
    	{
    		char str[500];
    		struct msgbuf msgbuf_send;
    		qid=msgget(MSGKEY_NEW,IPC_EXCL); 
    		if(qid<0) //先判断存不存在 不存在再创建
    		{
    			qid =msgget(MSGKEY_NEW,IPC_CREAT|0666);//创建一个消息队列 设置权限
    		}
    		printf("input messsage send\n");
    		while(1)//不断发送消息
    		{
    			scanf("%s",str);
    			msgbuf_send.msg_type=1;
    			strcpy(msgbuf_send.msg_text,str);
    			msgsnd(qid,&msgbuf_send,strlen(msgbuf_send.msg_text),0);	
    			
    		}
    		msgctl(qid,IPC_RMID,NULL);
    		
    	}
    	else if(pid==0)
    	{
    		struct msgbuf msgbuf_receive;
    		while(1)//获取消息队列 不在则不断循环判断等待创建出现
    		{
    			
    			if((qid=msgget(MSGKEY,IPC_EXCL))<0)
    			{
    				sleep(2);//等待两秒再次查询一下该消息队列在不在
    				continue;
    			}
    			msgrcv(qid,&msgbuf_receive,sizeof(msgbuf_receive.msg_text),0,0);
    			printf("receive message:  %s\n",(&msgbuf_receive)->msg_text);
    		}
    		msgctl(qid,IPC_RMID,NULL);
    		
    	}
    	else
    	{
    		perror("fork");
    	}
    	
    	
    	return 0;
    }
    
    

# 补充-ipcs指令

>   * 在unix/linux下，查看共享内存、信号量，队列等共享信息 相应的命令是`ipcs [-m|-s|-q]`  
>  `-m`列出共享内存，`-s`列出共享信号量，`-q`列出共享队列
>
>   * 清除命令是 `ipcrm [-m|-s|-q] $id`  
>  `-m` 删除共享内存，`-s`删除共享信号量，`-q`删除共享队列
>
>

# 补充 -小点知识

  1. 发送一条信息，消息队列数目+1
  2. 接受函数执行一次取一条，数量-1 如果没有信息会等待
  3. 关机内存清楚，消息队列不存在
  4. 通过fork可以实现两个进程的聊天
  5. 可以指定发送/接受指定类型的消息
  6. 消息队列分成三块内容：信息类型，消息长度，消息内容，
  7. Cat /proc/sys/kernel/msgmax 查看消息队列信息长度
  8. Cat /proc/sys/kernel/msgmni 查看有多少个消息队列
  9. 好处就是可以定义类型

# 总结

> 如有错误，欢迎指出

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406091722343.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

