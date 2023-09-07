> 距离上一次利用高并发技术实现360度行车记录仪功能已经过去半年了。开始写一系列关于系统编程和网络编程内容进行总结。  
>  温故而知新，欢迎大家讨论学习。

### 文章目录

  * 1 系统调用
  *     * 1.1 什么是系统调用
    * 1.2什么是库函数
    * 1.3 将hello写入到文件1.txt流程
    * 1.4 为什么要有缓冲区（补充）
    * 1.5 内核缓冲区和C标准缓冲区的区别
  * 2 open()文件操作函数
  *     * 2.1 函数原型
    * 2.2 参数描述
    * 2.3 代码举例
  * 3 write()文件操作函数
  *     * 3.1 函数原型
    * 3.2 参数使用
    * 3.3 代码举例
  * 4 read()文件操作函数
  *     * 4.1 函数原型
    * 4.2 参数使用
    * 4.3 代码举例
  * 5 close()文件操作函数
  *     * 5.1 函数原型
    * 5.2 参数使用
    * 5.3 代码举例
  * 6 lseek()文件操作函数
  *     * 6.1 函数原型
    * 6.2 参数使用
    * 6.3 三种使用举例（重）
  * 7 虚拟空间地址/PCB进程控制块/文件描述符表
  *     * 7.1 概念+图
    * 7.2 最大打开文件数
    * 7.3 阻塞和非阻塞(常见)
  * 补充-strcpy和strcnpy
  * 补充-linux man 1 2 3的作用
  * 补充-父子进程读写文件
  * 总结

# 1 系统调用

## 1.1 什么是系统调用

系统调用函数属于操作系统的一部分，是为了提供给用户进行操作的接口（API函数），使得用户态运行的进程与硬件设备(如CPU、磁盘、打印机、显示器)等进行交互。

  * 例如常见的系统调用 等等`write read open` …

## 1.2 什么是库函数

  1. 库函数可分为两类，一类是C语言标准库函数，一类是编译器特定的库函数。
  2. 库函数可以理解为是对系统调用函数的一层封装。尽管系统函数执行效率是比较高效而精简的，但有时我们需要对获取的信息进行更复杂的处理，或更人性化的需要，我们把这些处理过程封装成一个函数，再将许多这类的函数放在一个文件（库）一般放在 .lib文件。最后再供程序员使用。(自己封装的为动态库和静态库)

  * `#include<stdio.h>`使用的时候包含头文件就可以使用其中的库函数了
  * 例如常见的库函数`printf fwrite fread fopen`…等等

## 1.3 将hello写入到文件1.txt流程

    1. 首先fopen打开文件 fwrite参数附上要写入的内容
    2. 文本内容来到C标准缓冲区
    3. 如果满足条件就刷新C标准缓冲区，调用系统函数write进行写（补充：满了就会自动刷新）
    4. write却只是把要写入的内容写到内核缓冲区
    5. 如果内核缓冲区满足条件就刷新内核缓冲区，系统调用sys_write将缓冲区内容写入到磁盘（补充：有个进程会定时刷新内核缓冲区）
    6. 此时有进程读取1.txt文件内容，发现内核缓冲区就有这个文件内容，就直接从内核缓冲区读取

![image-20230316141754214](D:\MarkDown\picture\image-20230316141754214.png)

## 1.4 为什么要有缓冲区（补充）

**定义** ：缓冲区就是内存里的一块区域，把数据先存内存里，然后一次性写入硬盘中的文件，类似于数据库的批量操作。  
**好处** ：减少对硬盘的直接操作，硬盘的执行速度为毫秒级别，内存为纳秒级别。在硬盘直接操作读写效率太低。

## 1.5 内核缓冲区和C标准缓冲区的区别

C语言标准库函数fopen()每打开一个文件时候，其都会对应一个 **单独** 一个缓冲区而内核缓冲区是 **公用的** 。

## 1.6 文件编程概述

Linux操作系统提供一系列的API

* 打开 open
* 写 读 write read
* 光标定位 lseek
* 关闭 close

# 2 open()文件操作函数

