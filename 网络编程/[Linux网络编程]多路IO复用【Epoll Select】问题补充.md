> 对于epoll模型和select模型的补充，总结了一些忽视或者说高频的问题.  
>  `修正时间：10-06`

### 文章目录

  * 什么是IO多路复用
  * IO阻塞(BIO)模型
  * IO非阻塞（NIO）模型
  * IO复用的三种方式及其各自优缺点
  * epoll LT 与 ET模型的区别
  * 补充-再探epoll和select流程（重要）
  * 信号驱动IO和异步IO

# 什么是IO多路复用

  * IO多路复用是一种同步IO模型，实现一个线程可以监视多个文件句柄；一旦某个文件句柄就绪，就能够通知应用程序进行相应的读写操作；没有文件句柄就绪时会阻塞应用程序，交出cpu。多路是指网络连接，复用指的是同一个线程。（通俗的就是找一个秘书单独监督事件发生，再把产生动静的告诉我就好，不需要我自己去问。）

# IO阻塞(BIO)模型

  * 这是最常用的简单的IO模型。阻塞IO意味着当我们发起一次IO操作后一直等待成功或失败之后才返回，在这期间程序不能做其它的事情。阻塞IO操作只能对单个文件描述符进行操作。
  * 换个说法：服务端采用单线程，当accept一个请求后，在recv或send调用阻塞时，将无法accept其他请求（必须等上一个请求处recv或send完），无法处理并发

    
    
    // 伪代码描述
    while(1) {
      // accept阻塞
      client_fd = accept(listen_fd)
      fds.append(client_fd)
      for (fd in fds) {
        // recv阻塞（会影响上面的accept）
        if (recv(fd)) {
          // logic
        }
      }  
    }
    

  * 服务器端采用多线程，当accept一个请求后，开启线程进行recv，可以完成并发处理，但随着请求数增加需要增加系统线程，大量的线程占用很大的内存空间，并且线程切换会带来很大的开销，10000个线程真正发生读写事件的线程数不会超过20%，每次accept都开一个线程也是一种资源浪费

    
    
    // 伪代码描述
    while(1) {
      // accept阻塞
      client_fd = accept(listen_fd)
      // 开启线程read数据（fd增多导致线程数增多）
      new Thread func() {
        // recv阻塞（多线程不影响上面的accept）
        if (recv(fd)) {
          // logic
        }
      }  
    }
    
    

# IO非阻塞（NIO）模型

  * 我们在发起IO时，通过对文件描述符设置O_NONBLOCK flag来指定该文件描述符的IO操作为非阻塞。非阻塞IO通常发生在一个for循环当中，因为`每次进行IO操作时要么IO操作成功，要么当IO操作会阻塞时返回错误EWOULDBLOCK/EAGAIN`，然后再根据需要进行下一次的for循环操作，这种类似轮询的方式会浪费很多不必要的CPU资源，是一种糟糕的设计。
  * 换个说法：服务器端当accept一个请求后，加入fds集合，每次轮询一遍fds集合recv(非阻塞)数据，没有数据则立即返回错误，每次轮询所有fd（包括没有发生读写事件的fd）会很浪费cpu

    
    
    setNonblocking(listen_fd)
    // 伪代码描述
    while(1) {
      // accept非阻塞（cpu一直忙轮询）
      client_fd = accept(listen_fd)
      if (client_fd != null) {
        // 有人连接
        fds.append(client_fd)
      } else {
        // 无人连接
      }  
      for (fd in fds) {
        // recv非阻塞
        setNonblocking(client_fd)
        // recv 为非阻塞命令
        if (len = recv(fd) && len > 0) {
          // 有读写数据
          // logic
        } else {
           无读写数据
        }
      }  
    }
    

# IO复用的三种方式及其各自优缺点

