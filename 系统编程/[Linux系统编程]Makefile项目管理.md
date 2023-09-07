> 分别放在若干个目录中，makefile定义了一系列的规则
> 来指定哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，因为
> makefile就像一个`Shell脚本`一样，也可以执行操作系统的命令。–引自[百科词条](https://baike.baidu.com/item/Makefile/4619787?fr=aladdin)  
>  [往期文章链接](https://blog.csdn.net/weixin_44972997/article/details/115869213)  
>  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210427085303460.png?x-oss-
> process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

### 文章目录

  * 1 学习目标
  * 2 makefile概述
  * 3 目标、依赖、处理动作的概念（重要）
  * 4 makefile 基础规则
  *     * 4.1 一个规则的概念
    * 4.2 两个函数的概念
    * 4.3 三个自动变量的概念
  * 5 makefile 基础规则实例解释
  *     * 5.1 一个规则演示（加减乘除多文件编译）
    *       * 5.1.1 代码部分
      * 5.1.2 结果
      * 5.1.3 利用规则改进
    * 5.2 补充-ALL指定输出目标
    * 5.3 两个函数演示
    * 5.4 补充-clean用法
    * 5.5 三个自动变量用法
    * 5.6 补充-模板规则
    * 5.7 补充-静态模板规则
    * 5.8 补充-伪目标
    * 5.9补充-第三方函数的添加
  * 小点补充
  * 总结

# 1 学习目标

>
> 网上对于`makefile`总结学习文章有许多，但是大部分排版或者内容杂乱无章，所以还是自己进行一个总结学习，大家可以根据自己的阅读习惯选择适合自己的文章。  
>  在 **实例部分** 的代码是层层递进改变，每一次都是在之前的基础上修改。

  * 了解Make工具给我们带来的好处和便利
  * makefile的书写格式、关键字、函数。像C 语言有自己的格式、关键字和函数一样（ **重点** ）

# 2 makefile概述

make工具是一个根据makefile文件内容，针对目标（可执行文件）进行依赖性检测（要生成该可执行文件之前要有哪些中间文件）并执行相关动作（编译等）的工具
。而这个makefile文件类似一个脚本，其中内容包含make所要进行的处理动作以及依赖关系。

# 3 目标、依赖、处理动作的概念（重要）

>
> 学习make工具，需要明白三个概念：`目标、依赖、处理动作`。makefile所要进行的主要内容是明确目标、明确目标所依赖的内容、明确依赖条件满足时应该执行对应的处理动作。例如我们最终要实现a这个目标，但是需要依赖b，而b依赖于c的存在，则可以描述为：
    
    
    a:b
    	cmdbtoa
    b:c
    	cmdctob
    //---------------中文表示如下--------------
    目标：依赖条件
    （一个tab缩进）命令
    目标：依赖条件
    （一个tab缩进）命令
    

  * " : "代表依赖，另外每个处理动作之前都要用tab键分隔。
  * 上述四行的意思是：a依赖于b，而处理cmdbtoa；b依赖于c，而处理cmdctob。

# 4 makefile 基础规则

> 在网上一个比较好的视频学习中总结了一个口诀规则：
>
>   1. 一个规则
>   2. 二个函数
>   3. 三个自动变量
>

## 4.1 一个规则的概念

  1. 目标的时间必须晚于依赖条件的时间，否则，更新目标
  2. 依赖条件如果不存在，找寻新的规则去产生依赖条件。
  3. makefile的依赖是从上至下的，换句话说目标文件是第一句里的目标，如果不满足执行依赖，就会继续向下执行。如果满足生成目标的依赖，就不会再继续向下执行了。make会自动寻找规则里需要的材料文件，执行规则下面的行为生成规则中的目标。

## 4.2 两个函数的概念

> 为了方便学习，我们可以把
>
>   * `wildcard`、`patsubst`看做C语言的函数名。
>   * `src`、`obj` 看成变量名（表示函数的返回值）
>   * 函数名后面跟着的`./*.c %.c, %.o`看做函数的参数，根据顺序可以看做参数一、参数二、参数三
>

  * `src = $(wildcard ./*.c):` **作用** ：匹配当前工作目录下的所有.c 文件。将文件名组成列表，赋值给变量 src。 **等价于** `src = add.c sub.c div1.c`
  * `obj = $(patsubst %.c, %.o, $(src)):` **作用** 将参数 3 中，包含参数 1 的部分，替换为参数 2。 **等价于**`obj = add.o sub.o div1.o`

## 4.3 三个自动变量的概念

  * `$@`: 在规则的命令中，表示规则中的目标。
  * `$^`: 在规则的命令中，表示所有依赖条件。
  * `$<`: 在规则的命令中，表示第一个依赖条件。如果将该变量应用在模式规则中，它可将依赖条件列表中的依赖依次取出，套用模式规则。

# 5 makefile 基础规则实例解释

## 5.1 一个规则演示（加减乘除多文件编译）

> 演示利用makefile演示多文件编译，把加减乘除的函数分别放在不同的文件中，hello.c进行调用执行。

### 5.1.1 代码部分

**makefile**

    
    
    hello:hello.c add.c sub.c div1.c
    	gcc hello.c add.c sub.c div1.c -o hello
    
    

**hello.c**

    
    
    # include<stdio.h>
    
    int add(int,int);
    int sub(int,int);
    int div1(int,int);
    int main(int argc,char*argv[])
    {
    	int a=1,b=2;
    	printf("%d+%d=%d\n",a,b,add(a,b));
    	printf("%d-%d=%d\n",a,b,add(a,b));
    	printf("%d+%d=%d\n",a,b,add(a,b));
    	printf("over\n");
    	return 0;
    
    }
    

**add.c**

    
    
    # include<stdio.h>
    int sub(int a,int b)
    {
    	int c=a+b;
    	return c;
    }
    

**sub.c**

    
    
    # include<stdio.h>
    int sub(int a,int b)
    {
    	int c=a-b;
    	return c;
    }
    

**div1.c**

    
    
    # include<stdio.h>
    int sub(int a,int b)
    {
    	int c=a/b;
    	return c;
    }
    

### 5.1.2 结果

**演示结果**  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210426111854952.png)

### 5.1.3 利用规则改进

  * 这样带来的问题就是当你其中一个函数修改的时候，那么全部的文件都要重新编译，连接。如果每个文件单独执行需要1小时，全部重新编译时间就非常长了，我们完全可以让修改的文件执行生成`.o`文件。再跟其他的.o文件链接就可以了。

补充：不懂的预处理-编译-汇编-链接 查看往期文章

**修改makefile**

    
    
    hello:hello.o add.o sub.o div1.o
    	gcc hello.o add.o sub.o div1.o -o hello
    hello.o:hello.c
    	gcc -c hello.c -o hello.o
    add.o:add.c
    	gcc -c add.c -o add.o
    sub.o:sub.c
    	gcc -c sub.c -o sub.o
    div1.o:div1.c
    	gcc -c div1.c -o div1.o
    
    

  * 修改后，我们把`hello`当作目标，`hello.o add.o sub.o div1.o`作为依赖。如果文件第一次编译，这些.o文件不存在，那就去下面的文件需要生产规则。当我们单独修改`add.c`文件，由于依赖时间不能先于目标`add.o` 那么需要重新单独执行`gcc -c add.c -o add.o`。这样就省去了许多功夫。

## 5.2 补充-ALL指定输出目标

> makefile 默认第一个目标文件为终极目标，生成就跑路，这时候可以用 ALL 来指定终极  
>  目标指定目标的 makefile
    
    
    ALL:hello
    hello.o:hello.c
    	gcc -c hello.c -o hello.o
    add.o:add.c
    	gcc -c add.c -o add.o
    sub.o:sub.c
    	gcc -c sub.c -o sub.o
    div1.o:div1.c
    	gcc -c div1.c -o div1.o
    hello:hello.o add.o sub.o div1.o
    	gcc hello.o add.o sub.o div1.o -o hello
    
    

## 5.3 两个函数演示

> 利用函数生成的`src，obj`可以代替原本一长串的`.c or .o` ,如下展示。  
>  `*.c` 表示的是当前目录下所有.c文件 `%.c`表示当前目录下第一个。这边用`%`做到依次取出.c修改成`.o`
    
    
    src=$(wildcard *.c) # add.c sub.c div1.c hello.c
    obj=$(patsubst %.c,%.o,$(src)) # add.o sub.o div1.o hello.o
    
    ALL:hello
    hello:$(obj)
    	gcc $(obj) -o hello
    hello.o:hello.c
    	gcc -c hello.c -o hello.o
    add.o:add.c
    	gcc -c add.c -o add.o
    sub.o:sub.c
    	gcc -c sub.c -o sub.o
    div1.o:div1.c
    	gcc -c div1.c -o div1.o
    
    

## 5.4 补充-clean用法

> 为了方便使用，能够一次性清楚`.o`文件和输出文件。使用`clean`做出如下修改
    
    
    src=$(wildcard *.c) # add.c sub.c div1.c hello.c
    obj=$(patsubst %.c,%.o,$(src)) # add.o sub.o div1.o hello.o
    
    ALL:main
    main:$(obj)
    	gcc $(obj) -o main
    hello.o:hello.c
    	gcc -c hello.c -o hello.o
    add.o:add.c
    	gcc -c add.c -o add.o
    sub.o:sub.c
    	gcc -c sub.c -o sub.o
    div1.o:div1.c
    	gcc -c div1.c -o div1.o
    clean:
    	-rm -rf $(obj) main
    
    

  * `rm` 前面的`-`，代表出错依然执行。比如，待删除文件集合是 5 个，已经手动删除了 1 个，就只剩下 4个，然而删除命令里面还是 5 个的集合，就会有删除不存在文件的问题，不加这-，就会报错，告诉你有一个文件找不到。加了-就不会因为这个报错。

## 5.5 三个自动变量用法

  * `$@` ：在规则命令中，表示规则中的目标
  * `$<` ：在规则命令中，表示规则中的第一个条件，如果将该变量用在模式规则中，它可以将依赖条件  
列表中的依赖依次取出，套用模式规则

  * `$^` ：在规则命令中，表示规则中的所有条件，组成一个列表，以空格隔开，如果这个列表中有重复项，则去重

**做出如下修改**

    
    
    src=$(wildcard *.c) # add.c sub.c div1.c hello.c
    obj=$(patsubst %.c,%.o,$(src)) # add.o sub.o div1.o hello.o
    
    ALL:main
    main:$(obj)
    	gcc $(obj) -o $@
    hello.o:hello.c
    	gcc -c $< -o $@
    add.o:add.c
    	gcc -c $< -o $@
    sub.o:sub.c
    	gcc -c $< -o $@
    div1.o:div1.c
    	gcc -c $< -o $@
    clean:
    	-rm -rf $(obj) main
    
    

补充说明

  * sub，add 这些指令中使用 < 和 <和 <和^都能达到效果，但是为了模式规则，所以使用的$<

## 5.6 补充-模板规则

> 此时要添加一个乘法函数，就需要在 makefile 里面增加乘法函数的部分。扩展性就不行，所有就有了模板规则了
    
    
    src=$(wildcard *.c) # add.c sub.c div1.c hello.c
    obj=$(patsubst %.c,%.o,$(src)) # add.o sub.o div1.o hello.o
    
    ALL:main
    main:$(obj)
    	gcc $(obj) -o $@
    %.o:%.c
    	gcc -c $< -o $@
    clean:
    	-rm -rf $(obj) main
    
    

此时我们增加一个乘法的模板重新make ./![在这里插入图片描述](https://img-
blog.csdnimg.cn/20210426150208413.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210426150224580.png)

增加函数的时候，不用改 makefile，只需要增加.c 文件，改一下源码就行。

## 5.7 补充-静态模板规则

> 使用静态模式规则，就是指定模式规则给谁用，这里指定模式规则给 obj 用，以后文件多了，文件集合会有很多个，就需要指定哪个文件集合用什么规则。  
>  也可以理解成目标依赖从那个模板规则来.

  * `$(obj):%.o:%.c`

修改如下

    
    
    src=$(wildcard *.c) # add.c sub.c div1.c hello.c
    obj=$(patsubst %.c,%.o,$(src)) # add.o sub.o div1.o hello.o
    
    ALL:main
    main:$(obj)
    	gcc $(obj) -o $@
    $(obj):%.o:%.c
    	gcc -c $< -o $@
    
    clean:
    	-rm -rf $(obj) main
    
    

## 5.8 补充-伪目标

> 当前文件夹下有 ALL 文件或者 clean 文件时，会导致 makefile 瘫痪，所以添加一行
    
    
    src=$(wildcard *.c) # add.c sub.c div1.c hello.c
    obj=$(patsubst %.c,%.o,$(src)) # add.o sub.o div1.o hello.o
    
    ALL:main
    main:$(obj)
    	gcc $(obj) -o $@ 
    $(obj):%.o:%.c
    	gcc -c $< -o $@  
    
    clean:
    	-rm -rf $(obj) main
    
    .PHONY: clean All
    

## 5.9补充-第三方函数的添加

    
    
    src=$(wildcard *.c) # add.c sub.c div1.c hello.c
    obj=$(patsubst %.c,%.o,$(src)) # add.o sub.o div1.o hello.o
    myArgs=-Wall -g -pthread
    
    ALL:main
    main:$(obj)
    	gcc $(obj) -o $@ $(myArgs)
    $(obj):%.o:%.c
    	gcc -c $< -o $@ $(myArgs)
    
    clean:
    	-rm -rf $(obj) main
    
    .PHONY: clean All
    

# 小点补充

  * makefile 也可以写成Makefile
  * `src=$(wildcard *.c)` 表示的当前文件夹下的所有.c等价与`src=$(wildcard ./*.c)`

# 总结

> 重点关注实例部分是怎么层层递进修改的，如有错误，欢迎指出。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210426192127604.jpg?x-oss-
process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3Mjk5Nw==,size_16,color_FFFFFF,t_70)