## 2.1 函数原型

  * `int open(const char *pathname, int flags);`
  * `int open(const char *pathname, int flags, mode_t mode);`

## 2.2 参数描述

  * `pathname` ：路径+文件名
  * `flats` ： 操作模式 （`O_CREAT创建 O_RDONLY只读 O_WRONLY只写 O_RDWR可读可写`）
    当然也有组合使用`（O_RDONLY ｜O_CREAT 只读如果不存在则创建）(O_WRONLY ｜O_CREAT 只写如果不存在则创建)(O_WRONLY | O_APPEND 文件存在则追加写入)`
  * `mode`:当创建文件的时候需要在此位置设置权限。这个可以不需要掌握
  * 返回值: 失败返回`-1`,成功返回 整形（文件描述符）

## 2.3 代码举例

```cpp
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

#include<stdio.h>
#include <unistd.h>  // 提供对POSIX操作系统API的访问功能

int main()
{
    int fd;//文件描述符,为open()的返回值
    //以只写的方式打开God.txt这个文件
    // fd = open("./tmp.txt", O_WRONLY)；
    // 以追加写入的方式打开tmp.txt 这个文件
    // fd = open("./tmp.txt",O_WRONLY|O_APPEND);
    //只读的方式打开Fol.txt,若不存在则创建并设置权限为0777（表示可读可写可执行
    // fd = open("./Fol.txt",O_RDONLY|O_CREAT,0777);
    fd = open("./file1",O_RDWR);         //可读可写的方式打开，字符串本身就是一个指针
    if(fd == -1) {     //如果fd的返回值为-1
        printf("open file1 failed\n");
        fd = open("./file1",O_RDWR|O_CREAT, 0600);//如果没有文件file1，则创建一个file1
        if(fd > 0) {
            printf(f_attrib);
            printf("create file1 success!\n");
        }
    }
    close(fd);//关闭fd
    return 0;
} 
```


​    

# 3 write()文件操作函数

## 3.1 函数原型

  * `ssize_t write(int fd, const void *buf, size_t count);`

## 3.2 参数使用

  * `fd` 表示文件的文件描述符，即open函数的返回值
  * `buf` 写入的文本内容
  * `count` 写入的数据长度【使用srtlen非sizeof】简单理解就是将buf里面有count个字节的数据写入到fd指向的文件中去
  * 返回值 成功返回写入的字节数 失败返回-1

## 3.3 代码举例

```cpp
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include<stdio.h>
#include <unistd.h>
#include <string.h>

int main()
{
    int fd;//文件描述符,为open()的返回值
    char *buf = "I am ok!!!";
    // char buf[]="Welcome come to JMU";
    fd = open("./file1.txt",O_RDWR);//打开一个file1文件，如果文件不存在，则fd为-1
    if(fd == -1)
    {
        printf("open file1 failed\n");
        fd = open("./file1.txt",O_RDWR|O_CREAT,0600);//如果没有文件file1，则创建一个file1
        if(fd > 0)
        {
            printf("create file1 success!\n");
        }
    }
    write(fd,buf,strlen(buf));//把buf中的内容写入file1，
    close(fd);//关闭文件
    return 0;
}
```


# 4 read()文件操作函数

## 4.1 函数原型

  * `ssize_t read(int fd, void *buf, size_t count);`

## 4.2 参数使用

  * `.fd` 表示改文件的文件描述符，open的返回值
  * .`buf` 指缓冲区，读取的数据存放位置
  * .`count` 传入缓冲区的字节大小【使用sizeof非strlen】
  * .返回值 成功返回读出的字节数 失败返回-1
  * （值得注意的是也可能是以非阻塞的方式读一个设备文件和网络文件，后面网络编程常见）

## 4.3 代码举例