>
> 简单点说：select和epoll模型最大的区别在于（也是效率差别主要在于），epoll能知道哪些监听的文件句柄有读写请求，而select更像是告诉你有读写事件但是你要自己去比对是哪几个。

  * select：[[Linux网络编程]高并发-Select模型](https://blog.csdn.net/weixin_44972997/article/details/108004370)
  * poll：待补充
  * epoll：[[Linux网络编程]高并发-Epoll模型](https://blog.csdn.net/weixin_44972997/article/details/108921767)

两者区别：

  1. epoll不存在集合的覆盖 epoll_create会返回一个fd，指向空间包含全部的事件（结构体）

  2. epoll把要监听的每一个fd都包装成一个事件，并把这个事件记入epollfd 让epollfd来监听

  3. select产生动静是吧fd放入集合 但是epoll通过epoll_wait 把产生动静的fd所包装好的事件放入结构体数组

  4. select需要备份，需要重新创建数组放fd循环比对，epoll直接通过包装好的事件（结构体）就能获得fd，效率也更快（差别主要体现在这）

  5. 两者的区别是的select适合用户客服端不多的情况，而epoll没有客户端的上限

select缺点

  1. 最大并发数限制，因为一个进程所打开的 fd（文件描述符）是有限制的，由 FD_SETSIZE 设置，默认值是 1024，并且集合描述符最大也只能为1024，因此 select 模型的最大并发数就被相应限制了。

  2. 效率问题，采用循环的方式匹配数组内的fd是否在产生的动静集合中，如果连接的客户端数量很多，那么效率可想而知。

  3. 每调用一次select 就需要多个事件类型的fd_set需从用户空间拷贝到内核空间去，返回时select也会把保留了活跃事件的返回(从内核拷贝到用户空间)。当fd_set数据大的时候，这个过程消耗是很大的。

`总结：`前面两点缺陷在代码中都有体现。

简单例子：

  * [参考链接](https://www.zhihu.com/search?type=content&q=IO%E5%A4%8D%E7%94%A8%E5%A6%82%E4%BD%95%E5%9B%9E%E7%AD%94)  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/f6fa9e899659422998c98bf0e2dd7e49.png?x-oss-
process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAV2luZGFsb3Zl,size_20,color_FFFFFF,t_70,g_se,x_16)

# epoll LT 与 ET模型的区别

  * [参考1 有代码](https://zhuanlan.zhihu.com/p/107995399)

epoll水平触发: 只要监听的文件描述符中有数据，就会触发epoll_wait有返回值，这是默认的epoll_wait的方式;

epoll边沿触发 : 只有监听的文件描述符的读/写事件发生，才会触发epoll_wait有返回值;

通过epoll_ctl函数，设置该文件描述符的触发状态即可

    
    
    //水平触发
    evt.events = EPOLLIN;    // LT 水平触发 (默认) EPOLLLT
    evt.data.fd = pfd[0];
    
    //边沿触发
    evt.events = EPOLLIN | EPOLLET;    // ET 边沿触发
    evt.data.fd = pfd[0];
    

  * 管道+epoll的例子  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/61e1be456f114664b6bf3eff0c1ae2a7.png?x-oss-
process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAV2luZGFsb3Zl,size_20,color_FFFFFF,t_70,g_se,x_16)  
![在这里插入图片描述](https://img-
blog.csdnimg.cn/4a84d546b4e44eb5a45ed0b2f7d7548f.png?x-oss-
process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAV2luZGFsb3Zl,size_20,color_FFFFFF,t_70,g_se,x_16)

  * 换一种说法：[参考2](https://blog.csdn.net/wangquan1992/article/details/105957575?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163301242916780274179937%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=163301242916780274179937&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-1-105957575.pc_search_result_hbase_insert&utm_term=socket%20epoll%E7%9A%84%E8%BE%B9%E7%BC%98%E8%A7%A6%E5%8F%91%E5%92%8C%E6%B0%B4%E5%B9%B3%E8%A7%A6%E5%8F%91&spm=1018.2226.3001.4187)

  * Level_triggered(水平触发)：当被监控的文件描述符上有可读写事件发生时，`epoll_wait()会通知处理程序去读写。如果这次没有把数据一次性全部读写完(如读写缓冲区太小)，那么下次调用 epoll_wait()时，它还会通知你在上没读写完的文件描述符上继续读写，当然如果你一直不去读写，它会一直通知你！！！`如果系统中有大量你不需要读写的就绪文件描述符，而它们每次都会返回，这样会大大降低处理程序检索自己关心的就绪文件描述符的效率！！！

  * Edge_triggered(边缘触发)：当被监控的文件描述符上有可读写事件发生时`，epoll_wait()会通知处理程序去读写。如果这次没有把数据全部读写完(如读写缓冲区太小)，那么下次调用epoll_wait()时，它不会通知你，也就是它只会通知你一次，直到该文件描述符上出现第二次可读写事件才会通知你（根据上一个说法 数据应该还是在的）！！！`这种模式比水平触发效率高，系统不会充斥大量你不关心的就绪文件描述符！！！

![在这里插入图片描述](https://img-
blog.csdnimg.cn/f92f175822694962b1c08b6c084ce77e.png?x-oss-
process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAV2luZGFsb3Zl,size_20,color_FFFFFF,t_70,g_se,x_16)

**注意点**
：ET模式下，它只会提示一次，直到下次再有数据流入之前都不会再提示了，无论fd中是否还有数据可读。所以在ET模式下，read一个fd的时候一定要把它的buffer读完，或者遇到EAGAIN错误

# 补充-再探epoll和select流程（重要）

  * [这个写的非常好](https://zhuanlan.zhihu.com/p/126278747)

  * select更在细致的执行流程

    1. 在调用select之前告诉select 应用进程需要监控哪些fd可读、可写、异常事件，这些分别都存在一个fd_set数组中。
    2. 然后应用进程调用select的时候把3个fd_set传给内核（这里也就产生了一次fd_set在`用户空间到内核空间的复制`），内核收到fd_set后对fd_set进行遍历，然后一个个去扫描对应fd是否满足可读写事件。
    3. 如果发现了有对应的fd有读写事件后，内核会把fd_set里`没有事件状态的fd句柄清除`，然后把有事件的fd返回给应用进程（这里又会把fd_set从`内核空间复制用户空间`）。
    4. 最后应用进程收到了select返回的活跃事件类型的fd句柄后，`再向对应的fd发起数据读取或者写入数据操作`。
  * epoll更在细致的执行流程

    1. 创建内核事件表（epoll_create）。这里主要是向内核申请创建一个fd的文件描述符作为内核事件表（`B+树结构的文件`，没有数量限制），这个描述符用来保存应用进程需要监控哪些fd和对应类型的事件。 （`简单理解内核申请一个B+树来监听事件`）
    2. 添加或移出监控的fd和事件类型（epoll_ctl）。调用此方法可以是向内核的内核事件表 动态的添加和移出fd 和对应事件类型。
    3. epoll_wait 绑定回调事件：`内核向事件表的fd绑定一个回调函数`。当监控的fd活跃时，会调用callback函数把事件加到一个活跃事件队列里;最后在epoll_wait 返回的时候内核会把活跃事件队列里的fd和事件类型返回给应用进程。
  * 总结

    * 最后，从epoll整体思路上来看，采用事先就在内核创建一个事件监听表，后面只需要往里面添加移出对应事件，因为本身事件表就在内核空间，所以就避免了向select、poll一样每次都要把自己需要监听的事件列表传输过去，然后又传回来，这也就`避免了事件信息需要在用户空间和内核空间相互拷贝的问题`。

    * 然后epoll并不是像select一样去遍历事件列表，然后逐个轮询的监控fd的事件状态，而是事先就建立了fd与之对应的回调函数，`当事件激活后主动回调callback函数，这也就避免了遍历事件列表的这个操作`,所以epoll并不会像select和poll一样随着监控的fd变多而效率降低，这种事件机制也是epoll要比select和poll高效的主要原因。

# 信号驱动IO和异步IO

  * [参考链接 ](https://www.jianshu.com/p/486b0965c296)