```cpp
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include<stdio.h>
#include <unistd.h>
#include <string.h>
 #include <stdlib.h>
int main()
{
        int fd;//文件描述符,为open()的返回值
        char *buf = "I am ok!!!";
        fd = open("./file1",O_RDWR);
        if(fd == -1)
        {
                printf("open file1 failed\n");
                fd = open("./file1",O_RDWR|O_CREAT,0600);//如果没有文件file1，则创建一个file1
                if(fd > 0)
                {
                        printf("create file1 success!\n");
                }
        }
        int n_write = write(fd,buf,strlen(buf));//如果写入成功，则返回写入字节个数
        if(n_write != -1)
        {
                printf("write %d byte to file1\n",n_write);
        }
		// 这里关闭fd,然后重新打开文件，是为了让光标重新回到开头
        close(fd); 
        fd = open("./file1",O_RDWR);

        char *readBuf;//要开辟空间，否则会产生段错误
        readBuf = (char *)malloc(sizeof(char)*n_write+1);//开辟readbuf空间

        int n_read = read(fd,readBuf,n_write);//如果读取成功，则返回读取字节个数

        printf("read %d,context:%s\n",n_read,readBuf);//打印出读取的字节个数和内容
        close(fd);

        return 0;
}
```


# 5 close()文件操作函数

## 5.1 函数原型

  * .`int close(int fd);`

## 5.2 参数使用

  * .fd 表示改文件的文件描述符，open的返回值
  * .返回值 成功为0 失败返回-1

## 5.3 代码举例

```c
 int  fd;
 fd=open("tmp.txt",O_RDONLY);
 close（fd）；
```


# 6 lseek()文件操作函数

## 6.1 函数原型

  * .`offt lseek(int fd, off_t offset, int whence);`

## 6.2 参数使用

  * `.fd` 表示改文件的文件描述符，open的返回值
  * .`offset` 表示偏移量，将文件读写指针相对whence移动offset个字节
  * .`whence` 指出偏移的方式
**whence参数补充说明**
`SEEK_SET`:偏移到文件头+ 设置的偏移量 
`SEEK_CUR`：偏移到当前位置+设置的偏移量 
`SEEK_END`：偏移到文件尾置+设置的偏移量

## 6.3 三种使用举例（重）

（1） 返回当前的偏移量

```cpp
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include<stdio.h>
#include <unistd.h>
#include <string.h>

int main()
{
    int fd,ret;
    fd = open("file1.txt", O_RDWR);
    ret = lseek(fd, 0, SEEK_CUR);
    printf("%d\n",ret);
    close(fd);
}
```


（2）返回文件大小（就是将光标移动到了文件尾

```cpp
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include<stdio.h>
#include <unistd.h>
#include <string.h>

int main()
{
    int fd,ret;
    fd = open("file1.txt", O_RDWR);
    ret = lseek(fd, 0, SEEK_END);
    printf("%d\n",ret);
}
```

（3）☆扩充文件大小 
**特别注意扩充文件大小后 需要写入内容 否则扩充不生效**

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include<stdio.h>
#include <unistd.h>
#include <string.h>

int main()
{
    int fd,ret;
    char a[]="JMU WELCOME";
    fd = open("file1.txt", O_RDWR);
    ret = lseek(fd, 1000, SEEK_END);
    write(fd,a,strlen(a));
    printf("%d\n",ret);
}
```



```cpp
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include<stdio.h>
#include <unistd.h>
#include <string.h>
 #include <stdlib.h>
int main()
{
    int fd;
    char *buf = "I am ok!!!";
    fd = open("./file1.txt",O_RDWR);

    if(fd == -1)
    {
        printf("open file1 failed\n");
        fd = open("./file1",O_RDWR|O_CREAT,0600);
        if(fd > 0)
        {
            printf("create file1 success!\n");
        }
    }
    int n_write = write(fd,buf,strlen(buf));
    if(n_write != -1)
    {
        printf("write %d byte to file1\n",n_write);
    }
    // close(fd);
    //fd = open("./file1",O_RDWR);    
    lseek(fd,0,SEEK_SET);//替换上面两句，重新把光标放到开头，相当于相对于文件开头偏移0个字节
    char *readBuf;
    readBuf = (char *)malloc(sizeof(char)*n_write+1);//开辟空间

    int n_read = read(fd,readBuf,n_write);

    printf("read %d,context:%s\n",n_read,readBuf);
    close(fd);

    return 0;

}
```

# 7 creat()

![image-20230316151232177](D:\MarkDown\picture\image-20230316151232177.png)

```c
//Test5.c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include<stdio.h>
#include<string.h>
int main()
{
        int fd;
        char *buf = "I will be successful\n";

        fd = creat("./file1",S_IRWXU);   //可读可写可执行

        close(fd);
        return 0;
}
```

![image-20230316151340831](D:\MarkDown\picture\image-20230316151340831.png)

# 7 虚拟空间地址/PCB进程控制块/文件描述符表

## 7.1 概念+图

对于每一个进程，系统都会为其分配一个0-4G的虚拟空间地址，0-3G为用户空间，用户可操作部分。3~4G为内核，其中PCB控制块也存在内核中（补充：每一个进程都有一个PCB）PCB结构体包括文件标识符表以及其他很多信息。  
文件描述符表,结构体 PCB 的成员变量 file_struct *file 指向文件描述符表。从应用程序使用角度，该指针可理解记忆成一个字符指针数组，下标
0/1/2/3/4…找到文件结构体。本质是一个键值对
0、1、2…都分别对应具体地址。但键值对使用的特性是自动映射，我们只操作键不直接使用值。新打开文件返回文件描述符表中未使用的最小文件描述符。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210408205735225.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210408205449204.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

## 7.2 最大打开文件数

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021040820582761.png)

## 7.3 阻塞和非阻塞(常见)

产生阻塞的场景，读设备文件，读网络文件（后面网络编程非常常见）。

# 补充-strcpy和strcnpy

  * `char *strcpy(cnost char *src, char *dst);`
  * `char *strncpy(char *dest, char *src, size_t num);`

strncpy并没有拷贝串后的\0字符，而strcpy却拷贝了。这充分说明，strncpy是为拷贝字符而生的，而strcpy是拷贝字符串而生的。但两者都不能越界拷贝。只要正确使用strncpy,
那就比strcpy安全。


​    
    #include <iostream>
    using namespace std;
     
    int main()
    {
    	char str[4] = "xyz";
    	str[3] = 'w'; //故意将最后的'\0'换成'w'
    	
    	char *p = "abc";
    	strncpy(str, p, sizeof(str) - 1); //只拷贝了3个字符，没有拷贝'\0'
    	cout << str << endl;
     
    	return 0;
    }


常见错误：输出abcw，所以，拷贝的时候需要对串进行清零处理，memset

# 补充-linux man 1 2 3的作用


​    
    1、Standard commands （标准命令）
    2、System calls （系统调用）
    3、Library functions （库函数）
    4、Special devices （设备说明）
    5、File formats （文件格式）
    6、Games and toys （游戏和娱乐）
    7、Miscellaneous （杂项）
    8、Administrative Commands （管理员命令）
    9 其他（Linux特定的）， 用来存放内核例行程序的文档。


例如  
man 1 ls  
man 2 open  
man 3 printf

# 补充-父子进程读写文件


​    
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <fcntl.h>
    #include <stdio.h>
    #include <unistd.h>
    #include <wait.h>
    #include <string.h> 
    int main(int argc,char *argv[])
    {
    	pid_t pid;
    	pid =fork();
        int fd ;
    	fd =open("tmp.txt",O_RDWR|O_CREAT,0777);
    	if(fd<0)
    	{
    		perror("open");
    		return -1;
            } 
    	if(pid>0)
    	{
    		char buf[128];
    		printf("parentʼ\n");
    		while(1)
    		{
    			memset(buf,0,sizeof(buf));//��� buf
    			fgets(buf,sizeof(buf),stdin);
    			printf("input:%s\n",buf);
    			if(write(fd,buf,strlen(buf))<0)
    			perror("write");			
    		}
    	}
    	else if(pid==0)
    	{
    		char buf[128];
    		printf("child\n");
    		while(1)
    		{       
                int len;
    			memset(buf,0,sizeof(buf));
    			len =read(fd,buf,sizeof(buf));
    			if(len>0)
    			{
    				printf("child read:%s",buf);
    			}
    			sleep(1);			
    		} 
    	}
    	else
    	{
    		perror("fork");
    	}	
    }


# 总结

> 如有错误欢迎指出…

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406091722343.png?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

