

![力扣 C_+ 工程师学习路线.png](E:\MarkDown\picture\1658328824-UjKjaL-C__ 工程师学习路线.png)











# 语言

## 解释型语言
>·由解释器根据输入的数据当场执行而不生成任何的目标程序，
>·如在终端上打一条命令或语句，解释程序就立即将此语句解释成一条或几条指令并提交硬件立即执行且将执行结果反映到终端，从终端把命令打入后，就能立即得到计算结果。这的确是很方便的，很适合于一些小型机的计算问题。但解释程序执行速度很慢，
>·如果源程序中出现循环，则解释程序也重复地解释并提交执行这一组语句，这就造成很大浪费。 

## 编译型语言
>·先将源代码编译成目标语言之后通过连接程序连接到生成的目标程序进行执行
>·编译程序工作时，先分析，后综合，从而得到目标程序。所谓分析，是指词法分析和语法分析；所谓综合是指代码优化，存储分配和代码生成。为了完成这些分析综合任务，编译程序采用对源程序进行多次扫描的办法，每次扫描集中完成一项或几项任务，也有一项任务分散到几次扫描去完成的。

1.通过上面的分类，我们可以将**C++和python都归类为强类型语言。**python变量无需声明并不意味着就是弱类型，弱类型是指能够进行隐式转换，python是不能这么转换的，每个实例类型是固定的，转换实例类实际上是重新创建一个内存空间。

2.C++为编译型语言；python为解释型的脚本语言。

3.C++效率高，编程难；python效率低，编程简单。同样的功能，或许python可以很快的写出代码，但运行所需的时间需要成倍于C++。

## 静态数据类型语言/静态类型语言

如c++，类型检查发生在编译时，因此编译器必须知道程序中每一个变量对应的数据类型。





## 基础

1.输出变量类型

```c++
int a = 4;
cout << typeid(a).name() << endl;
```



vs：F5和绿色三角功能一样，运行且调试；Ctrl+F5，只运行
vs code：除了F5，还多了个小三角，可以点小三角或者右键run code，停止程序是在输出那里右键stop

2.

一个字节为8位，一个字由32或64比特构成，也就是4或8字节

float 一个字，即四个字节；double 两个字；long long 三到四个字
int 一般为四个字节，即一个字；

对于浮点数运算，一般选用double，因为计算代价和float差不多，且float通常精度不够

3.

return;   // 只能用在返回类型是void的函数中，且对于这种函数，不要求非要有return，这类函数在最后一句会隐式的执行return

4.

1G(吉字节)=1024MB(兆字节)=1024 * 1024KB(千字节) = 1024 * 1024 * 1024B(字节)，这里1KB也可以等于1000B











# 黑马

## C++基础入门



### 2.c++中 # 的作用(仅在宏定义中有效)

1、单个井号（#）：在宏展开的时候会将#后面的参数替换成相应的字符串

2、两个井号（##）：起连接符的作用，将前后两个参数连接为一个子串，但连接后的子串不能当成字符串形式

```cpp
#include <stdio.h>
#define STR(s) #s
#define CONS(a,b) (int)(a##e##b)
#define PASTER(n) printf("token" #n " = %d\n", token##n)
int main(void)
{
#ifdef STR
    printf(STR(XJ1));  // 输出XJ1
#endif
    
#ifdef CONS
    printf("\n%d\n",CONS(2,3));  // 输出2000
#endif
    
	int token1 = 10;
	PASTER(1);  // 输出token1 = 10
    return 0;
}

/* 第一个宏，用#把参数转化为一个字符串
 * 第二个宏，用##把2个宏参数粘合在一起，及aeb,2e3也就是2000
 * 第三个宏，将后面的参数连接在一起，变为token1，即刚赋值过的变量
*/
```



### 3.sizeof

利用sizeof关键字可以**统计数据类型所占内存大小**，单位是字节

```c++
cout << sizeof(vector);
```

vs：

64位：
debug x64   -> 32 == 4 * 8，这里一个Int指针为8字节
release x64  -> 24 == 3 * 8

32位：
debug x86    -> 16 == 4 * 4，这里Int指针为4字节
release x86   -> 12 == 3 * 4

可以发现，release都会比debug小。这里vector，debug是四个指针，release优化后就为3个指针了。
通过查看STL源码可以看到vector有四个成员变量 
 _A  allocator; 
 iterator  _First,  _Last,  _End; 

Debug：调试版本，包含调试信息，所以容量比Release大很多，并且不进行任何优化（优化会使调试复杂化，因为源代码和生成的指令间关系会更复杂），便于程序员调试。**Debug模式下生成两个文件，除了.exe或.dll文件外，还有一个.pdb文件，该文件记录了代码中断点等调试信息** 

Release：发布版本，不对源代码进行调试，编译时对应用程序的速度进行优化，使得程序在代码大小和运行速度上都是最优的。（调试信息可在单独的PDB文件中生成）。**Release模式下生成一个文件.exe或.dll文件**

vs code没有release、debug、x86、x64这些选项，是因为在.json配置文件里设置过了



### 4.优先级

数字越大，优先级越高

![数字越大，优先级越高](E:\MarkDown\picture\image-20220415234140006.png)



### 5.随机数生成

```cpp
void srand(unsigned int seed)
// seed  -- 这是一个整型值，用于伪随机数生成算法播种。
```

**用法:** 

​	它初始化随机种子，会提供一个种子，这个种子会对应一个随机数，如果使用相同的种子后面的 rand() 函数会出现一样的随机数。
​	为了防止随机数每次重复，常常使用系统时间来初始化，即使用 time函数来获得系统时间，将time_t型数据转化为(unsigned)型再传给srand函数，即: srand((unsigned) time(&t)); 还有一个经常用法，不需要定义time_t型t变量,即: srand((unsigned) time(NULL)); 直接传入一个空指针，因为你的程序中往往并不需要经过参数获得的数据。
​	只需在主程序开始处调用 srand((unsigned)time(NULL)); 即可，后面直接用rand就可以了。

```cpp
srand((unsigned int)time(NULL));
int num = 0;
//系统生成随机数,rand()%区间，这里rand() % 100表示生成0-99的随机数
	/*
	要取得 [a,b] 的随机整数，使用 (rand() % (b-a+1))+ a;
	要取得 [a,b) 的随机整数，使用 (rand() % (b-a))+ a;
	要取得 (a,b] 的随机整数，使用 (rand() % (b-a))+ a + 1;
	通用公式: a + rand() % n；其中的 a 是起始值，n 是整数的范围。
	*/
int sj = rand() % 100 + 1;  // [1,100]
rand() % 100 + 2
```



### 6.do while

与while循环区别在于，do...while先执行一次循环语句，再判断循环条件；且在末尾有分号

```cpp
do
{
	cout << num << endl;
	num++;

} while (num < 10);
```



### 7.goto跳转语句

 `goto 标记;`

```cpp
 int main() {
	cout << "1" << endl;

	goto FLAG;

	cout << "2" << endl;
	cout << "3" << endl;
	cout << "4" << endl;

	FLAG:

	cout << "5" << endl;
	system("pause");
	return 0;
}
```





### 9.指针

**指针的作用：** 可以通过指针间接访问内存
指针变量可以通过" * "操作符，操作指针变量指向的内存空间，这个过程称为解引用

```cpp
int a = 10; 
int *p;//指针

//指针变量赋值
p = &a; //通过 & 符号 获取变量的地址,指针指向变量a的地址
cout << &a << endl; //打印数据a的地址
cout << p << endl;  //打印指针变量p

cout << "*p = " << *p << endl;  // 10
// 所有指针类型在32位操作系统下是4个字节，在64位系统下是8字节
cout << sizeof(p) << endl;
```



**空指针**：指针变量指向内存中编号为0的空间
**用途：**初始化指针变量
**注意：**空指针指向的内存是不可以访问的

**野指针**：指针变量指向非法的内存空间

```cpp
// 空指针
int *p = NULL;
// 野指针
// eg:指针变量p指向内存地址编号为0x1100的空间
int * p = (int *)0x1100;
```

空指针和野指针都不是我们申请的空间，因此不要访问。



指针和引用作为形参：

```cpp
#include<iostream>
 
using namespace std;

// 当把指针作为参数进行传递时，也是将实参的一个拷贝传递给形参，两者指向的地址相同，但不是同一个变量，在函数中改变这个变量的指向不影响实参
void testPTR(int* p) {
	cout << p << endl;  // 0x61ff08
	int a = 12;
	p = &a;
	cout << p << endl;  // 0x61fedc

}
// 对于引用，等价于int* const p = &a;，指向不可变，但指向的值可以变
void testREFF(int& p) {
	int a = 12;
	p = a;

}
int main()
{
	int a = 10;
	int* b = &a;
	cout << "b:" << b << endl;  // 0x61ff08
	testPTR(b);
	cout << "new b:" << b << endl;  // 0x61ff08
	cout << a << endl;// 10
	cout << *b << endl;// 10

	a = 10;
	testREFF(a);
	cout << a << endl;//12
}
```





### 11.值传递和地址传递

>1. **对于内置类型(如int)而言，pass-by-value通常比pass-by-reference高效。STL的迭代器和函数对象也同样如此，因为习惯上它们都被设计为passed by value**
>       传值虽然会增加**栈**的一个副本的开销，但是如果传引用（相当于传指针，引用在编译器底层的其实也是指针来实现的），则获取入参的value 则需要寻址访问获取参数值，多了得到地址值的一个过程。对于比较小的对象，内存占用也是差不多的
>       对于STL的迭代器，可以看作是指针，因此没有必要使用 pass by reference
>       对于函数对象，函数对象是行为类似函数的一个对象，一般是重载了operator() 的 struct 或者 class。这种函数对象，一般没有构造和析构函数，体积很小。在我们调用函数对象的时候，实际上不是真的传递了这个对象，而是**传递了指向该对象的一个指针**。事实上，把函数对象传递给算法时，比传递一个函数的效率高得多，《effcient c＋＋》中的实验结论，使用函数对象一般是裸函数的1.5倍，最多能快2倍多
>
>   ```cpp
>   // 如果一个类将()运算符重载为成员函数，这个类就称为函数对象类，这个类的对象就是函数对象。
>   // 函数对象是一个对象，但是使用的形式看起来像函数调用，实际上也执行了函数调用
>   // 函数对象测试
>   #include <algorithm>
>   #include <vector>
>   #include <ctime>
>   #include <iostream>
>    
>   struct neg {
>   	int operator()(int x) { return -x; }
>   };
>    
>   void foo1(int i, neg& n) {
>   	int val = n(i);
>   }
>   
>   void foo2(int i, neg n) {
>   	int val = n(i);
>   }
>   
>   void test1() {
>   	clock_t start, end;
>   	start = clock();
>   	for (size_t i = 0; i < 10000000; ++i)
>   		foo1(rand(), neg());
>   	end = clock();
>   	std::cout << " pass by reference cost: " << end - start << "ms" << std::endl;
>   }
>   
>   void test2() {
>       clock_t start, end;
>   	start = clock();
>   	for (size_t i = 0; i < 10000000; ++i)
>   		foo2(rand(), neg());
>   	end = clock();
>   	std::cout << " pass by reference cost: " << end - start << "ms" << std::endl;
>   }
>   int main()
>   {
>   	test1();  // 地址传递 593
>       test2();  // 值传递 461
>   }
>   ```
>
>   
>
>2. **对于用户自定义的类，pass-by-reference-to-const更好**
>       pass-by-value创建的副本系由对象的拷贝构造函数产出，可能让pass-by-value比较费时，会额外多出一次拷贝构造和析构的调用，若对象是类的话，由于类成员的多样化，可能会额外多出若干次构造和析构
>       且即使小型对象拥有并不昂贵的拷贝构造函数，还是可能有效率上的争议。某些编译器对待 内置类型 和 用户自定义类型 有着不同的态度，纵然两者拥有相同的底层表述。举个例子，某些编译器拒绝把只由一个double组成的对象放进缓存器内，却很乐意在一个正规基础上对光秃秃的doubles 那么做。当这种事发生，你更应该以byreference方式传递此等对象，因为编译器当然会将指针吧(references的实现体)放进缓存器内，绝无问题。另一个理由就是，作为一个用户自定义类型，其大小容易有所变化。一个type目前虽然小，但将来可能会变大，因为其内部实现可能改变。甚至当改用另一个cpp编译器都可能改变type的大小。
>
>3. **以by reference方式传递参数也可以避免slicing（对象切割）问题**
>       当一个派生类对象以by value方式传递并被视为一个base class对象，调用的将是base class的拷贝构造函数，因此derived class对象的那些特化性质全被切割掉了，仅留下一个base class对象
>
>   ```cpp
>   class Window {
>   public:
>       ...
>       std::string name() const;  // 返回窗口名称
>       virtual void display() const;  // 显式窗口和名称。虚函数，两类有不同显式方式
>   };
>   class WindowGood:public Window {
>   public:
>       ...
>       virtual void display() const;  // 显式窗口和其名称
>   };
>   
>   // 错误示例
>   void print(Window w) {
>       cout << w.name();
>       w.display();
>   }
>   WindowGood w;
>   // 虽然传入的是派生类，但由于这里是值传递，WindowGood会被构造成为Window对象，因此WindowGood的所有特化信息都会被切除。函数内调用的都是window::xx
>   print(w);
>   
>   // 正确示例
>   // 传入的什么类型，w就表现出那种类型。传递的是指针
>   void print(const Window& w) {
>       cout << w.name();
>       w.display();
>   }
>   WindowGood w;
>   print(w);
>   ```
>
>   



```cpp
//地址传递
void swap1(int &p1, int &p2)
{
    int temp = p1;
	p1 = p2;
    p2 = temp;
}
void swap2(int * p1, int *p2)  // 结构体做函数参数的地址传递也是这种形式
{
	int temp = *p1;
	*p1 = *p2;
	*p2 = temp;
}

int main() {

	int a = 10;
	int b = 20;
    swap1(a,b);  //地址传递会改变实参

	swap2(&a, &b);  //地址传递会改变实参

	cout << "a = " << a << endl;

	cout << "b = " << b << endl;

	system("pause");

	return 0;
}
```

对于数组

```cpp
void bubbleSort(int * arr, int len){}//int * arr 也可以写为int arr[]，这里也可以是int arr[10]，因为数组名可以当作指向该数组第一个元素的指针，且在形参中不需要指定数组的大小，传入一个指针就行，想要确定几行几列的话还需要另外定义参数进行传入
int main()
{
	int arr[10] = { 4,3,6,9,1,2,10,8,7,5 };
	int len = sizeof(arr) / sizeof(int);
	bubbleSort(arr, len);  // 调用
}
```

对于结构体

> 形参改为指针，可以减少内存空间，而且不会复制新的副本出来，因此使用较多
>
> 若要避免对数据的修改，在形参前加const

```cpp
struct student
{
	//成员列表
	string name;  //姓名
	int age;      //年龄
	int score;    //分数
};
//地址传递
void printStudent2(const student *stu)  //加const防止函数体中的误操作，一般const student& stu用的更多
{
	//stu->age = 28;
	cout << "子函数中 姓名：" << stu->name << " 年龄： " << stu->age  << " 分数：" << stu->score << endl;
}
int main() {
	student stu = { "张三",18,100};
	//地址传递
	printStudent2(&stu);

	system("pause");
	return 0;
}
```





### 12.预处理器、预编译/条件编译指令 

预处理器：确保头文件多次包含仍能安全工作，在编译之前执行；预处理器负责将程序中的预处理变量替换成它的真实值

预处理功能：

* #include
* 头文件保护符
  * 依赖于**预处理变量**（由预处理器管理的变量），#define指令将一个名字设定为预处理变量（预处理变量通常全部大写），#ifdef  #ifndef 检查某个指定的预处理变量是否已经定义



> 预处理指令：#include  #define  #undef  #program once
>
> #undef：取消已定义的宏
>
> 条件编译指令：#ifdef  #ifndef  #endif  ；#if  #elif  #else  #endif
>
> \#ifdef如果宏已经定义，则编译下面代码
> #ifndef如果宏没有定义，则编译下面代码
>
> #if如果给定条件为真，则编译下面代码
> #elif如果前面的#if给定条件不为真，当前条件为真，则编译下面代码
>
> #endif结束一个#if……#else条件编译块    

#### #define

```cpp
#include <stdio.h>
#define MAX(x,y) (((x)>(y))?(x):(y))
#define MIN(x,y) (((x)<(y))?(x):(y))
int main(void)
{
#ifdef MAX    //判断这个宏是否被定义
    printf("3 and 5 the max is:%d\n",MAX(3,5));
#endif
#ifdef MIN
    printf("3 and 5 the min is:%d\n",MIN(3,5));
#endif
    return 0;
}

/*
 * (1)三元运算符要比if,else效率高
 * （2）宏的使用一定要细心，需要把参数小心的用括号括起来。比如不加x、y的小括号，(x*x)，带入b+2，则会出现b+2*b+2
 * 因为宏只是简单的文本替换，不注意，容易引起歧义错误。
*/
```

#### 2.尽量以constexpr、scoped enum、inline替换#define

> 宁以编译器替换预处理器

```cpp
#define ASPECT_RATIO 1.653
```

记号名称ASPECT_RATIO也许从未被编译器看见；也许在编译器开始处理源码之前它就被预处理器移走了。于是记号名称ASPECT_RATIO 有可能没进入记号表(symbol table)内。于是当你运用此常量但获得一个编译错误信息时，可能会带来困惑，因为这个错误信息也许会提到1.653 而不是ASPECT_ RATIO。 如果ASPECT_RATIO 被定义在一个非你所写的头文件内，你肯定对1.653以及它来自何处毫无概念，于是你将因为追踪它而浪费时间。这个问题也可能出现在记号式调试器(symbolic debugger)中，原因相同：你所使用的名称可能并未进入记号表(symbol table)

**constexpr**

```cpp
constexpr double ASPECT_RATIO = 1.653;
```

























#### #program once和\#ifndef

为了避免同一个文件被include多次，C/C++中有两种方式，一种是#ifndef方式，一种是#pragma once方式。在能够支持这两种方式的编译器上，二者并没有太大的区别，但是两者仍然还是有一些细微的区别。 

\#ifndef的方式受C/C++语言标准支持。它不光可以保证同一个文件不会被包含多次，也能保证内容完全相同的两个文件（或者代码片段）不会被不小心同时包含。 当然，缺点就是如果不同头文件中的宏名不小心“撞车”，可能就会导致你看到头文件明明存在，编译器却硬说找不到声明的状况——这种情况有时非常让人抓狂。 由于编译器每次都需要打开头文件才能判定是否有重复定义，因此在编译大型项目时，ifndef会使得编译时间相对较长，因此一些编译器逐渐开始支持#pragma once的方式。

\#pragma once一般由编译器提供保证：同一个文件不会被包含多次，需要编译器支持才行，因此没有#ifndef通用。注意这里所说的“同一个文件”是指物理上的一个文件，而不是指内容相同的两个文件。你无法对一个头文件中的一段代码作pragma once声明，而只能针对文件。 其好处是，你不必再费劲想个宏名了，当然也就不会出现宏名碰撞引发的奇怪问题。大型项目的编译速度也因此提高了一些。 对应的缺点就是如果某个头文件有多份拷贝，本方法不能保证他们不被重复包含。
但#program once是编译器相关的，就是说在这个编译系统上能用，但在其他的编译系统上就不一定能用，所以其可移植性较差。一般如果强调程序跨平台，还是选择使用“**#ifndef,  #define,  #endif**”比较好，如13介绍的标准写法。

### 13.前置声明和头文件标准写法(防卫式声明)

防止重复的include

但里面这三块内容有点问题，只需要包含第一块。在[Google C++ 风格指南](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/headers/#forward-declarations)中有说明，要尽量避免使用前置声明，且类定义写在 .cpp 中。

> 尽可能地避免使用前置声明。使用 `#include` 包含需要的头文件即可。

![image-20220429180828323](E:\MarkDown\picture\image-20220429180828323.png)

```cpp
// eg.complex.h
#ifndef __COMPLEX__  // 两个_
#define __COMPLEX__

// 中间写声明的东西

#endif
```

**前置声明**（forward declaration）：是类、函数和模板的纯粹声明，没伴随着其定义，用来代替一些直接包含头文件的情况

**要求**：**其声明的类是文件所声明的类的数据成员时，是指针成员或引用成员**（而不是对象成员）；其声明的类是文件所声明的类的成员函数的参数或返回值时，该函数在文件中不存在定义。

eg:

```CPP
class Bar;

class foo
{
public:
	Bar getBar();  // 成员函数的返回值情况
private:
	Bar* _bar;  // 指针成员情况
};
```

优点：

* 前置声明能够节省编译时间，多余的 `#include` 会迫使编译器展开更多的文件，处理更多的输入。

* 前置声明能够节省不必要的重新编译的时间。 `#include` 使代码因为修改某个头文件后需要编译多个无关的依赖文件

缺点：

* 前置声明隐藏了依赖关系，头文件改动时，用户的代码会跳过必要的重新编译过程。

* 前置声明可能会被库的后续更改所破坏。前置声明函数或模板有时会妨碍头文件开发者变动其 API. 例如扩大形参类型，加个自带默认参数的模板形参等等。

* 前置声明来自命名空间 `std::` 的 symbol 时，其行为未定义。

* 很难判断什么时候该用前置声明，什么时候该用 `#include` 。极端情况下，用前置声明代替 `#include` 甚至都会暗暗地改变代码的含义（这一块不是很理解
* 前置声明了不少来自头文件的 symbol 时，就会比单单一行的 `include` 冗长。
* 仅仅为了能前置声明而重构代码（比如用指针成员代替对象成员）会使代码变得更慢更复杂.



### 14.extern

* **引用同一个文件中的变量**
  利用extern关键字，使用在后边定义的变量。

```cpp
#include<stdio.h>
int func();
int main()
{
	func();
	extern int num;
	printf("%d",num);
	return 0;
}
int num = 3;
int func()
{
	printf("%d\n",num);
}
```

*  **引用另一个文件中的全局变量或函数**

如果全局变量定义在了.h文件中，若这个头文件被多次引用，则会被重复定义

因此我们可以在.cpp文件中定义全局变量，在需要用到的.h文件中extern（头文件中放的是声明，定义任何变量都是一种很业余的行为）；另外一种方法是可以是在a.cpp文件中定义了全局变量int global_num ，可以在对应的a.h头文件中写extern int global_num ，这样其他源文件可以通过include a.h来声明她是外部变量就可以了。

* **可用于C＋＋程序中调用c函数的规范问题**

这是给链接器用的，告诉链接器在链接的时候用C语言规范来链接。主要原因是C＋＋和C程序编译完成后在目标代码中命名规则不同。C＋＋语言在编译的时候为了解决的多态问题，会将函数名和参数联合起来生成一个中间的名称，而c语言则不会，因此会造成链接时找不到对应的情况，此时C就需要用extern "C"进行链接指定，这告诉编译器，请保持我的名称，不要给我生成用于链接的中间名。

常用形式：

```cpp
//xx.h
extern int add(...)

//xx.c
int add(){

}

//xx.cpp
extern "C" {
    #include "xx.h"
}
```



## C++核心编程

### 2.构造函数的初始化列表/初始值列表

**语法：**`构造函数(....)：属性1(值1),属性2（值2）... {}`

> 注意，成员的初始化顺序与它们在类定义中的出现顺序一致，与构造函数初始值列表中初始值的前后位置关系无关
>
> 所以如果一个成员是用另一个成员来初始化的，我们就需要注意一下这两个成员在类定义中出现的顺序

**与赋值初始化的主要区别：**

​	对于在函数体中初始化，是在所有的数据成员被分配内存空间后才进行的。
​	初始化列表是给数据成员分配内存空间时就进行初始，就是说分配一个数据成员只要冒号后有此数据成员的赋值表达式(此表达式必须是括号赋值表达式)，那么分配了内存空间后在进入函数体之前给数据成员赋值，就是说初始化这个数据成员此时函数体还未执行。 
​	==因此如果成员是const、引用，或者属于某种未提供默认构造函数的类类型，它们就一定需要初值，不能被赋值，我们必须通过构造函数初始值列表为这些成员提供初值==

> 条款四
> 对于在函数体中初始化，即基于赋值的版本，首先调用**default构造函数**为那些成员设初值，然后在使用**拷贝赋值运算符**立即再对他们赋予新值；构造函数的初始化列表的方法避免了这一问题，初值列中针对各个成员变量设置的实参，被拿去作为各成员变量的构造函数的实参，依次进行**拷贝构造**。**比起先调用默认构造函数再调用拷贝赋值操作符，单只用一次拷贝构造函数是更为高效的**

**使用初始化列表的原因：**

​	赋值初始化是通过在函数体内进行赋值初始化，是在构造函数当中做赋值的操作，而初始化列表是做纯粹的初始化操作。我们都知道，C++的赋值操作是会产生临时对象的(即会先进行一次默认构造然后再赋值)。临时对象的出现会降低程序的效率。因此用成员初始化列表会更快一点。详细解释：
​	对于内置类型，如int, float等，使用初始化列表和在构造函数体内初始化差别不是很大，为了一致性最好也通过成员初值列来初始化；但是对于**类**类型来说，最好使用初始化列表，因为使用**初始化列表少了一次调用默认构造函数的过程**，这对于数据密集型的类来说，是非常高效的。

```cpp
class Person {
public:
	////传统方式初始化
	//Person(int a, int b, int c) {
	//	m_A = a;
	//	m_B = b; 
	//	m_C = c;
	//}

	//初始化列表方式初始化，有参构造
	Person(int a, int b, int c) : m_A(a), m_B(b), m_C(c) {}
    // 这里也可以给赋个初值
    Person(int a = 0, int b = 0, int c = 0) : m_A(a), m_B(b), m_C(c) {}
    // 如果参照下面那个写成无参构造的形式
    // 但好像没啥意义。其中int()即类型名加()表示调用int的默认构造函数
	Person() :m_A(int()), m_B(int()), m_C(int()) {}
    
	void PrintPerson() {
		cout << "mA:" << m_A << endl;
		cout << "mB:" << m_B << endl;
		cout << "mC:" << m_C << endl;
	}
    
private:
	int m_A;
	int m_B;
	int m_C;
};

int main() {
	Person p(1, 2, 3);
	p.PrintPerson();
	Person p1{p};  // 这样也可以初始化
	p1.PrintPerson();

	system("pause");
	return 0;
}
```

对应的无参构造写法可以参考

![image-20220506223928543](E:\MarkDown\picture\image-20220506223928543.png)



例子：

```cpp
#include<iostream>
#include<fstream>
using namespace std;

class Test1
{
public:
Test1() // 无参构造函数
{cout << "Construct Test1" << endl ;}
Test1(const Test1& t1) // 拷贝构造函数
{cout << "Copy constructor for Test1" << endl ;this->a = t1.a ;}
Test1& operator = (const Test1& t1) // 赋值运算符
{cout << "assignment for Test1" << endl ;this->a = t1.a ;return *this;}
int a ;
};
class Test2
{
public:
Test1 test1 ;
Test2(Test1 &t1)
{test1 = t1 ;}
};

int main(){
	Test1 t1;
	Test2 t2(t1);
	system("pause");
	return 0;	
}
```

输出： 
 Construct Test1 
 Construct Test1 
 assignment for Test1

解释一下： 
 第一行输出对应调用代码中第一行，构造一个Test1对象 
 第二行输出对应Test2构造函数中的代码，用默认的构造函数初始化对象test1 // 这就是所谓的初始化阶段 
 第三行输出对应Test2的赋值运算符，对test1执行赋值操作 // 这就是所谓的计算阶段



同样看上面的例子，我们使用初始化列表来实现Test2的构造函数。

```cpp
class Test2
{
public:
  Test1 test1 ;// Test1是一个类
  Test2(Test1 &t1):test1(t1){}  // 构造函数
}
```

使用同样的调用代码，输出结果如下： 
 Construct Test1 
 Copy constructor for Test1

当调用Test2的构造函数时，第一行调用Test1的构造函数，第二行直接调用拷贝构造函数初始化Test1，省去了调用默认构造函数的过程。 



```cpp
#include<iostream>
#include<fstream>
using namespace std;
class Test1{
public:
  Test1(){a=0;cout << "默认构造函数" << endl;}
  Test1(const Test1& t){a=t.a;cout << "拷贝构造函数" << endl;}
  Test1& operator = (const Test1& t1) // 赋值运算符
{cout << "为Test1赋值" << endl ;this->a = t1.a ;return *this;}
  int a;
};
class Test2
{
public:
  Test1 test1 ;// Test1是一个类
  Test2(Test1 &t1):test1(t1){cout << "T2拷贝构造函数" << endl;}  // 构造函数
  //Test2(Test1 &t1){
  //    test1 = t1;
  //    cout << "T2拷贝构造函数" << endl;
  //} 
};

int main(){
	Test1 t1;
	Test2 t2(t1);
	system("pause");
	return 0;	
}
```

![image-20220508221648649](E:\MarkDown\picture\image-20220508221648649.png)



### 3.构造函数的调用规则

* 如果用户定义有参构造函数，c++不在提供默认无参构造，但是会提供默认拷贝构造，因此定义有参构造后需要用户再定义无参构造，否则调用无参构造时会出错


* 如果用户定义拷贝构造函数，c++不会再提供其他构造函数

特例：

只定义有参构造时，需要用户再定义无参构造。但也可以**在定义有参构造时加上默认参数**

```cpp
#include<iostream>
#include<fstream>
using namespace std;

class Test2
{
public:
  Test2(int a1 = 0, int b1 = 0){
  a=a1;
  b=b1;
  }
  // Test2(int a1 = 0, int b1 = 0):a(a1),b(b1){}
  int a,b;
};

int main(){
	Test2 t2;
	system("pause");
	return 0;	
}
```





特化和泛化

```cpp
#include<iostream>
#include<fstream>
using namespace std;

class a{
	int x;
	int y;
	int z;
};

template<typename type>
struct b{
	typedef a a1;
};
int main(){
	b<int>::a1 p;
	system("pause");
	return 0;	
}
```





## C++提高编程



### 成员模板

一个模板可以在一个类或类模板中声明，这样的类模板称为成员模板;成员模板的定义既可以在类(或类模板)定义内部，也可以在类(或类模板)定义的外部:当类模板的成员模板在类模板定义的外部定义时，应该完整地指定类模板的参数和成员模板的参数。例如:

```cpp
template <class T>template <class I> void queue::assign(I beg, I end);
```

上述代码中template < class T>是类模板，作为第一个模板形参表; template< class T>是成员模板，作为第二个模板形参表。

```cpp
template <class T>class string   //类模板string
{
public:
	template<class T2>int compare (const. T2&);
}

template <class T>template<class T2>int string<T>: :compare (const T2& s)         //在类外部定义成员函数
{
	......
}
```

上述代码在类模板string中，使用了另一个类模板类型的成员函数

```cpp
template <typename T> class bC
{
public;
	template <typename V> class HC   //类模板HC作为模板bC的成员
	{
	public:
		V m;
		V n;
	public:
		HC(){} ;
		void show() {cout<<m<<endl;};
	}
	HC <T> A;
	HC<int>B;
public:
	bC(){};
	template <typename U>U show m(){ };   //类模板定义成 员函数
}
```

上述代码举例说明使用类模板作为另一类模板的成员。另外，成员模板具有一些使用规则。

> - 成员模板遵循常规访问控制；
> - 成员模板不能为虚(virtue)；
> - 局部类不能拥有成员模板；
> - 析构器不能是模板类型；
> - 成员函数模板不能重载基类的虚函数；
> - 成员模板具有复杂的转换功能。



# c++ primer



## 2.main函数的return

return 0：一般用在主函数结束时，按照程序开发的一般惯例，表示成功完成本函数。
return -1：:表示返回一个代数值，一般用在子函数结尾。按照程序开发的一般惯例，表示该函数失败；

以上两个是约定俗成，系统提供的函数绝大部分定义为int类型返回值的都是这样的。返回值是返回给系统用的，给系统看得。一般做调试的时候也会用的，当出现错误的时候可以根据返回值来确定问题出在哪一个函数上的。

自己输出的话看不出差别



## 5.string、char*、string_view(待补充)

> 注意，字符串字面值（如“aaaa”）不是标准库类型string的对象，其类型为const char [x+1]，如这个例子是const char [5]。但他们可以相加，不过要确保+的两端至少有一个string
>
> void getNext(int* next, const string& s){}  // 这里string&将string声明为引用可以避免对元素的拷贝；常量引用主要用来修饰形参，防止误操作

1、std::string的问题：

* 会引发堆内存分配
* 析构函数非平凡，全局对象销毁顺序难以预测，存在生命周期结束后被访问的风险（例如该 `std::string` 被其他全局对象引用）等。

![image-20220526225804213](E:\MarkDown\picture\image-20220526225804213.png)
 也可以string s = {"value"};

![image-20220526230221615](E:\MarkDown\picture\image-20220526230221615.png)

```cpp
string s1, s2;
cin>>s1>>s2;  // 读入时遇见空白符等就停止

getline(cin,s1);// 读取一整行，可以保留空白符，遇见换行符停止
```

遍历每个字符：**范围for**语句（c++11

```cpp
for (auto c : str)
    cout << c << endl;

for (auto &c : str)  // 通过引用可以改变每个字符
    c = toupper(c);  // 这步是将小写字母变为大写字母

for(const auto &s : text){}  // 这里text为存储string的vector
// 因为text的元素是string，可能非常大，将s声明为引用可以避免对元素的拷贝；又因为这里不需要对s进行写操作，因此const

cout<<string[0];  // 通过下标运算符。注意，vector和string的下标运算符可用于访问已存在的元素，而不能用于添加元素
```



2、constexpr char[]

```c++
constexpr char kMyConstString[] = "hello world"; 
```

问题：容易在调用中退化为const char*，导致获取字符串长度的复杂度变为O(n)。为了避免计算长度的开销，调用参数需要一路都额外带一个 `int` 或者 `size_t` 的长度。 

对于const char*：
<img src="E:\MarkDown\picture\image-20220926162849294.png" alt="image-20220926162849294" style="zoom: 67%;" />

```cpp
// 指向的值不可以修改，A对；可以改变其指向，故B错，CD对
/*
结论如下：
    在foo函数中，可以使main函数中p指向的新的字符串常量。
    在foo函数中，可以使main函数中的p指向NULL。
    在foo函数中，可以使main函数中的p指向由malloc生成的内存块，并可以在main中用free释放，但是会有警告。但是注意，即使在foo中让p指向了由malloc生成的内存块，但是仍旧不能用p[1]='x';这样的语句改变p指向的内容。
    在foo中，不能用(*pp)[1]='x';这样的语句改变p的内容。

所以，感觉gcc只是根据const的字面的意思对其作了限制，即对于const char*p这样的指针，不管后来p实际指向malloc的内存或者常量的内存，均不能用p[1]='x'这样的语句改变其内容。但是很奇怪，在foo里面，对p指向malloc的内存后，可以用snprintf之类的函数修改其内容。
*/
#include <stdio.h>
#include <stdlib.h>
#include <stdio.h>
void foo(const char **pp)
{
//    *pp=NULL;
//    *pp="Hello world!";
        *pp = (char *) malloc(10);
        snprintf(*pp, 10, "hi google!");
//       (*pp)[1] = 'x';
}
int main()
{
    const char *p="hello";
    printf("before foo %s/n",p);
    foo(&p);
    printf("after foo %s/n",p);
    p[1] = 'x';
    return;
}
```









3、std::string_view

c++17新特性，较为常用

```c++
constexpr std::string_view myString = "hello world";
auto myString = "hello world"s;  // 自动的话为string

// 这里必须为sv，否则会报错。且编译时必须g++ -std=c++17 test.cpp
// string_view字面量的后缀是 sv。（string字面量的后缀是 s）
constexpr auto myString = "hello world"sv;  // using namespace std::literals
```

有O(1)的方法获取长度



[使用上的注意事项](C++ 更常用 string 还是 char* 呢？ - raaay0608的回答 - 知乎 https://www.zhihu.com/question/483774144/answer/2493158216)



## 6.字符串字面值

" "括起来的0个或多个字符构成字符串型字面值
实际上是由常量字符构成的数组(array)，编译器再每个字符串的结尾处添加一个空字符'\0'，因此其实际长度要比它的内容多1

若两个字符串字面值位置紧邻且仅由空格、缩进和换行符分割，则为一个整体

```cpp
cout<< "aaaaaa" 
    "a a a a ";
```



## 7.初始化

> 初始化和赋值不同，初始化的含义是创建变量时赋予其一个初始值，而赋值的含义是把对象的当前值擦除，而以一个新值替代

> 前面有提到 构造函数的初始化列表 ，是不同的
>
> 可以先了解下[隐式转换](https://www.cnblogs.com/apocelipes/p/14415033.html)

### 1.列表初始化(C++11)、聚合初始化

> c++2.0笔记，补充了4.Unifrom Initialization和5.6.Initializer_list

c++11中用{}来初始化变量，称为列表初始化，可以用来初始化对象

c++17中引入[聚合初始化](https://www.apiref.com/cpp-zh/cpp/language/aggregate_initialization.html)

> 列表初始化是基于隐式转换并以initializer_list为底层实现。（可以写一个initializer_list为形参的构造函数来验证

```cpp
// 初始化方式
int a = 0;
int a = {0};  // 列表初始化
int a{0};  // 列表初始化，也是直接初始化(c++20)的一种形式

// 这里是针对数组或者类类型（通常为结构或者联合）的一种列表初始化形式,详细来说是聚合初始化
// 在聚合初始化期间声明但未显式初始化的数组成员将进行零初始化
int record[26]={};  // 26个0
int myArr3[5] = { 8, 9, 10 };  // 8 9 10 0 0
int intArr1[2][2]{{ 1, 2 }, { 3, 4 }};

// 在c++17中，对auto做了一些改进
auto v   {1};         // v 推导为 int
auto w   {1, 2};      // error: 列表初始化只有为单元素
/* 下面是 C++17 增加的新规则 */
auto x = {1};         // x 推导为 std::initializer_list<int>
auto y = {1, 2};      // y 推导为 std::initializer_list<int>
auto z = {1, 2, 3.0}; // error: 列表初始化中的元素类型必须相同
/*
C++17 对列表初始化规则的增强可以总结为:
	* auto var {one_element}; 将 var 推导为 与 one_element 相同的类型
    * auto var {element1, element2, ...}; 为非法, 编译错误
    * auto var = {element1, element2, ...};将var推导为std::initializer_list<T> 类型, 其中 T 推导为 element 类型
*/


class Complexaa {
public:
	int real;
	std::string aa;

	Complexaa(int a, std::string b):real(a), aa(b) {}
};
int main() {
	Complexaa c1 = {12,"aaa"};  // 列表初始化，对于非聚合类型，直接调用了构造函数
    Complexaa c1{12,"aaa"};  // 列表初始化
	return 0;
}
```

对于内置类型的变量，花括号包围的列表最多包含一个值，且该值所占空间不应该大于目标类型的空间
若使用列表初始化且初始值存在丢失信息的风险，则编译器将报错
![image-20220525204814506](E:\MarkDown\picture\image-20220525204814506.png)

#### 列表初始化详细分类

列表初始化可以分为拷贝初始化和直接初始化两种类型：

​	==对于列表初始化，在类没有用户声明的构造函数时，类是聚合类型，（list中的参数对object成员逐个初始化，若list参数个数小于T成员个数，剩余成员调用value initialization）因此为**拷贝初始化**，使用了=，因此会发生隐式转换，无法使用explicit构造函数。换句话说，无论构造函数是否explicit都会被考虑，但是如果重载决议为一个explicit函数，则此调用错误==
​	==在类有自己定义的构造函数时，通过编译器使用普通的函数匹配来选择与我们提供的参数最匹配的构造函数，并不会先生成临时对象再拷贝，且加不加=没有影响，均不会调用拷贝运算符或拷贝构造函数，为**直接初始化**，无论构造函数是否explicit，都有可能被调用==

相应知识补充：

* 直接初始化（不使用等号）时，我们实际上是要求编译器使用普通的函数匹配来选择与我们提供的参数最匹配的构造函数

* 拷贝初始化（使用等号初始化一个变量）通常使用拷贝构造函数完成。拷贝构造函数不仅在我们用等号=定义变量时调用，在下列情况下也会调用：

  (1) 根据另一个同类型的对象显式或隐式初始化一个对象；

  (2) 将一个对象作为实参传递给一个非引用类型的形参；

  (3) 从一个返回类型为非引用类型的函数返回一个对象；

  (4) **用花括号列表初始化一个数组中的元素或一个聚合类中的成员；**

  (5) 标准库容器初始化，或者调用insert或push成员时，容器会对其元素进行拷贝初始化。

  ```cpp
  A a3 = a2;                  //拷贝初始化1
  A a4 = 2;                   //拷贝初始化2
  A a5 = A(3);                //拷贝初始化3
  ```

  ![image-20220727100603731](E:\MarkDown\picture\image-20220727100603731.png)
  ![image-20220727100929096](E:\MarkDown\picture\image-20220727100929096.png)

* 聚合类型：

1. 类型是一个普通数组，如int[5]，char[]，double[]等
2. 类型是一个类，且满足以下条件：
   * 没有用户声明的构造函数
   * 没有用户提供的构造函数(允许显示预置或弃置的构造函数)
   * 没有私有或保护的非静态数据成员
   * 没有基类
   * 没有虚函数
   * 没有{}和=直接初始化的非静态数据成员
   * 没有默认成员初始化器

对于一个聚合类型，使用列表初始化相当于对其中的每个元素分别赋值；对于非聚合类型，需要先自定义一个对应的构造函数，此时列表初始化将调用相应的构造函数。



#### 列表初始化返回值

> 在返回值类型为对象（不能是对象的引用）的函数中可以返回{}的列表初始化：
> {}返回值的实际类型为initiallizer list（但不能声明为std::initializer_list），所有{}对象都是隐式创建的std::initializer_list类型字面量（都是右值），因此不能是对象的引用

![image-20220606150658602](E:\MarkDown\picture\image-20220606150658602.png)
	若函数返回的是内置类型，则花括号包围的列表最多包含一个值，且该值所占空间不应该大于目标类型的空间（如用{long double}初始化一个int)；若函数返回的是类类型， 则由类本身定义初始值如何使用（如vector< string> v{10,"hi"}表示10个值为hi的元素)

#### 引入std::initializer_list

* initializer_list为一个轻量级STL模板，声明在头文件<initializer_list>中，定义在命名空间std中
* 任意的STL容器都与未指定长度的数组有一样的初始化能力，可以填入任何数量的同类型数据，因此可以用STL容器轻易对固定类型的类进行赋值
* initializer_list是一个轻量级的模板，可以接受任意长度的同类型的数据也就是接受可变长参数，同时作为STL容器它具有STL容器的共同特征（如迭代器）    
  * 只有三个成员接口：begin() end() size()
  * 只能被整体的初始化和赋值，迭代器遍历的数据仅可读，不能对单个数据进行修改
* 所有{}对象都是隐式创建的std::initializer_list类型字面量（右值），广泛用于实现列表初始化（不需要头文件）

==疑问：intializer_list是一个轻量级的模板，可以接受任意长度的同类型的数据，而所有{}对象都是隐式创建的std::initializer_list类型字面量。但像Complexaa c1 = {12,"aaa"}; 这种例子怎么解释？？==

解答：

* 如果实参类型相同，可以传递std::initializer_list类型
* 如果实参类型不同，使用Variadic Templates可变参数模板

```cpp
// 如果实参类型相同，可以传递std::initializer_list类型
class P {
public:
    // 有两个重载版本的构造函数,uniform initialization时优先调用接收initializer list的重载版本
    P(int a, int b) {
        cout << "P(int, int), a=" << a << ", b=" << b << endl;
    }

    P(initializer_list<int> initlist) {
        cout << "P(initializer list<int>), values= ";
        for (auto i : initlist)
            cout << i << ' ';
        cout << endl;
    }
};

P p(77, 5);		// P(int, int), a=77, b=5，小括号，调用第一种情况
P q{77, 5};		// P(initializer list<int>), values= 77 5
P r{77, 5, 42}; // P(initializer list<int>), values= 77 5 42
P s = {77, 5};	// P(initializer list<int>), values= 77 5


//******************************************************************************
// 如果实参类型不同
// 当不写这个构造函数时，是一个聚合类型，对于一个聚合类型，使用列表初始化相当于对其中的每个元素分别赋值；加上构造函数后为非聚合， 对于非聚合类型，需要先自定义一个对应的构造函数，此时列表初始化将调用相应的构造函数。
class Complexaa {
public:
	int real;
	std::string aa;
	
	// 有参构造
	Complexaa(int a, std::string b):real(a), aa(b) {cout << "ctor";}
};
int main() {
	Complexaa c1 = {12,"aaa"};
}



// 如果实参类型不同，使用Variadic Templates可变参数模板也可以实现
// 这是自己猜测的、写出来的一个实现。具体像三个参数就没法这么写了
class Complexaa {
public:
	int real;
	std::string aa;
	
	// 有参构造
	template<typename T, typename... Types>
	Complexaa(int a, std::string b):real(a), aa(b) {cout << "ctor";}
	
	// 可变参数列表，这是按自己想法写的
	template<typename... Arg>
    Complexaa( Arg... args )
    {
		PutArg(args...);
    }

	template <typename Head, typename... Rail>
    void PutArg(Head head, Rail... last)
    {
		real = head;
		PutArg(last...);
    }
    template<typename T>
    void PutArg( T value )
    {
		aa = value;
		cout << "Variadic Templates";
    }
};

int main() {
	Complexaa c1 = {12,"aaa"};  // 输出Variadic Templates
	return 0;
}
```



> 零初始化和[常量初始化(还没怎么看)](https://en.cppreference.com/w/cpp/language/constant_initialization)与下面要讲的默认初始化和值初始化不在一个层次上，前者是静态存储期限对象的行为，后者是初始化器的行为。

### 2.默认初始化

定义变量时**没有指定初值**，则变量被默认初始化，此时变量被赋予了“默认值”，默认值由变量类型决定

* 定义于任何函数体之外的变量被初始化为0
* 定义在函数体内部的内置类型变量将不被初始化。
  局部变量有自动对象和局部静态对象两种，自动对象是只存在于块执行期间的对象，如形参。执行默认初始化时，内置类型的未初始化局部变量将产生未定义的值；
  局部静态对象是定义成static类型的局部静态变量，若没有显式的初始值，将执行值初始化，初始化为0

> 对于类类型，总结一下下图，就是对于类类型，会调用默认构造函数，对于构造函数没赋值的，再递归进行默认初始化；若用的合成的默认构造函数，则对每个对象执行默认初始化。

![image-20220926121432795](E:\MarkDown\picture\image-20220926121432795.png)

### 3.值初始化

就是用数值初始化变量。如果没有给定一个初始值，就会根据变量或类对象的类型提供一个初始值

```cpp
int j = 0; // 值初始化
// 值初始化，可以只提供vector或其他顺序容器(不包括array)对象容纳的元素数量，此时库会创建一个值初始化的元素初值，并把它赋给容器中的所有元素。若vector对象的元素是内置类型，则元素初始值自动设为0；若是某种类类型，如string，则元素由类默认初始化，若有的不支持默认初始化，又明确要求提供初始值，那么这种只提供元素数量的话无法完成初始化操作
// ()表示提供的值是用来构造对象的
vector<int> ivec(10);  // ivec中有10个元素，每个都初始化为0
vector<int> ivec(10,1);  // ivec中有10个元素，每个都初始化为1
vector<string> vv(10);  // 10个元素，每个都是空的string对象

int a(0);
vector<int> ivec2(ivec);  // 拷贝
```

**值初始化的出现场景**

1）在数组初始化的过程中如果我们提供的初始值数量少于数组的大小时，剩余的为提供初始值部分就会执行值初始化。

2）当我们不使用初始值定义一个局部静态变量时。因为是静态的，所以该局部静态变量不再存储在栈区，而是存储在全局区。此时，就会根据类型进行设置该变量的初始值，即值初始化。

3）当我们通过书写形如T()的表达式显示请求值初始化时，其中T是类型名(vector的一个构造函数只接受一个实参用于说明vector的大小)，它就是使用一个这种形式的实参来对它的元素初始器继续值初始化）。
![image-20220926121541659](E:\MarkDown\picture\image-20220926121541659.png)

### 4.类内初始值

对类进行初始化，有构造函数的初始化列表、构造函数函数体内赋值（效率最低）、类内初始值这几种方法

```cpp
// 当我们提供类内初始值时，必须以符号=或者{}表示。类内初始值就是创建类的时候就对里面的成员进行初始化。也可以不设置类内初始值，用构造函数的初始化列表来初始化类内成员
vector<string> a = {"a", "an", "the"};
vector<string> a{"a", "an", "the"};
```

优点：
比如要定义多个构造函数，每个构造函数都用列表初始化的方法初始化会比较麻烦。有类内初始值后就可以简单一些，只改想改的

```cpp
class Group {
public:
    Group() {}
    Group(int a): data(a) {}
    Group(Mem m): mem(m) {}
    Group(int a, Mem m, string n): data(a), mem(m), name(n) {}
private:
    int data = 1;
    Mem mem{0};
    string name{"Group"};
};
```



### 5.拷贝初始化和直接初始化

> 当我们使用=为一个新创建的对象提供初始化器时，会使用拷贝初始化。如果我们向函数传递对象或以传值方式从函数返回对象，以及初始化一个数组或一个聚合类时，也会使用拷贝初始化。

[直接初始化](https://www.apiref.com/cpp-zh/cpp/language/direct_initialization.html)

<img src="E:\MarkDown\picture\image-20220721203416389.png" alt="image-20220721203416389" style="zoom: 67%;" />

<img src="E:\MarkDown\picture\image-20220721203537194.png" alt="image-20220721203537194" style="zoom:67%;" />







## 8.分离式编译

即允许将程序分割为若干个文件，每个文件可被独立编译。因此c++将声明和定义分开

```cpp
//声明可以多次，定义只能一次
//声明，一般写在头文件中
int max(int a, int b);
int max();  // 函数的声明不包含函数体，因此也就无需形参的名字，可以省略
int max(int, int);  // 或者可以只省略形参名
extern int i;  // 声明
class alei{  // 类的定义，通常写在头文件中
    
};

//定义，一般写在cpp文件中
int j;//声明并定义j
int max(int a, int b)
{
	return a > b ? a : b;
}
```

类通常被定义在头文件中，头文件通常包含那些只能被定义一次的实体，如类、const和constexpr变量



## 9.复合类型：引用和指针

复合类型是指基于其他类型定义的类型，如引用和指针

### 1）引用

> **左值**：指求值结果为对象或函数的表达式。一个表示对象的非常量左值可以作为赋值运算符的左侧运算对象。
> **右值**：是指一种表达式，其结果是值而非值所在的位置，通常为字面常量或表达式求值过程中创建的临时对象。只能出现在赋值运算符的右侧。
>
> 左值持久，右值短暂
>
> 这里注意前置和后置递增，前置递增/递减运算符将对象本身作为**左值**返回，后置递增/递减运算符将对象原始值的副本作为**右值**返回。前置版本首先将运算对象加 1（或减 1），然后把改变后的对象作为求值结果。后置版本也将运算对象加 1（或减 1），但是求值结果是运算对象改变之前那个值的副本。因此像是用迭代器遍历容器时，最好用前置版本



#### Lvalue references左值引用

注意

1. 引用并非对象，只是一个别名，所有操作都在与之绑定的对象上进行
2. 引用必须被初始化，如int &a；就是错误的
3. 引用类型的初始值必须是一个对象，不能是字面值或某个表达式的计算结果，如int &a = 10;就是错误的
4. 除了两种特殊情况外（55页，对应黑马/基础入门/10/5，534页），其他所有引用的类型都要和与之绑定的对象严格匹配



（参照黑马/C++基础入门/10/4和5）

#### Rvalue references右值引用

见c++2.0





### 2）指针

```cpp
int *a, *b;
```

1. 指针本身是一个对象，因此允许赋值和拷贝等行为
2. 无需在定义时赋初值
3. 除了两种特殊情况外（56页，534页），其他所有指针的类型都要和它所指向的对象严格匹配，因为在声明语句中指针的类型实际上被用于指定它所指向对象的类型，所以必须匹配
4. 建议初始化所有指针；若使用了未经初始化的指针，则该指针所占内存空间的当前内容会被看作一个地址值，访问该指针，则相当于去访问一个本不存在的位置上的一个可能本不存在的对象。若指针所占内存空间刚好又有内容，就会被当成某个地址，就无法分辨是否合法了

#### 空指针

```cpp
int *a = nullptr;
int *b = 0;
// 需要先#include cstdlib，但实测也不用
int *c = NULL;
```

nullptr是c++11的新关键字，是为了解决NULL表示空指针在C++中函数重载时具有二义性的问题，因此最好使用nullptr表示空指针；也可以通过将指针初始化为字面值0来生成空指针；

也可以用名为NULL的预处理变量来给指针赋值，其值为0。其中预处理器是运行于编译过程之前的一段程序，会自动将预处理变量替换为实际值，因此用NULL初始化指针和用0初始化指针是一样的

[具体解释](https://blog.csdn.net/qq_18108083/article/details/84346655?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-84346655-blog-53667468.pc_relevant_antiscanv3&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-84346655-blog-53667468.pc_relevant_antiscanv3&utm_relevant_index=1)

#### void*指针

可用于存放任意对象的地址

但只能拿它和别的指针比较、作为函数的输入输出、赋值给另一个void* 指针
不能直接操作void* 指针所指向的对象，因为这个对象的类型是未知的，因此无法确定可以做哪些操作

#### 指向指针的指针**

而***表示指向指针的指针的指针
![image-20220526153219882](E:\MarkDown\picture\image-20220526153219882.png)
![image-20220526153243161](E:\MarkDown\picture\image-20220526153243161.png)

*ppi，一次解引用得到指向ival的指针pi，**ppi得到ival的值

#### 指向指针的引用

```cpp
int i = 42;
int *p;
int *&r = p;  // 从右向左阅读，&表明r为一个引用，*表明r引用的是一个指针

r = &i;  // r引用了指针p，这里就相当于p = &i;
*r = 0;  // *p = 0，p又指向i，即i = 0;
```



## 12.size_t和size_type

> 提高代码的可移植性、有效性或者可读性，或许同时提高这三者。
>
> 在数组下标和内存管理函数之类的地方广泛使用

**size_t**

​	定义在cstddef头文件中，不过一般只要include随便一个头文件，就有cstddef了
​	size_t是一种数据相关的unsigned类型，表示**任何对象所能达到的最大长度**。在C++中，设计 size_t 就是为了适应多个平台的，size_t在**32位系统**上定义为 **unsigned int**，也就是**32位无符号整型**。在**64位系统**上定义为 **unsigned long** ，也就是**64位无符号整形**。size_t 的目的是提供一种可移植的方法来声明与系统中可寻址的内存区域一致的长度，编译器会根据不同系统来替换标准类型。size_t的引入增强了程序在不同平台上的**可移植性**。

**size_type**

定义在 < string>和< vector>中，STL类中定义的类型属性，用以保存任意string和vector类对象的长度

```cpp
string::size_type = 2;
std::vector<std::string>::size_type = 1;
```



# Ⅰ c++基础

## 第三章 字符串、向量和数组

### 1.命名空间namespace

C++标准程序库中的所有标识符都被定义于一个名为std（standard）的namespace中。NameSpace（命名空间），是为了解决命名冲突的问题而引入的概念。
标识符是一个由程序员定义的名称，代表程序的某些元素。变量名就是标识符的示例。

预处理变量不属于命名空间std，它由预处理器负责管理。用到一个预处理变量时，预处理器会自动将其替换为实际值

作用：
避免不经意的名字定义冲突，以及使用库中相同名字导致的冲突
标准库定义的所有名字都在命名空间std中

限制：
需要使用 作用域运算符:: 访问命名空间中的函数等，或用using声明（注意，头文件不应包含using声明

用法：

```cpp
#include一些东西
namespace wh{
    ...
    里面模板、类、函数等什么都可以放
}
```

例子：

* 作用域运算符::

```cpp
#include <string>
#include <iostream>
#include <memory> //shared_ptr

namespace jj01
{
class Base1 {  };
class Derived1: public Base1 {  };	
class Base2 {  };
class Derived2: public Base2 {  };

	
template <class T1, class T2>
struct pair {
  typedef T1 first_type;
  typedef T2 second_type;

  T1 first;
  T2 second;
  pair() : first(T1()), second(T2()) {}
  pair(const T1& a, const T2& b) : first(a), second(b) {}

  template <class U1, class U2>
  pair(const pair<U1, U2>& p) : first(p.first), second(p.second) {}
};

void test_member_template()
{
	cout << "test_member_template()" << endl;
    pair<Derived1, Derived2> p; 
    pair<Base1, Base2> p2(pair<Derived1, Derived2>());     
    pair<Base1, Base2> p3(p);   

		
    Base1* ptr = new Derived1; 	//up-cast 
	shared_ptr<Base1> sptr(new Derived1); 	//simulate up-cast
        //Note: make sure your environment support C++2.0 at first.
}
    
} //namespace


int main(int argc, char** argv)  // 这里先不用管
{
	jj01::test_member_template();	
}
```

* using声明

或者直接using namespace std;

![image-20220526224607917](E:\MarkDown\picture\image-20220526224607917.png)

### 2.迭代器

> **迭代器就是对指针的一层封装，提供了统一的接口**。

![image-20220527155052441](E:\MarkDown\picture\image-20220527155052441.png)

迭代器类型：

> 注意，对于迭代器，建议使用**前置**版本的递增递减。后置版本总会拷贝一个副本，对于整数和指针类型，编译器可能会进行一定的优化，但对于迭代器类型，会消耗巨大。

```c++
vector<int>::iterator it = a1.begin();
string::iterator it1;


vector<int> v1;
const vector<int> v2;
vector<int>::const_iterator it2 = v2.begin();  // 只能读不能写，若vector或string对象是一个常量，则只能使用const_iterator
string::const_iterator it3;

auto it4 = v1.begin();  // iterator
auto it5 = v2.begin();  // const_iterator
auto it4 = v1.cbegin();  // auto得到的默认值有时并不是我们想要的，如只需要读操作时，就可以用cbegin和cend，这样auto得到的类型就是const_iterator

for(vector<int>::iterator it = a1.begin();it!=a1.end();++it){}
for(auto it = a1.begin();it!=a1.end();++it)
  *it = toupper(*it);

cout << *it++;  // ++的优先级高于*，相当于*(it++)
```

> 注意，但凡是使用了迭代器的循环体，都不要向迭代器所属的容器添加元素，因为任何一种可能改变对象容量的操作，如push_back，都会使迭代器失效

![image-20220527161615584](E:\MarkDown\picture\image-20220527161615584.png)
这里的类型最好选用auto，因为迭代器相减，得到的两个迭代器间的距离的类型为difference_type的带符号整形数



### 3.vector

```cpp
// 值初始化，可以只提供vector对象容纳的元素数量，此时库会创建一个值初始化的元素初值，并把它赋给容器中的所有元素。若vector对象的元素是内置类型，则元素初始值自动设为0；若是某种类类型，如string，则元素由类默认初始化，若有的不支持默认初始化，又明确要求提供初始值，那么这种只提供元素数量的话无法完成初始化操作
// ()表示提供的值是用来构造对象的
vector<int> v(n);  // n个，都为0
vector<int> v1(n,1);  // n个，都为1
vector<string> vv(10);  // 10个元素，每个都是空的string对象

vector<int> v2(v1);
vector<vector<int>> v1(m, vector<int>(n,0))
    
// 列表初始化
vector<int> vvv{10};  // 一个元素，10
vector<string> a = {"a", "an", "the"};  // 同样的形式可用于类内初始值的提供
vector<string> a{"a", "an", "the"};

// 动态分配对象
std::vector<int> *pv = new std::vector<int>{0,1,2};
std::cout << pv << std::endl;  // 0x1011690，指向的vector对象的地址
std::cout << (*pv)[1] << std::endl;  // 1，*pv通过解引用访问对象，通过[1]是调用对象方法
std::cout << &(*pv)[0] << std::endl;  // 0x10116a8，调用对象方法后再取地址，就是vector所存元素的地址了
std::cout << &(*pv)[1] << std::endl;  // 0x10116ac
delete pv;
```



### 4.数组

> 尽量使用vector和迭代器，而不是数组和指针



一维数组名称的**用途**：

1. 可以统计整个数组在内存中的长度，也可计算数组元素个数(int len = sizeof(array)/sizeof(array[0]);)
2. 可以获取数组在内存中的首地址





数组与vector类似，但区别在于数组的大小固定，因此对某些特殊的应用来说性能较好，但损失了一些灵活性；且数组不允许拷贝和赋值，即不允许将数组内容拷贝给其他数组作为其初始值，也不能用数组为其他数组赋值
![image-20220527163826396](E:\MarkDown\picture\image-20220527163826396.png)

```c++
int a[]={0, 1, 2, 3, 4};
auto b(a);  // 当使用数组作为auto变量的初始值时，推断得到的类型是指针（注意，数组没有这种拷贝操作
// 这里编译器实际执行的操作为auto b(&a[0]);
decltype(a) c;  // 而decltype()得到的就是一个数组了
```



#### 数组作为形参：

![image-20220606094724122](E:\MarkDown\picture\image-20220606094724122.png)





#### 存放指针的数组、数组的指针及引用

```c++
int arr[10];
int * p = arr;  //指向数组的指针，*p为数组第一个元素
int *ptrs[10];  // ptrs是含有10个整形指针的数组
// 数组的维度是紧跟着被声明的名字的，这里由内向外阅读，(*Parray)意味着Parray是一个指针，[10]可知Parray是指向一个大小为10的数组的指针，最后看int知类型。引用同理
int (*Parray)[10] = &arr;  // Parray指向一个含有10个整数的数组
int (&arrRef)[10] = arr;  // arrRef引用一个含有10个整数的数组，这里必须有括号

// 由内向外，先由&知道arry是一个引用，观察右边知道，引用的对象是一个大小为10的数组，最后观察左边，知道数组的元素类型是指向Int的指针
int *(&arry)[10] = ptrs;  // arry引用一个含有十个int型指针的数组

for(auto i:arr)
  cout<<i<<" ";

int ia[3][4];
int i;
for (auto &row : ia)
    for (auto &col : row){
        col = cnt;
        ++cnt;
}
// 不改变值时要写成这种形式，将外层循环的控制变量声明成引用，来避免数组被自动转成指针
// 若for (auto row : ia)，因为row不是引用，所以编译器对其初始化时会自动将其转换成指向该数组内首元素的指针
// 若是多维的形式，除了最内层外其他层都要是引用类型
for (const auto &row : ia)
    for (auto col : row){
        cout << col;
}
```



#### 返回数组指针

![image-20220606174642662](E:\MarkDown\picture\image-20220606174642662.png)

声明一个返回数组指针的函数

![image-20220606174733567](E:\MarkDown\picture\image-20220606174733567.png)
![image-20220606174756370](E:\MarkDown\picture\image-20220606174756370.png)

使用尾置返回类型
![image-20220606153234725](E:\MarkDown\picture\image-20220606153234725.png)

使用decltype![image-20220606174556978](E:\MarkDown\picture\image-20220606174556978.png)

















#### 下标和指针

数组的名字是一个指向数组首元素的指针，因此当对数组使用下标运算符时，其实是对指向数组元素的指针执行下标运算。编译器会自动执行上述转换操作

```c++
int ia[] = {1,2,1,3,1,4,5,6,3};
int i = ia[2];  // ia是指向首元素的指针，ia[2]得到(ia+2)所指的元素
int *p = &ia[2];  // 这里p指向了ia的第三个即索引为2的元素
int j = p[1];  // 这里p[1]等价于*(P+1)，为ia[3];
int k = p[-2];  // ia[0]

void print(const int* beg, const int* end) {
    while (beg != end) cout << *beg++ << endl;
}
print(begin(ia), end(ia));  // 数组也可以用标准库的begin和end函数
```



#### 使用数组初始化vector

```c++
int a[] = {1, 2, 3, 4, 5};
vector<int> v(begin(a),end(a));  // 只需指明要拷贝区域的首尾地址
vector<int> v2(a+1,a+4);
```





## 第四章 表达式



1.求值顺序

大部分运算符没有执行顺序，因此若表达式指向并修改了同一个对象，将会引发错误并产生未定义的行为

```cpp
cout<<i<<++i;  // 这个输出表达式是未定义的，无法推断先执行i还是++i
```

有求值顺序的四个运算符：&&  ||   ?:    ,
![image-20220530162047189](E:\MarkDown\picture\image-20220530162047189.png)

2.sizeof

有sizeof(type)和sizeof expr两种形式
![image-20220530224608879](E:\MarkDown\picture\image-20220530224608879.png)

3.显式转换

> P144
>
> 即强制类型转换(cast)，要尽量避免使用强制类型转换

命名的强制类型转换：cast-name< type >(expression);
type:目标类型；expression:要转换的值；
cast-name：static_cast、dynamic_cast、const_cast、reinterpret_cast











## 第五章 语句

1.遍历每个字符：**范围for**语句（c++11

```cpp
for (auto c : str)
    cout << c << endl;

for (auto &c : str)  // 通过引用可以改变每个字符
    c = toupper(c);  // 这步是将小写字母变为大写字母

for(const auto &s : text){}  // 这里text为存储string的vector
// 因为text的元素是string，可能非常大，将s声明为引用可以避免对元素的拷贝；又因为这里不需要对s进行写操作，因此const

cout<<string[0];  // 通过下标运算符。注意，vector和string的下标运算符可用于访问已存在的元素，而不能用于添加元素
```





## 第六章 函数

### 1、main传递形参

g++编译后，生成a.exe，此时比如输入a -d -o ofile data0，其中a表示可执行文件，后面的为输入argv数组的参数，argc为自动计算的数组参数数量，这里为5，分别为-d -o ofile data0和最后的0。使用时要注意，可选参数从argv[1]开始，argv[0]为程序的名字
![image-20220606104314683](E:\MarkDown\picture\image-20220606104314683.png)

```cpp
#include<iostream>
using namespace std;

// char *argv[]表示指向c风格字符串的指针
int main(int argc, char *argv[]){
	cout << "参数数量：" << argc << endl;
	for(int i = 0; i <= argc; i++) {
		cout << argv[i] << endl;
	}
	return 0;
}
```

### 2、可变形参

#### C++11:std::initializer_list

> 作用：如果函数的实参数量未知，但全部实参的类型都相同，就可以使用此类型的形参，使其可以作用于可变数量的实参

`std::initializer_list<T>` 类型对象是一个访问 `const T` 类型对象数组的轻量代理对象。
其迭代器等操作与vector相同， **与`vector`不同的是，`initializer_list`对象中的元素永远是常量值，我们无法改变`initializer_list`对象中元素的值**。

![image-20220606105550819](E:\MarkDown\picture\image-20220606105550819.png)

```cpp
#include <initializer_list>
```

<img src="E:\MarkDown\picture\image-20220606143354474.png" alt="image-20220606143354474" style="zoom:55%;" />
<img src="E:\MarkDown\picture\image-20220606143406276.png" alt="image-20220606143406276" style="zoom:55%;" />



#### 省略符形参

> 可以用来传递可变数量的实参，但仅用于C和C++通用的类型，为了便于C++程序访问某些特殊的C代码，一般只用于与C函数交互的接口程序

```cpp
void foo(parm_list,...);  // 指定了部分形参的类型，对于这部分要执行类型检查；而省略符形参对应的实参无需类型检查
void foo(...);
```



### 3、6.5.3 assert与NDEBUG

这种方法类似于头文件保护。程序可以包含一些用于调试的代码，这些代码只在开发程序时使用要用到两项预处理功能：

* 预处理宏assert

```cpp
#include<cassert>
assert(expr);  // 表达式为假，assert输出信息并终止程序运行；为真则什么都不做
assert(word.size() > threshold);
```

* NDEBUG预处理变量

若定义了NDEBUG，则不进行DEBUG，assert什么都不做；默认情况下没有定义，assert执行运行时检查

```cpp
// 用法一 关闭调试状态
#define NDEBUG

// 用法二 编写自己的条件调试代码
// NDEBUG未定义，则运行代码
void print(....) {
#ifndef NDEBUG
    cerr << __func__ << size << endl;
#endif
}

// 编译器为每个函数定义了__func__，是const char的一个静态数组，用于存放函数名字
// 预处理器定义了四个对于程序调试很有用的名字
```

<img src="E:\MarkDown\picture\image-20220713105016691.png" alt="image-20220713105016691" style="zoom:37%;" />



### 4.函数指针

> 指向函数而非对象
>
> 指向某种特定类型，由返回类型和形参类型共同决定

```cpp
bool lengthCompare(const string &, const string &);

// 定义
// pf指向一个类型为bool(const string &, const string &)的函数
bool (*pf)(const string &, const string &);

// 使用
pf = lengthCompare;
pf = &lengthCompare;  // 等价，取地址符可选
pf = 0;  // 不指向任何函数

// 创建并赋值
bool (*pf)(const string &, const string &) = lengthCompare;

// 通过函数指针调用函数
bool b1 = pf("hello", "hi");
bool b1 = (*pf)("hello", "hi");  // 等价，解引用可选

// 函数指针作为形参，这里是当作指针使用
// useBigger的声明
void useBigger(const string &s1, const string &s2, bool pf(const string &, const string &)); // pf也可写为(*pf)
    
// 自动将lengthCompare转换成指向该函数的指针
useBigger(s1,s2,lengthCompare);
```

* 通过类型别名和decltype简化函数指针的代码

  在第一行的声明中，编译器自动将Func表示的函数类型转换成指针
  在第二行的声明中，这里decltype返回函数类型，因此加上*才能得到指针<img src="E:\MarkDown\picture\image-20220610180949587.png" alt="image-20220610180949587" style="zoom:47%;" /><img src="E:\MarkDown\picture\image-20220610181109590.png" alt="image-20220610181109590" style="zoom:50%;" />

* 返回指向函数的指针

  ```cpp
  // 通过类型别名简化
  using F = int(int*, int);  // F为函数类型
  using PF = int(*)(int*, int);  // PF为指针类型
  
  PF f(int);  // 返回指向函数的指针，这里f的形参对应指向函数的返回值
  F f(int);  // 错误，不能返回一个函数
  F *f(int);  // 正确，显式地指定返回类型是指向函数的指针
  
  // 直接声明
  int (*f(int))(int*, int);
  
  // 尾置返回类型（c++11，对于返回类型比较复杂的函数最有效
  // 在本应出现返回类型的地方放置一个auto
  auto f(int) -> int(*)(int*, int);
  
  
  // 已知返回函数的情况下，通过decltype简化
  // 注意，decltype()返回函数类型，因此需要加上*显式地表明我们需要返回指针
  string::size_type sumLength(const string&, const string&);
  decltype(sumLength) *getFcn(const string&);
  ```
  

## 第七章 类

### 1.定义类相关的非成员函数

即对于一些属于类的接口组成部分的函数，又不想写在类中。那我们会把这样的接口函数的声明放在类的声明的同一个文件夹内

<img src="E:\MarkDown\picture\image-20220718155513935.png" alt="image-20220718155513935" style="zoom:50%;" />
![image-20220718160032245](E:\MarkDown\picture\image-20220718160032245.png)

### 2.友元类和友元函数

将函数或类在类内用friend声明为友元，即可访问类的非公有成员

​	注意，友元声明在类内出现的位置不限，友元不是类的成员，也不受它所在区域访问控制级别的约束；且==友元的声明仅仅是指定了访问的权限==，我们仍需要**在类外再对函数进行一次声明**，一般将其和类放在同一个文件夹中

```cpp
// 友元类
// 友元类的成员函数可以访问此类所有成员
class a {
	friend class b;  // 将b类声明为a类的友元，b就可以访问a类的私有部分了
}

// 令成员函数作为友元
// 必须先定义b类，并声明clear函数，但不能定义它，并在clear函数使用a类的成员前先声明a类
// 再定义a类，包括对b::clear()的友元声明
// 最后定义clear函数
class a {
	friend void b::clear();  // 将b类的clear成员函数声明为a类的友元
}
```



### 3.类的作用域

函数的返回类型通常出现在函数名之前，因此当成员函数定义在类的外部时，返回类型中的名字位于类的作用域之外，需要指明是哪个类的成员

![image-20220722220505092](E:\MarkDown\picture\image-20220722220505092.png)

```cpp
Window_mgr::ScreenIndex Window_mgr::addScreen() {
    ...
}
```

****

> 注意，类的定义分两步处理:
> ==编译成员的声明 -> 直到类全部可见后才编译函数体==

***

**类型名要特殊处理**

![image-20220722221624045](E:\MarkDown\picture\image-20220722221624045.png)



### 4.委托构造函数

> c++新标准扩展了构造函数初始值（构造函数的初始化列表）的功能，一个委托构造函数可以使用它所属类的其他构造函数执行它自己的初始化过程

![image-20220723210655316](E:\MarkDown\picture\image-20220723210655316.png)

其中，接受istream&的构造函数委托给了默认构造函数，默认构造函数是Sales_data():Sales_data("", 0, 0) {}，又委托给了三参数构造函数，都执行完后再执行{}里的内容

### 5.隐式的类类型转换

​	我们可以为类定义隐式转换规则。如果构造函数只接受一个实参，则它实际上定义了转换为此类型的隐式转换机制，将这种构造函数称为==转换构造函数==。**能通过一个实参调用的构造函数定义了一条从构造函数的参数类型向类类型 隐式转换 的规则。**

![image-20220724205623180](E:\MarkDown\picture\image-20220724205623180.png)

![image-20220724204208235](E:\MarkDown\picture\image-20220724204208235.png)

像 Sales_data a = {"aaa"};这种也是隐式转换，像Sales_data a{"aaa"};、Sales_data a("aaa");就不是

![image-20220724205714116](E:\MarkDown\picture\image-20220724205714116.png)

### 6.explicit

见c++2.0

### 7.类的静态成员/静态数据成员

目的：有时类需要它的一些成员与类本身直接相关，而不是与类的各个对象保持关联。

[见](https://blog.csdn.net/weixin_39731083/article/details/81915954)

**声明静态成员**

​	类体中的数据成员的声明前加上static关键字，该数据成员就成为了该类的静态数据成员。静态成员可以是public或private，类型可以是常量、引用、指针、类类型等。

​	类的静态成员存在于任何对象之外。类似的，静态成员函数也不与任何对象绑定在一起，它们不包含this指针，因此也不能声明成const的

**使用类的静态成员**
<img src="E:\MarkDown\picture\image-20220725201217718.png" alt="image-20220725201217718" style="zoom:67%;" />
<img src="E:\MarkDown\picture\image-20220725201310840.png" alt="image-20220725201310840" style="zoom:60%;" />

**定义静态成员**

![image-20220725203020648](E:\MarkDown\picture\image-20220725203020648.png)

```cpp
// 一般都是类外部定义

//xxx.h文件
class base {
private:
    static int _i;//声明
};

//xxx.cpp文件
// 一般类的声明写在.h中，非内联函数的定义写在.cpp文件中，而静态数据成员的定义也一起放在.cpp文件中
int base::_i=10;  // 定义(初始化)时不受private和protected访问限制。且需要定义在任何函数之外，一旦被定义，就将一致存在于程序的整个生命周期中
```

**静态成员的类内初始化**

> 即使一个常量静态数据成员在类内部被初始化了，通常情况下也应该在类的外部定义一下该成员

![image-20220725203942721](E:\MarkDown\picture\image-20220725203942721.png)

<img src="E:\MarkDown\picture\image-20220725210406893.png" alt="image-20220725210406893" style="zoom:50%;" />![image-20220725211517456](E:\MarkDown\picture\image-20220725211517456.png)

**静态成员可以用于某些场景，而普通成员不能**

> 不完全类型：已经声明但是尚未定义的类型。不完全类型不能用于定义变量或者类的成员，但是用不完全类型定义指针或者引用是合法的

![image-20220726094044832](E:\MarkDown\picture\image-20220726094044832.png)

# Ⅱ c++标准库





## 第九章 顺序容器

​	容器分为顺序容器（又名序列式容器）和关联容器两种。所有容器（除array外）都提供高效的动态内存管理，因此可以向容器中添加元素而不必担心元素存储在哪
​	顺序容器的顺序与元素加入容器时的位置相对应。具有快速顺序访问元素的能力，但存在性能折中：向容器添加或从容器中删除元素的代价；非顺序访问容器中元素的代价。
​	关联容器则根据关键字的值来存储元素

> 标准库提供了三种顺序容器适配器，在后面会详细说
> stack 栈，单口，先进后出，默认基于deque，实质是容器适配器
> queue 队列，双口，先进先出，默认基于deque，实质是容器适配器
> priority_queue，相比queue可以为队列中的元素建立优先级，默认情况下，优先级用元素类型上的小于运算符确定，默认基于vector，和queue都定义在queue头文件中

![image-20220726191455046](E:\MarkDown\picture\image-20220726191455046.png)

***



==所有容器类型都提供的操作==

> 所有迭代器都定义了递增运算符，从当前元素移动到下一个元素。迭代器范围为[begin,end)，这样可以构成一个合法的迭代器范围，即
> <img src="E:\MarkDown\picture\image-20220726195039327.png" alt="image-20220726195039327" style="zoom:50%;" />
>
> ![image-20220726194509755](E:\MarkDown\picture\image-20220726194509755.png)

![image-20220726192134912](E:\MarkDown\picture\image-20220726192134912.png)
![image-20220726192204816](E:\MarkDown\picture\image-20220726192204816.png)
注意，对于关系运算符，需要保证容器中的元素支持，如自定义类型就不可以

![image-20220726192334659](E:\MarkDown\picture\image-20220726192334659.png)
![image-20220726203038593](E:\MarkDown\picture\image-20220726203038593.png)**将一个容器初始化为另一个容器的拷贝:**

* 可以直接拷贝整个容器，需要容器类型及其元素类型必须匹配

* (array除外)拷贝由一个 迭代器对 指定的元素范围。不要求容器类型相同，且元素类型也可以不同，只要能转换就行，如const char*可以转换为string

```cpp
list<const char*> au = {"aa", "bb", "cc"};
queue<string> w(au.begin(), au.end());
```

![image-20220728085016775](E:\MarkDown\picture\image-20220728085016775.png)

* 由于右边运算对象的大小可能与左边运算对象的大小不同，因此array类型不支持assign，也不允许用花括号包围的值列表进行赋值。其中assign操作只需要保证类型相容即可，如char*和string
* 交换两个容器内容的操作保证会很快，因为元素本身并未交换，swap只是交换了两个容器的内部数据结构，不对任何元素进行拷贝、删除或插入，可以保证在常数时间内完成。而对于array，会真正交换他们的元素，因此swap时间与array中元素的数目成正比

***

==顺序容器特有操作：==

**添加元素**

![image-20220728095045914](E:\MarkDown\picture\image-20220728095045914.png)
![image-20220728095841849](E:\MarkDown\picture\image-20220728095841849.png)

* 对于insert操作，如vector不支持push_front，就可以通过v.insert(v.begin(),"aa");来实现相同的功能
* 对于新标准引入的emplace，区别于push和insert传递元素类型的对象，然后将对象拷贝到容器中，或者说创建一个局部临时对象，并将其压入容器中；emplace将参数传递给元素类型的构造函数，在容器管理的内存空间中直接创建对象，因此传递给emplace函数的参数必须与元素类型的构造函数相匹配，`c.emplace_back();  // 使用sales_data的默认构造函数`
  <img src="E:\MarkDown\picture\image-20220728103318333.png" alt="image-20220728103318333" style="zoom:50%;" />

**访问元素**

![image-20220728103436011](E:\MarkDown\picture\image-20220728103436011.png)
访问元素的成员函数返回的是引用，若容器是const对象，则返回值是const的引用，若不是，则是普通引用。因此可以通过引用来接收，就可以更改容器中的元素

```cpp
std::vector<int> v = {1, 2, 3};
auto& vv = v.back();
std::cout << vv << std::endl;
vv = 5;
for(auto i : v) {
	std::cout << i << " ";
}
```

**删除元素**

![image-20220728140232248](E:\MarkDown\picture\image-20220728140232248.png)

**改变容器大小**

![image-20220728180610451](E:\MarkDown\picture\image-20220728180610451.png)

注意，若容器内保存的是类类型元素，且resize向容器添加新元素，则必须提供初始值，或元素类型必须提供一个默认构造函数

**迭代器**

[C++迭代器（STL迭代器）iterator详解](http://c.biancheng.net/view/338.html)



> 仅string、vector、deque、array迭代器支持的操作：
> ![image-20220726194627332](E:\MarkDown\picture\image-20220726194627332.png)

![image-20220726203053733](E:\MarkDown\picture\image-20220726203053733.png)

***

==vector对象是如何增长的==

​	vector和string由于将元素连续存储，若添加元素时空间已满，就需要将已有元素从旧位置移动到新空间中，然后添加元素，释放旧存储空间（**只有操作需求超出vector容量时才会重新分配内存空间**）。为了减少容器空间重新分配次数，vector和string会分配比新的空间需求更大的内存空间（通常是**翻倍**），这就使得虽然vector每次重新分配内存空间时都要移动所有元素，但扩张操作能比list和deque更快
![image-20220728184458898](E:\MarkDown\picture\image-20220728184458898.png)
​	对于reserve函数，仅当需要的内存空间超过当前容量时，其调用才会改变vector的容量；对于shrink_to_fit函数，其目的是退回不需要的内存空间，但具体的实现可以选择忽略此请求，因此调用这个函数并不保证一定退回内存空间

***

==额外的string操作==

**构造**

<img src="E:\MarkDown\picture\image-20220728190432300.png" alt="image-20220728190432300" style="zoom:50%;" />
<img src="E:\MarkDown\picture\image-20220728190602983.png" alt="image-20220728190602983" style="zoom:50%;" />
<img src="E:\MarkDown\picture\image-20220728190624222.png" alt="image-20220728190624222" style="zoom:50%;" />

<img src="E:\MarkDown\picture\image-20220728191441312.png" alt="image-20220728191441312" style="zoom:50%;" />

**substr**

若pos+n大于string的大小，则只拷贝到string末尾

<img src="E:\MarkDown\picture\image-20220728191733079.png" alt="image-20220728191733079" style="zoom:50%;" />

**修改string的操作**

除了常规版本的函数外，还定义了这些额外的版本

s.push_back('a');

s += 'a';(string重载了+运算符)

<img src="E:\MarkDown\picture\image-20220728193629008.png" alt="image-20220728193629008" style="zoom:50%;" />
<img src="E:\MarkDown\picture\image-20220728193700749.png" alt="image-20220728193700749" style="zoom:50%;" />

这里的cp为如下形式：const char *cp = "aaaa";

**string搜索操作**

npos为const string::size_type类型，并初始化为值-1；

<img src="E:\MarkDown\picture\image-20220729152211364.png" alt="image-20220729152211364" style="zoom:50%;" />
<img src="E:\MarkDown\picture\image-20220729152229901.png" alt="image-20220729152229901" style="zoom:50%;" />

**compare函数**

根据s是等于、大于还是小于参数指定的字符串，返回0、正数或负数

<img src="E:\MarkDown\picture\image-20220729154111730.png" alt="image-20220729154111730" style="zoom:50%;" />

**数值转换**

<img src="E:\MarkDown\picture\image-20220729155314173.png" alt="image-20220729155314173" style="zoom:50%;" />

```cpp
string s2 = "pi = 3.14";
d = stod(s2.substr(s2.find_first_of("+-.0123456789")));
```





## 第十章 泛型算法

>  定义在头文件algorithm中。具体算法的介绍见770页附录A、

### 1.初识

​	迭代器令算法不依赖于容器类型，泛型算法本身不会执行容器的操作，只会运行于迭代器之上，因此算法永远不会改变底层容器的大小，永远不会直接添加或删除元素。但算法依赖于元素类型的操作，如==、<等。
​	标准库定义了一类特殊的迭代器，称为插入器，给这类迭代器赋值时，它们会在底层的容器上执行插入操作。因此当一个算法操作一个这样的迭代器时，可以完成向容器添加元素的效果

**1.只读算法**

```cpp
// find，返回指向第一个等于给定值的元素的迭代器
auto result = find(vec.cbegin(), vec.cend(), val);

int ia[] = {1, 2, 3};
int val = 83;
int* result = find(begin(ia), end(ia), val);  // 或(ia+1, ia+3, val)

// accumulate求和，第三个参数为和的初值，其类型决定了函数中使用哪个加法运算符以及返回值的类型
// 这里的vec可以是任何能加到int上的类型
int sum = accumulate(vec.cbegin(), vec.cend(), 0);
// 将v中每个元素连接到string上
// 注意，这里要string("")显式创建。若传递""的话，则是const char*类型，这个类型没有+运算符，将会编译错误
string sum = accumulate(v.cbegin(), v.cend(), string(""));

// equal，确定两个序列是否保存相同的值，对每个元素进行比较。都相等才返回true。第二个序列中的元素数目至少与第一个序列一样多；两个容器中的元素可以是不同类型，如strign和const char*
equal(a.cbegin(), a.cend(), b.cbegin());
```

***

**2.写容器元素的算法**

必须确保序列原大小至少不小于我们要求算法写入的元素数目，因为算法不会执行容器操作，它们自身不可能改变容器的大小。

```cpp
// 将给定范围内元素写入0
fill(vec.begin(), vec.begin() + vec.size()/2, 0);
    
// 将给定值赋予迭代器指向的元素开始的指定个元素
// 注意，不能在空容器上调用fill_n
fill_n(v.begin(), v.size(), 0);

// 拷贝算法copy，返回目的位置迭代器（递增后）的值
auto ret = copy(begin(a1), end(a1), a2);
// replace算法，将所有的0替换成2
replace(l.begin(), l.end(), 0, 2);
// replace_copy，可以保留原序列不变，接受额外第三个迭代器参数，指出调整后序列的保存位置。其中back_insert见 3.额外的迭代器
replace_copy(l.begin(), l.end(), back_inserter(ivec), 0, 2);
```

***

**3.重排容器元素的算法**

```cpp
void elimDups(vector<string> &words) {
	// 排序
    sort(words.begin(), words.end());
    // unique重排输入序列，将相邻的重复项消除(并不是真的删除，而是将其覆盖了)，并返回一个指向不重复值范围末尾的迭代器
    auto end_unique = unique(words.begin(), words.end());
    // erase删除重复单词
    words.erase(end_unique, words.end());
}
```

<img src="E:\MarkDown\picture\image-20220731191907526.png" alt="image-20220731191907526" style="zoom:40%;" />
![image-20220731191921122](E:\MarkDown\picture\image-20220731191921122.png)

***

### 2.==可调用对象==

> C++中有如下几种可调用对象：**函数、函数指针、lambda表达式、bind对象、函数对象**
>
> **函数对象**
>
> 重载了函数调用运算符()的类的对象，即为函数对象
>
> **函数指针**
>
> ```cpp
> // 回顾一下函数指针
> # include <iostream>
> 
> // 被调用的函数
> int fun(int x, int y) {
>     std::cout << x + y << std::endl;
> 	return x + y;
> }
> 
> // 形参为函数指针
> int fun1(int (*fp)(int, int), int x, int y) {
> 	return fp(x, y);
> }
> 
> // 定义一个函数指针类型Ftype
> typedef int (*Ftype)(int, int);
> int fun2(Ftype fp, int x, int y) { 
> 	return fp(x, y);
> }
> 
> int main(){
>     // 一般对于函数来说，函数名即为函数指针
> 	fun1(fun, 100, 100);
> 	fun2(fun, 200, 200);
> }
> ```

#### **1.谓词 -> 向算法传递函数**

> 我们可以通过额外的参数 **谓词** 来使用函数的其他重载过的版本
>
> 注意，如sort函数，需要接收一个二元谓词。因此若是在一个类中（如leetcode）定义了一个成员函数来作为谓词，则必须加上static，使该成员函数变为静态成员函数。因为如果我们定义了一个两个参数的函数，因为它是类的成员函数，所以实际的参数是三个，即加上了this，这样传到sort中就会报错。加上static后变为静态成员函数， 就没有了this指针这个参数，变回二元谓词。

****

​	分为一元谓词和二元谓词，即接受一个或两个参数。谓词是一个**可调用的表达式**，返回结果是一个能用作条件的值

```cpp
// 二元谓词
// 注意，这里const string &s1要用引用，否则y
bool isShorter(const string &s1, const string &s2) {
	return s1.size() < s2.size();
}
// 按长度从小到大排序
sort(w.begin(), w.end(), isShorter);
// 在保留原相对顺序的同时，按长度从小到大排序，即相同长度的字符的顺序还是原顺序
stable_sort(w.begin(), w.end(), isShorter);
```

#### ==2.function函数对象==

​	由于可调用对象的定义方式比较多，但是函数的调用方式较为类似，因此需要使用一个统一的方式保存可调用对象或者传递可调用对象。于是，c++11中引入了std::function

​	std::function是一个可调用对象包装器，是一个类模板，可以容纳除了**类成员函数指针**之外的所有可调用对象，它可以用统一的方式处理函数、函数对象、函数指针，并允许保存和延迟它们的执行。

```cpp
// 定义function的一般形式
# include <functional>
std::function<函数类型>
    
# include <iostream>
# include <functional>

typedef std::function<int(int, int)> comfun;

// 普通函数
int add(int a, int b) { return a + b; }

// lambda表达式
auto mod = [](int a, int b){ return a % b; };

// 函数对象类
struct divide{
    int operator()(int denominator, int divisor){
        return denominator/divisor;
    }
};

int main(){
	comfun a = add;
	comfun b = mod;
	comfun c = divide();
    std::cout << a(5, 3) << std::endl;
    std::cout << b(5, 3) << std::endl;
    std::cout << c(5, 3) << std::endl;
}
```

​	std::function可以取代函数指针的作用，因为它可以延迟函数的执行，特别适合作为回调函数使用。它比普通函数指针更加的灵活和便利。

故而，std::function的作用可以归结于：

* std::function对C++中各种可调用实体(普通函数、Lambda表达式、函数指针、以及其它函数对象等)的封装，形成一个新的可调用的std::function对象，简化调用；
* std::function对象是对C++中现有的可调用实体的一种类型安全的包裹(如：函数指针这类可调用实体，是类型不安全的)。



#### ==3.lambda表达式==

> c++11
>
> lambda表达式是一个可调用对象，可以用于算法。底层依赖函数对象的机制实现

> lambda表达式的不可替代性：
> 	算法只能接受一元谓词**或**二元谓词，如find_if只能接受一元谓词，equal接受二元谓词。这就导致想要传递参数可能会超出算法对谓词的限制，如想用find_if接受一个string和一个长度，来判断string的长度是否大于给定长度时，就无法传递第二个参数来表示长度。而lambda表达式就可以通过捕获列表得到更多的参数，这也是函数所做不到的（因为算法大都在一个函数里面调用，而函数虽然可以接受全局变量作额外参数，但很不方便，没法像lambda的捕获列表一样灵活。后面会讲如何用函数代替含捕获列表的lambda表达式）
>
> ​	lambda表达式常用于只在一两个地方使用的简单操作。在lambda的捕获列表为空的情况下，若要在很多地方使用相同的操作，通常应该定义一个函数。类似的，如果一个操作需要很多语句才能完成，通常使用函数更好

> 补充：
> 	对于一个对象或一个表达式，如果可以对其使用调用运算符，则称其为可调用的。如e为一个可调用的表达式，则可以e(args)

<img src="E:\MarkDown\picture\image-20220731212448277.png" alt="image-20220731212448277" style="zoom:40%;" />

```cpp
// 注意，如果一个lambda体包含return之外的任何语句，则编译器假定此lambda返回void，如{ if (i < 0) return -i; else return i; }就是错误的，因为被推断为void，但返回了int值
// 若需要定义返回类型，必须使用尾置返回类型
transform(v.begin(), v.end(), v.begin(),
         [](int i) -> int
         { if (i < 0) return -i; else return i; });
```

```cpp
/************单独使用一个lambda表达式******************/
// 捕获列表(常为空)和函数体不能省略
// 这里f是个可调用对象
auto f = []{return 2;};
cout << f();

/*****************lambda表达式作为谓词*****************/
// 空捕获列表表明此lambda不使用它所在函数中的任何局部变量
// 注意，lambda不能有默认参数
[](const string &s1, const string &s2) 
	{return s1.size() < s2.size();}
// 调用
stable_sort(w.begin(), w.end(), 
           [](const string &s1, const string &s2) 
			{return s1.size() < s2.size();} );
// 与函数作为谓词的对比
bool isShorter(const string &s1, const string &s2) 
{ return s1.size() < s2.size(); }
stable_sort(w.begin(), w.end(), isShorter);


void aa(const string &s) {
	cout << s << " ";
}

/*********************使用捕获列表******************/
// 这里使用了所在函数的中的局部变量
[sz](const string &a)
	{ return a.size() >= sz; };
// 我们想构建一个函数，目的是求大于等于一个给定长度的单词有多少，并使程序只打印大于等于给定长度的单词
void biggies(vector<string> &words, vector<string>::size_type sz) {
	elimDups(words);
    stable_sort(words.begin(), words.end(), isShorter);
    // 获取一个迭代器，指向第一个满足size() >= sz的元素
    // find_if接受一个unaryPred，即一元谓词
    auto wc = find_if(words.begin(), words.end(), 
		[sz](const string &a)
			{ return a.size() >= sz; });
    // 计算满足size() >= sz的元素的数目
    auto count = words.end() - wc;
    // 打印长度大于等于给定值的单词
    // for_each接受一个unaryOp,即使用来自输入序列的一个实参来调用的可调用对象（谓词是可调用表达式，lambda是可调用对象，但这两个没啥差别，需要可调用对象的for_each可以输入一个谓词，需要谓词的find_if也可以接受一个lambda可调用对象。因此这两个东西使用的时候不用去区分
    // for_each(wc, words.end(),aa);
    for_each(wc, words.end(),
             [](const string &s) {cout << s << " ";});
}
```

***

**lambda是函数对象**

​	当我们编写了一个lambda后，编译器将该表达式翻译成一个未命名类的未命名对象。在lambda表达式产生的类中含有一个重载的函数调用运算符。

```cpp
stable_sort(w.begin(), w.end(), 
           [](const string &s1, const string &s2) 
			{return s1.size() < s2.size();});
// 如果一个类将()运算符重载为成员函数，这个类就称为函数对象类，这个类的对象就是函数对象。
// 函数对象是一个对象，但是使用的形式看起来像函数调用，实际上也执行了函数调用
// lambda表达式的行为类似于下面这个函数对象类的一个未命名对象，或者称为函数对象
// 这个类只有一个函数调用运算符成员，因为默认情况下lambda不能改变捕获的变量，因此是const成员函数；如果lambda被声明为可变的，则调用运算符就不是const的了
class ShorterString {
public:
    bool operator()(const string &s1, const string &s2) const
		{ return s1.size() < s2.size();};
};
// ShorterString()是新构造的对象
stable_sort(w.begin(), w.end(), ShorterString());

// 对于引用捕获，编译器直接使用该引用而无需在lambda产生的类中将其存储为数据成员
// 对于值捕获，变量会被拷贝到Lambda中，因此会为每个值捕获的变量建立对应的数据成员，同时创建构造函数
auto wc = find_if(words.begin(), words.end(), 
		[sz](const string &a)
			{ return a.size() >= sz; });
class SizeComp{
public:
    SizeComp(size_t n):sz(n) {}
    bool operator()(const string &a) const
			{ return a.size() >= sz; }
private:
    size_t sz;
}
auto wc = find_if(words.begin(), words.end(), SizeComp(sz));
```

***

**lambda捕获和返回**

​	由上面的分析可知，当定义一个lambda时，编译器生成一个与lambda对应的新的（未命名的）类类型。因此可以认为，当向一个函数传递一个lambda时，同时定义了一个新类型和该类型的一个对象。
​	因此默认情况下，从lambda生成的类都包含一个lambda所捕获的变量的数据成员，且在lambda对象创建时被初始化

**值捕获**

采用值捕获的前提是**变量可以拷贝**。被捕获的变量的值在lambda创建时就拷贝
![image-20220801140835383](E:\MarkDown\picture\image-20220801140835383.png)

**引用捕获**

​	需要确保被引用的对象在lambda执行的时候是存在的，因为lambda捕获的都是局部变量，在函数结束后就没了

​	注意，当从一个函数返回lambda时，与函数不能返回一个局部变量的引用类似，此lambda也不能包含引用捕获

```cpp
void fcn2() {
	size_t v1 = 42;
    auto f2 = [&v1] { return v1; };
    v1 = 0;
    cout << f2(); // 0
}

// 引用捕获的必要性
// 如捕获一个ostream的引用来输出数据
// ostream对象不能拷贝，因此捕获os的唯一方法就是捕获其引用，或指向os的指针
void biggies(......., ostream &os = cout, char c = ' ') {
    .......
	for_each(words.begin(), words.end(), 
            [&os, c](const string &s) { os << s << c; });
}
```

**隐式捕获**

​	通过在捕获列表中写一个=或&，让编译器根据函数体中的代码推断捕获列表。注意，混合使用隐式捕获和显式捕获时，捕获列表的第一个元素必须是&或=，且显式捕获的变量必须使用与隐式捕获不同的方式

```cpp
// os隐式捕获，引用捕获方式；c显式捕获，值捕获方式
for_each(......,
        [&, c](const string &s) { os << s << c; });
```

![image-20220801142534458](E:\MarkDown\picture\image-20220801142534458.png)

**可变lambda**

​	对于值捕获，一般不会改变其值，但我们可以在参数列表的后面加上关键字mutable来改变这个被捕获的变量的值

```cpp
void fcn3() {
	size_t v1 = 40;
    auto f = [v1] () mutable { return ++v1; };
    v1 = 0;
    auto j = f();  // j为41
}
```

***

#### ==4.bind绑定器==

**绑定器的两个阶段**

1.c++STL中的绑定器：
bind1st : 把operator()的第一个形参变量绑定成一个确定的值
bind2nd : 把operator()的第二个形参变量绑定成一个确定的值

2.c++11从Boost库中引入了std::bind绑定器和std::function函数对象机制到标准库中

**标准库bind函数**

​	定义在头文件functional中。可以用来生产一个可调用对象来适应原对象的参数列表。
​	可以将bind函数看作一个通用的函数适配器，它接受一个可调用对象，生成一个新的**可调用对象**来“适应”原对象的参数列表。

```cpp
#include<functional>
using namespace std::placeholders;
auto newCallable = bind(callable, arg_list);
```

​	newCallable为一个可调用对象，arg_list是一个逗号分隔的参数列表，对应给定的callable参数。即当我们调用newCallable时，newCallable会调用callable，并传入arg_list中的参数。其中arg_list中的参数可能包含形如_n的名字作为占位符，如 _1为newCallable的第一个参数

```cpp
#include<functional>
// 示例：我们想在words容器中找一个元素小于sz的数
// 这时候可以使用find_if来查询，但find_if的输入需要是个二元的函数对象，而我们的需求是要求一元函数对象。这就造成了一个问题：我们不能使用库里提供的greater、less这些函数，因为都是二元的

// 普通方法：用lambda表达式写一个一元函数对象
auto wc = find_if(words.begin(), words.end(), 
		[sz](const string &a)
			{ return a.size() >= sz; });

// 进阶方法：用函数代替含捕获列表的lambda表达式
/******************使用STL中的绑定器*******************/
// bind + 二元函数对象 =》 一元函数对象
// bind1st + greater, 即 sz > b，找到第一个符合条件的就是小于sz的
// bind2nd + less, 即 sz > 70
auto wc = find_if(words.begin(), words.end(), 
		bind1st(greater<int>(), sz));
auto wc = find_if(words.begin(), words.end(), 
		bind2nd(less<int>(), sz));
/******************使用c++11中的bind绑定器******************/
// 示例一：用greater
#include<algorithm>
using namespace std;
int main() {
    // _n为参数占位符，定义在名为placeholders的命名空间中，而这个命名空间又定义在std命名空间中，因此要对每个_n提供单独的using声明
	// using std::placeholders::_1;
	// 或者直接下面这样
	using namespace std::placeholders;
	vector<int> words = {1, 3, 5, 7};
	auto it = find_if(words.begin(), words.end(), 
		bind(greater<int>(), 3, _1));
	cout << *it;
}
// 示例二：自写二元函数
bool mygreater(int sz, int b) {return sz > b;}
auto it = find_if(words.begin(), words.end(), 
		bind(mygreater, 3, _1));

/*******************小例子*********************/
// 这里g需要两个参数，f有五个参数，g的第二和第一个参数作为f的第三个和第五个参数传递过去
auto g = bind(f, a, b, _2, c, _1);
// bind可以来重排参数顺序，如
sort(w.begin(), w.end(), isShorter);
sort(w.begin(), w.end(), bind(isShorter, _2, _1));

// 直接调用了函数
void hello(string str) {cout << str << endl;}
bind(hello, "嗷嗷嗷")();
bind(hello, _1)("嗷嗷嗷");
/*******************绑定引用参数*********************/
class Test {
public:
    int sum(int a, int b) {return a + b;}
};
// 对于类的成员方法，前面要取个地址，且需要绑定一个Test对象才能用，因此多了个Test()
cout << bind(&Test::sum, Test(), 20, 30)();

/*******************function*********************/
// function函数对象类型，将bind返回的绑定器复用起来了
function<void(string)> func1 = bind(hello, _1);
func1("aa");
func2("bbbbb");


for_each(words.begin(), words.end(), 
	[&os, c](const string &s) { os << s << c; });
ostream &print(ostream &os, const string &s, char c) {
	return os << s << c;
}
// 注意，bind拷贝其参数，若希望传递一个引用，需要用标准库ref函数（和bind一样定义在functional头文件中），其返回一个对象，包含给定的引用；若要传递一个const引用，需要使用cref
for_each(words.begin(), words.end(), bind(print, ref(os), _1, ' '));
```

[bind原理图示](https://www.cnblogs.com/xusd-null/p/3698969.html)

### 3.额外的迭代器

![image-20230307183456830](E:\MarkDown\picture\image-20230307183456830.png)

除了为每个容器定义的迭代器外，标准库在头文件iterator中还定义了额外几种迭代器

* 插入迭代器：迭代器适配器，这些迭代器被绑定到一个容器上，可用来向容器插入元素
* 流迭代器：被绑定到输入或输出流上，可用来遍历所关联的IO流
* 反向迭代器：迭代器适配器，迭代器向后而不是向前移动。除了forward_list之外的标准库容器都有反向迭代器
* 移动迭代器：迭代器适配器，这些专用的迭代器不少拷贝其中的元素，而是移动它们（480页

***

**插入迭代器**

​	是一种迭代器适配器，它接受一个容器，生成一个迭代器，能实现向给定容器添加元素。
​	**插入迭代器**可以保证算法有足够元素空间来容纳输出数据，是一种向容器中添加元素的迭代器，当我们通过一个插入迭代器赋值时，一个与赋值号右侧值相等的元素被添加到容器中![image-20220801215504669](E:\MarkDown\picture\image-20220801215504669.png)

插入器有三种类型：

* **back_inserter**创建一个使用push_back的迭代器
* **front_inserter**创建一个使用push_front的迭代器（前提是容器支持push_front），元素总是插入到容器第一个元素之前
* **inserter**创建一个使用inserter的迭代器。此函数接受第二个参数，这个参数必须是一个指向给定容器的迭代器。元素将被插入到给定迭代器所表示的元素之前

**back_inserter**

![image-20220730224353140](E:\MarkDown\picture\image-20220730224353140.png)

**inserter**

```cpp
vector<int> v;
auto iter = v.begin();
auto it = inserter(v, iter);

*it = val;  // 将元素插入到iter原来所指向元素之前的位置

// 等价于
it = c.insert(it, val);  // 此时在it指向元素之前创建一个val，返回指向新元素位置的迭代器
++it;  // 指向原来的元素

```

**front_inserter**

元素总是插入到容器第一个元素之前

```cpp
list<int> lst = {1, 2, 3, 4};
list<int> lst2, lst3;
// lst2变为4 3 2 1
copy(lst.cbegin(), lst.cend(), front_inserter(lst2));
```

***

**iostream迭代器**

​	iostream类型不是容器，但标准库定义了可以用于这些IO类型对象的迭代器。**istream_iterator**读取输入流，**ostream_iterator**向一个输出流写数据。这些迭代器将它们对应的流当作一个特定类型的元素序列来处理。通过使用流迭代器，我们可以用泛型算法从流对象读取数据以及向其写入数据

**istream_iterator**

​	创建一个流迭代器时，必须指定迭代器将要读写的对象类型。且istream_iterator使用>>来读取流，因此要读取的类型必须定义了输入运算符
![image-20220802105013312](E:\MarkDown\picture\image-20220802105013312.png)

```cpp
// 将istream_iterator绑定到一个流。对于一个绑定到流的迭代器，一旦其关联的流遇到文件尾或遇到IO错误，迭代器的值就与尾后迭代器相等
// istream_iterator允许使用懒惰求值，即绑定到一个流时，标准库并不保证迭代器立即从流读取数据，具体实现可以推迟从流中读取数据，直到我们使用迭代器时才真正读取
istream_iterator<int> int_it(cin);
// 默认初始化迭代器，会被定义为空的istream_iterator，从而可以作为尾后迭代器使用
istream_iterator<int> int_eof;
while (int_it != int_eof)
    vec.push_back(*int_it++);  // 这里后置运算符从流中读取下一个值，但返回的是迭代器的旧值，因此解引用就是之前读的值
// 或者直接从迭代器范围构造vec
vector<int> vec(int_it, int_eof);
// 也可以使用算法操作流迭代器
cout << accumulate(int_it, int_eof, 0);

ifstream in("afile");  // 打开给定文件
istream_iterator<string> str_it(in);
```

**ostream_iterator**

​	可以对任何具有输出运算符(<<)的类型定义ostream_iterator。且允许提供一个字符串作为第二参数，在输出每个元素后都会打印此字符串。此字符串必须是一个C风格字符串，即一个字符串字面常量或一个指向以空字符结尾的字符数组的指针
​	区别于istream_iterator，ostream_iterator不允许空的或表示尾后位置

![image-20220802141234985](E:\MarkDown\picture\image-20220802141234985.png)

```cpp
// 将vec中每个元素写到cout，每个元素后加个空格
ostream_iterator<int> out_iter(cout, " ");
for (auto e : vec)
    // *out_iter++ = e;  // 赋值语句实际上将元素写到cout
    out_iter = e;  // 可以忽略解引用和递增运算
// 或者直接copy，更为简单
copy(vec.begin(), vec.end(), out_iter);
cout << endl;
```

​	注意，*和++实际上对ostream_iterator对象不做任何事，因此可以忽略。但还是推荐第一种写法，这样流迭代器的使用与其他迭代器保持一致。

重写书店程序：

<img src="E:\MarkDown\picture\image-20220802143134075.png" alt="image-20220802143134075" style="zoom:60%;" />
<img src="E:\MarkDown\picture\image-20220802143022703.png" alt="image-20220802143022703" style="zoom:50%;" />

***

**反向迭代器**

​	反向迭代器：迭代器适配器，在容器中从尾元素向首元素反向移动，递增迭代器会移动到前一个元素，递减会移动到下一个元素。除了forward_list之外的标准库容器都有反向迭代器，可以通过rbegin、crbegin、rend、crend等成员函数获得
​	反向迭代器需要递减运算符，而forward_list和流迭代器都不支持递减运算，因此不能从forward_list和流迭代器创建反向迭代器



<img src="E:\MarkDown\picture\image-20220802175905510.png" alt="image-20220802175905510" style="zoom:45%;" />

![image-20220802180058774](E:\MarkDown\picture\image-20220802180058774.png)
<img src="E:\MarkDown\picture\image-20220802180118225.png" alt="image-20220802180118225" style="zoom:45%;" />

**注意事项**

**rcomma和rcomma.base()指向不同的元素**

<img src="E:\MarkDown\picture\image-20220802182004332.png" alt="image-20220802182004332" style="zoom:40%;" />

```cpp
// 有一个名为line的string，保存着一个逗号分隔的单词列表
// 打印line中第一个单词
auto comma = find(line.cbegin(), line.cend(), ',');
cout << string(line.cbegin(), comma) << endl;
// 打印line中最后一个单词
auto rcomma = find(line.crbegin(), line.crend(), ',');
// 错误，这会从后往前逆序输出单词
cout << string(line.crbegin(), rcomma) << endl;
// 我们需要将comma转换为一个普通迭代器，可以通过reverse_iterator的base成员函数完成这一转换吗，此成员函数会返回普通迭代器
cout << string(rcomma.base(), line.cend()) << endl;
```

***

### 4.泛型算法结构

​	泛型算法，即类型无关的算法，算法通过在迭代器上进行操作来实现类型无关
​	算法所要求的迭代器操作可分为5个迭代器类别：输入迭代器、输出迭代器(输入迭代器和输出迭代器比较特殊，它们不是把数组或容器当做操作对象，而是把输入流/输出流作为操作对象)、前向迭代器、双向迭代器、随机访问迭代器。==这里迭代器是按它们所提供的操作来分类的，这种分类形成了一种层次，除了输出迭代器外(和输入迭代器同级)，一个高层类别的迭代器支持底层类别迭代器的所有操作==
![image-20220802184046376](E:\MarkDown\picture\image-20220802184046376.png)
<img src="E:\MarkDown\picture\image-20220802185835334.png" alt="image-20220802185835334" style="zoom:60%;" />
<img src="E:\MarkDown\picture\image-20220802185904950.png" alt="image-20220802185904950" style="zoom:60%;" />



<img src="E:\MarkDown\picture\image-20220802094629229.png" alt="image-20220802094629229" style="zoom:50%;" />

容器适配器 stack 和 queue 没有迭代器，它们包含有一些成员函数，可以用来对元素进行访问

### 5.特定容器算法

​	对于list和forward_list，这两个类型分别提供双向迭代器和前向迭代器，而像通用版本的sort要求随机访问迭代器，因此无法使用，故定义了这些成员函数版本的算法。因此优先使用成员函数版本的算法，性能也会更高。

<img src="E:\MarkDown\picture\image-20220802191726107.png" alt="image-20220802191726107" style="zoom:47%;" />

![image-20220802190939427](E:\MarkDown\picture\image-20220802190939427.png)
![image-20220802191006594](E:\MarkDown\picture\image-20220802191006594.png)

此为链表特有的操作：

![image-20220802191621807](E:\MarkDown\picture\image-20220802191621807.png)

***



## 第十一章 关联容器

### 1.概述

​	顺序容器中的元素是按它们在容器中的位置来顺序保存和访问的；而关联容器中的元素是按关键字来保存和访问的，按关键字==有序==保存元素（默认情况下标准库使用关键字类型的<运算符来比较两个关键字，即从小到大），**支持高效的关键字查找和访问**。map（关联数组）中的元素是关键字-值(key-value)对；set只包含一个关键字，支持高效的关键字查询操作
​	关联容器支持295页表9.2的普通容器操作，不支持顺序容器的位置相关的操作，如push_back等
​	关联容器的==迭代器都是双向==的

![image-20220802193857492](E:\MarkDown\picture\image-20220802193857492.png)

<img src="E:\MarkDown\picture\image-20220802194006066.png" alt="image-20220802194006066" style="zoom:50%;" />

**定义**

```cpp
map<string, string> a = {
    {"key", "value"},
    {"aa", "bb"}
};
```

#### **有序容器关键字类型的要求**

​	关键字类型必须定义元素比较的方法
<img src="E:\MarkDown\picture\image-20220802201757408.png" alt="image-20220802201757408" style="zoom:40%;" />

```cpp
bool cc(const Sales_data &lhs, const Sales_data &rhs) {
    return lhs.isbn() < rhs.isbn();
}
// 以isbn的顺序进行排序
// 为了使用自己定义的操作，需要额外提供一个比较操作类型——即一个函数指针类型，指向那个函数
// decltype选择并返回操作数的数据类型
// decltype(函数)返回函数类型，因此需要加上*显式地表明我们需要返回指针
// 这里用cc来初始化a对象，这里构造函数的参数可以用cc来代替&cc，因为函数名会自动转化为一个指针
multiset<Sales_data, decltype(cc)*> a(cc);
```

**pair类型**

​	定义在头文件utility中。pair的默认构造函数对数据成员进行值初始化

![image-20220802205416399](E:\MarkDown\picture\image-20220802205416399.png)

```cpp
pair<string, int> process(vector<string> &v) {
	if (!v.empty())
        return {v.back(), v.back().size()};  // 列表初始化
    	// return pair<string, int>(v.back(), v.back().size());  // 显式构造
    	// return make_pair(v.back(), v.back().size());
    else
        return pair<string, int>();  // 隐式构造一个空pair
}
```



### 2.关联容器的操作



==关联容器==

![image-20220726192605465](E:\MarkDown\picture\image-20220726192605465.png)
![image-20220726192623659](E:\MarkDown\picture\image-20220726192623659.png)





除了第九章表9.2中所有容器公有的操作和类型外
<img src="E:\MarkDown\picture\image-20220803092136310.png" alt="image-20220803092136310" style="zoom:45%;" />
<img src="E:\MarkDown\picture\image-20220803092409729.png" alt="image-20220803092409729" style="zoom:45%;" />

> 注意，pair的关键字部分是const的，因此map.first是不能改变的；set中的关键字也是const的，虽然set类型同时定义了iterator和const_iterator，但都只允许只读访问set中元素



**关联容器和算法**

通常不对关联容器使用泛型算法，而是使用其定义的成员函数

<img src="E:\MarkDown\picture\image-20220803093422573.png" alt="image-20220803093422573" style="zoom:50%;" />

***

**添加元素**

> emplace与insert的区别：emplace可以避免不必要的临时对象的产生
>
> insert：Value to be copied (or moved) to the inserted elements.
> emplace：在构造新元素时避免不必要的复制或移动操作。
> emplace使用了 C++11 的两个新特性 **变参模板** 和 **完美转发**。”变参模板”使得 emplace 可以接受任意参数，这样就可以适用于任意对象的构建； ”完美转发”使得接收下来的参数 能够原样的传递给对象的构造函数，这带来另一个方便性就是即使是构造函数声明为 explicit 它还是可以正常工作，因为它不存在临时变量和隐式转换。通过std::forward转发调用新元素的构造函数。即使容器中已有包含这个关键字的元素，也可能构造元素，该情况下新构造的元素将被立即销毁
>
> ```cpp
> struct Bar {
>     Bar(int a) {}
>     explicit Bar(int a, double b) {}
> };
> 
> int main(void)
> {
>     vector<Bar> bv;
>     bv.push_back(1);        // 隐式转换生成临时变量
>     bv.push_back(Bar(1));   // 显示构造临时变量
>     bv.emplace_back(1);     // 没有临时变量
> 
>     //bv.push_back({1, 2.0});   //  无法进行隐式转换
>     bv.push_back(Bar(1, 2.0));  //  显示构造临时变量
>     bv.emplace_back(1, 2.0);    //  没有临时变量
> 
>     return 0;
> }
> ```

<img src="E:\MarkDown\picture\image-20220803094056034.png" alt="image-20220803094056034" style="zoom:67%;" />

```cpp
// 向map添加元素
w.insert({word, 1});
w.insert(pair<string, size_t>(word, 1));
w.insert(make_pair(word, 1));
w.insert(map<string, size_t>::value_type(word, 1));

map<string, complex<double>> scp;
scp.insert({"world", {1, 2}});
// 对于map来说，想要通过emplace避免临时变量的构造会比较麻烦：
// map 类型的 emplace 处理比较特殊，因为和其他的容器不同，map 的 emplace 函数把它接收到的所有的参数都转发给 pair 的构造函数。对于一个 pair 来说，它既需要构造它的 key 又需要构造它的 value。如果我们按照普通的 的语法使用变参模板，我们无法区分哪些参数用来构造 key, 哪些用来构造 value。
// 解决方式是使用 C++11 中提供的 tuple ,既可以接受异构变长参数，又可以区分 key 和 value
// 这种方式是有问题的，因为这里有歧义，第一个 tuple 会被当成是 key，第二 个tuple会被当成 value。最终的结果是类型不匹配而导致对象创建失败，为了解决 这个问题，C++11 设计了 piecewise_construct_t 这个类型用于解决这种歧义，它 是一个空类，存在的唯一目的就是解决这种歧义，全局变量 std::piecewise_construct 就是该类型的一个变量。
scp.emplace(piecewise_construct,
            make_tuple("hello"),
            make_tuple(1, 2));
// 可以使用 forward_as_tuple 替代 make_tuple，该函数会帮你构造一个 tuple 并转发给 pair 构造。
scp.emplace(piecewise_construct,
            forward_as_tuple("hello"),
            forward_as_tuple(1, 2));

```

**insert的返回值**

​	返回值依赖于容器类型和参数。对于**不包含重复关键字**的容器，**添加单一元素的insert和emplace版本返回一个pair**，first成员是一个迭代器，指向具有给定关键字的元素；second成员是一个bool值，指出元素是插入成功还是已存于容器中。若关键字已在容器中，则insert什么事情也不做，且bool部分为false；若关键字不存在，元素被插入容器中，且bool值为true
​	对于multimap和multiset，返回一个迭代器

```cpp
map<string, size_t> count;
string word;
while(cin >> word) {
    auto ret = count.insert({word, 1});
    if (!ret.second)
        ++ret.first->second;  // 递增计数器，即count里的value
}

std::multimap<std::string, std::string> a;
auto ret = a.insert({"aa", "bb"});
auto ret2 = a.insert({"aa", "cc"});
std::cout << ret->first << " " << ret->second << std::endl;
std::cout << ret2->first << " " << ret2->second << std::endl;
```

***

**删除元素**

对于erase(k)，若返回值为0，表明想要删除的元素并不在容器中

![image-20220803135317354](E:\MarkDown\picture\image-20220803135317354.png)

***

**map的下标操作**

​	对于multimap和unordered_multimap不能进行下标操作，因为可能有多个值与一个关键字相关联
​	对于下标操作，存在的一个问题就是如果关键字不在map中，下标操作会插入一个新元素，其值为0，因此检查一个元素是否存在时一般会用find

![image-20220803140126545](E:\MarkDown\picture\image-20220803140126545.png)

​	通常情况下，解引用一个迭代器所返回的类型与下标运算符返回的类型是一样的。但对于map进行下标操作，会获得一个mapped_type对象；解引用map迭代器时，会得到一个value_type对象
![image-20220803141847085](E:\MarkDown\picture\image-20220803141847085.png)

***

**访问元素**

![image-20220803142013792](E:\MarkDown\picture\image-20220803142013792.png)
![image-20220803142045574](E:\MarkDown\picture\image-20220803142045574.png)

​	对于multi中有多个元素具有给定关键字，则这些元素在容器中会相邻存储，因此可以find找到第一个关键字，再通过count遍历；也可以通过lower_bound和upper_bound来遍历；也可以直接通过equal_range获得

```cpp
for (auto beg = a.lower_bound(search_item), end = a.upper_bound(search_item); beg != end; ++beg)
    cout << beg->second << endl;

for (auto pos = a.equal_range(search_item); pos.first != pos.second; ++pos.first)
    cout << pos.first->second;
```

***

**单词转换程序练习**

```cpp
// 392页
#include<fstream>
#include<map>
#include<sstream>
const string &
transform(const string &s, const map<string, string> &m) {
	auto map_it = m.find(s);
	if (map_it != m.cend())
		return map_it->second;
	else
		return s;
}
map<string, string> buildMap(ifstream &map_file) {
	map<string, string> trans_map;
	string key;
	string value;
	// ifstream从文件读，用法可以看作cin >> a
	// getline函数属于string的操作，从一个给定的istream读取一行数据，存入一个给定的string对象中
	// cin读入时遇见空白符等就停止，getline读取一整行，可以保留空白符，遇见换行符停止
	// 这里读取第一个单词存入key中，行中剩余内容存入value
	while (map_file >> key && getline(map_file, value))
		if (value.size() > 1)
			// 跳过前面的空格
			trans_map[key] = value.substr(1);
		else
			throw runtime_error("no rule for "+key);
	return trans_map;	
}
void word_transform (ifstream &map_file, ifstream &input) {
	auto trans_map = buildMap(map_file);
	string text;
	while (getline(input, text)) {
		istringstream stream(text);
		string word;
		bool firstword = true;  // firstword则不打印空格，不是firstword则在单词间打印一个空格
		while (stream >> word) {
			if (firstword)
				firstword = false;
			else
				cout << " ";
			cout << transform(word, trans_map);
		}
		cout << endl;
	}
}

int main(){
	ifstream a("C:\\Users\\yzew\\Desktop\\a.txt");
	ifstream b("C:\\Users\\yzew\\Desktop\\b.txt");
	word_transform(a, b);
	return 0;
}
```

***

### 3. 无序容器

> 无论在有序容器中还是在无序容器中，具有相同关键字的元素都是相邻存储的

​	无序容器使用一个哈希函数（将给定类型的值映射到整形(size_t)值的函数）即hash<key_type>类型的对象和关键字类型的==运算符来组织元素。用哈希技术而不是比较操作来存储和访问元素，因此这类容器的性能依赖于哈希函数的质量
​	通常可以用一个无序容器替换对应的有序容器，因为它们都提供了相同的操作，但无序容器的输出通常会与有序容器的版本不同

**管理桶**

​	无序容器在存储上组织为一组桶，每个桶保存零个或多个元素。无序容器使用一个哈希函数将元素映射到桶。为了访问一个元素，容器首先计算元素的哈希值，它指出应该搜索哪个桶。容器将具有一个特定哈希值的所有元素都保存在相同的桶中。如果容器允许重复关键字，所有具有相同关键字的元素也都会在同一个桶中。因此，无序容器的性能依赖于哈希函数的质量和桶的数量和大小。
​	对于相同的参数，哈希函数必须总是产生相同的结果。理想情况下，哈希函数还能将每个特定的值映射到唯一的桶。但是，将不同关键字的元素映射到相同的桶也是允许的。当一个桶保存多个元素时，需要顺序搜索这些元素来查找我们想要的那个。计算一个元素的哈希值和在桶中搜索通常都是很快的操作。但是，如果一个桶中保存了很多元素，那么查找一个特定元素就需要大量比较操作。

![image-20220803163142039](E:\MarkDown\picture\image-20220803163142039.png)

#### 无序容器对关键字类型的要求

​	默认情况下，无序容器使用关键字类型的==运算符来比较元素，并使用一个hash<key_type>类型的对象来生成每个元素的哈希值。标准库为**内置类型**（包括指针）提供了hash模板，还为一些标准库类型，包括**string**和**智能指针**类型定义了hash，因此可以直接定义这三个类型的无序容器
​	想要定义关键字类型为自定义类类型的无序容器，需要自己提供hash模板版本（626页

这里不使用默认的hash，而是使用类似于为有序容器重载关键字类型的默认比较操作

```cpp
// 哈希值计算函数
// hasher函数使用一个建立在string类型上的标准库hash类型对象来计算ISBN成员的哈希值
size_t hasher(const Sales_data &sd) {
	return hash<string>()(sd.isbn());
}
// 用eqOp函数替代==运算符
bool eqOp(const Sales_data &lhs, const Sales_data &rhs) {
    return lhs.isbn() == rhs.isbn();
}
// 定义类型别名来简化
using a = unordered_multiset<Sales_data, decltype(hasher)*, decltype(eqOp)*>;
// 参数是桶大小、哈希函数指针和相等性判断运算符指针
a bookstore(42, hasher, eqOp);

// 若类定义了==运算符，则可以只重载哈希函数
unordered_multiset<Sales_data, decltype(hasher)*> bookstore(42, hasher);
```













### 3.使用标准库：文本查询程序



























​	返回值优化（Return value optimization，缩写为RVO）是C++的一项编译优化技术，即删除保持函数返回值的临时对象。它通过转换源代码和对象的创建来加快源代码的执行速度。当函数需要返回一个对象的时候，如果自己创建一个临时对象用户返回，那么这个临时对象会消耗一个构造函数（Constructor）的调用、一个复制构造函数的调用（Copy Constructor）以及一个析构函数（Destructor）的调用的代价。而如果稍微做一点优化，就可以将成本降低到一个构造函数的代价，这样就省去了一次拷贝构造函数的调用和一次析构函数的调用。

C++标准允许省略这些复制构造函数，即使这导致程序的不同行为，即使编译器把两个对象视作同一个具有副作用





# Ⅲ 类设计者的工具

## 第十三章 拷贝控制

> ​	当定义一个类时，我们显式或隐式地指定在此类型的对象拷贝、移动、赋值和销毁时做什么。一个类通过定义五种特殊的成员函数来控制这些操作：**拷贝构造函数、拷贝赋值运算符、移动构造函数、移动赋值运算符、析构函数**。我们称这些操作为**拷贝控制操作**



### 1.拷贝控制操作

### **拷贝构造函数**

> 拷贝构造函数在拷贝初始化、值传递的方式作为函数参数等几种情况下都会被隐式的调用，因此通常不应该是explicit的
>
> 为什么拷贝构造函数的参数必须是引用类型？
> 	在函数调用中，具有**非引用类型**的参数要进行拷贝初始化。拷贝构造函数被用来初始化非引用类类型参数。如果拷贝构造函数的参数不是引用类型，则其调用永远不会成功： 为了调用拷贝构造函数，我们需要拷贝它的实参，但为了拷贝实参，我们有需要调用拷贝构造函数，无限循环。

```cpp
// 一拷贝构造函数的第一个参数是自身类类型的引用，且 任何 额外参数都有默认值
// 一般都为 const T&
/*
主要有两种情形会隐式调用单参数构造函数：
 （1）同类型对象的拷贝构造；即用相同类型的其它对象来初始化当前对象;
 （2）不同类型对象的隐式转换。即其它类型对象隐式调用单参数拷贝构造函数初始化当前对象
*/
class Foo{
    Foo(const Foo&);  
};
```

**拷贝初始化**

​	对于直接初始化(无=)，实际上是要求编译器使用普通的函数匹配来选择与提供的参数最匹配的构造函数；而拷贝初始化是要求编译器将右侧运算对象拷贝到正在创建的对象中，有需要的话还要进行类型转换。拷贝初始化依靠拷贝构造函数或移动构造函数来完成·

​	拷贝初始化（使用等号初始化一个变量）通常使用拷贝构造函数完成。拷贝构造函数不仅在我们用等号=定义变量时调用，在下列情况下也会调用：

(1) 根据另一个同类型的对象显式或隐式初始化一个对象；

(2) 将一个对象作为**实参**传递给一个**非引用类型的形参**；

(3) 从一个返回类型为非引用类型的函数返回一个对象；

(4) **用花括号列表初始化一个数组中的元素或一个聚合类中的成员；**

(5) **标准库容器初始化，或者调用insert或push成员时，容器会对其元素进行拷贝初始化；与之相对，用emplace成员创建的元素都进行直接初始化**

```cpp
A a3 = a2;                  //拷贝初始化1
A a4 = 2;                   //拷贝初始化2
A a5 = A(3);                //拷贝初始化3
```



```cpp
// 拷贝构造函数会被隐式的调用（如在拷贝初始化的时候），因此不应该是explicit的
#include<iostream>
using namespace std;
class test{
public:
	test(int);
	explicit test(const test&);  // 定义为仅显示调用后，会影响第3和第4个例子
	int a;
};
test::test(int t):a(t) {
	cout << "yc" << endl;
}
test::test(const test& t):a(t.a) {
	cout << "copy" << endl;
}

int main() {
	test t1 = 10;  // 拷贝初始化，先将10通过有参构造函数转化，test t1 = test(10)，再隐式调用拷贝构造函数，test t1(test(10))。但这里编译器进行了一些优化，直接变为test t1(10)，调用的是 // 有参构造函数 //。但要求拷贝构造函数必须是存在且可访问的
	test t2(t1);  // 直接初始化，编译器使用普通的函数匹配来选择最匹配的构造函数，这里是 // 显式调用拷贝构造函数  //
	test t2 = t1;  // 拷贝初始化，相当于test t2(t1)，一般会通过拷贝或移动构造函数完成，这里是隐式调用拷贝构造函数
	test t3 = test(t1);  // 拷贝初始化，隐式调用拷贝构造函数，变为test t3(test(t1))
    
    test t2(10);  // 直接初始化，将t1的形式改写为这样后，可以使编译器略过拷贝/移动构造函数，但拷贝构造函数必须是存在且可访问的。如不能是private的
    
}
```



### **合成拷贝构造函数**

> ​	对于一个空类，如果你自己没声明，编译器就会为它声明(编译器版本的)一个copy构造函数、一个拷贝赋值运算符和一个析构函数。此外如果你没有声明任何构造函数，编译器也会为你声明一个default构造函数。所有这些函数都是public且inline。注意，**当这些函数被需要(调用)，它们才会被编译器创建出来。**
>
> ​	编译器合成的拷贝操作有三种情况，要么被定义为逐成员拷贝，要么被定义为对象赋值，要么被定义为删除的函数

**合成的拷贝构造函数**：会将其参数的非static成员逐个拷贝到正在创建的对象中：
	对于类类型的成员如string，会使用其拷贝构造函数来拷贝；对于内置类型成员则直接拷贝；
	对于数组，虽然不能直接拷贝，但合成拷贝构造函数会逐元素地拷贝一个数组类型的成员；而若数组元素是类类型，则使用元素的拷贝构造函数来进行拷贝。

对某些类来说，合成拷贝构造函数用来阻止我们拷贝该类类型的对象
<img src="E:\MarkDown\picture\image-20220809204856660.png" alt="image-20220809204856660" style="zoom:45%;" />



***



### **拷贝赋值运算符**

```cpp
Sales_data a, b;
a = b;  // 使用Sales_data类的拷贝赋值运算符，若类未定义，则由编译器合成

// 注意拷贝赋值运算符和拷贝构造函数调用的区别
// 有新对象被定义，一定会有个构造函数被调用
// 如果没有新对象被定义，就是赋值操作
Sales_data a(b);  // 拷贝构造函数
a = b;  // 拷贝赋值运算符
Sales_data a = b;  // 拷贝构造函数
```

重载运算符

​	重载运算符本质上是一个函数，名字由operator关键字后接要定义的运算符的符号。重载运算符的参数表示运算符的运算对象。像赋值运算符等必须定义为成员函数，其左侧运算对象就绑定到隐式的this参数上。对于一个二元运算符如赋值运算符，其右侧运算对象作为显式参数传递

```cpp
class Foo {
public:
    Foo& operator=(const Foo&);  // 拷贝赋值运算符，接收一个与其所在类相同类型的参数，返回一个指向其左侧运算对象的引用
};
```

### **合成拷贝赋值运算符**

> 合成的拷贝操作有三种情况：
>
> * 被定义为逐成员拷贝
> * 被定义为对象赋值
> * 被定义为删除的函数

​	在类未定义拷贝赋值运算符时，编译器会自动生成合成拷贝赋值运算符。类似拷贝构造函数，对于某些类，合成拷贝赋值运算符会被定义为删除的，用来禁止该类型对象的赋值(P450)。

对某些类来说，编译器将这些合成的成员定义为删除的函数:

* 如果**类的某个成员的析构函数是删除的或不可访问的**（例如，是private的），则类的**合成析构函数**被定义为删除的。
* 如果类的某个成员的**拷贝构造函数是删除的或不可访问的**，则类的合成拷贝构造函数被定义为删除的。如果类的某个成员的析构函数是删除的或不可访问的，则类合成的拷贝构造函数也被定义为删除的。
* 如果类的某个成员的拷贝赋值运算符是删除的或不可访问的，或是类有一个const的或引用成员，则类的合成拷贝赋值运算符被定义为删除的。
* 如果类的某个成员的析构函数是删除的或不可访问的，或是类有一个引用成员，它没有类内初始化器(参见2.6.1节，第65页)，或是类有一个const成员，它没有类内初始化器且其类型未显式定义默认构造函数，则该类的默认构造函数被定义为删除的



​	如果拷贝赋值运算符并非出于此目的，它会将右侧运算对象的每个非static成员赋予左侧运算对象的对应成员，这一工作是通过成员类型的拷贝赋值运算符来完成的。对于数组类型的成员,逐个赋值数组元素。合成拷贝赋值运算符返回一个指向其左侧运算对象的引用。
<img src="E:\MarkDown\picture\image-20220905201112378.png" alt="image-20220905201112378" style="zoom:40%;" />

***

### **析构函数**

​	析构函数释放对象使用的资源，并销毁对象的非static数据成员。
​	在一个构造函数中，成员的初始化是在构造函数的函数体执行之前（如列表初始化，若没有的话就默认初始化）完成的，且**按照它们在类中出现的顺序进行初始化**（初始化之后就开始执行函数体内的赋值、打印信息之类的操作）；在一个析构函数中，首先执行函数体（进行一些空间释放等操作，释放对象在生存期分配的所有资源），然后销毁成员，进行销毁的这个析构部分是**隐式**的，成员按初始化顺序的**逆序销毁**。销毁类类型的成员需要执行成员自己的析构函数，内置类型没有析构函数，因此销毁内置类型成员什么都不需要做。
​	对于普通指针，隐式**销毁一个内置指针类型的成员不会delete它所指向的对象**；而智能指针是类类型，有析构函数，智能指针成员在析构阶段会被自动销毁。

**析构函数的调用**

无论何时一个对象被销毁，就会自动调用其析构函数：

* 变量在**离开其作用域**时被销毁
* 当一个对象被销毁时，其成员被销毁
* 容器（包括数组）被销毁时，其元素被销毁
* 对于动态分配的对象，当对指向它的指针应用delete运算符时被销毁
* 对于临时对象，当创建它的完整表达式结束时被销毁

当指向一个对象的引用或指针离开作用域时，析构函数不会执行

### **合成析构函数**

​	类似拷贝构造函数和拷贝赋值运算符，对于某些类，合成析构函数被用来阻止该类型的对象被销毁(参见13.1.6 节，第450页)。如果不是这种情况，合成析构函数的函数体就为空。要注意成员是在析构函数体之后隐含的析构阶段中被销毁的，函数体自身并不直接销毁成员



***

### **三/五法则**

**需要析构函数的类，几乎可以肯定它也需要拷贝构造函数和拷贝赋值运算符（深浅拷贝）。**如一个类，若在构造函数中分配动态内存，则需要定义一个析构函数来释放内存，此时若不定义自己的拷贝构造函数和拷贝赋值运算符，则会出现多个对象指向相同内存的情况，在析构时就会进行多次delete。因此通常管理类外资源的类必须定义拷贝控制成员。（（例外是基类的析构函数，它需要设定为虚函数，但不一定需要拷贝操作））

**需要拷贝操作的类也需要赋值操作，反之亦然。**即如果一个类需要一个拷贝构造函数，几乎可以肯定它也需要一个拷贝赋值运算符，反之亦然。而都不必然意味着也需要析构函数。如作为一个例子，考虑一个类为每个对象分配一个独有的、 唯一的序号。 这个类需要一个拷贝构造函数为每个新创建的对象生成一个新的、独一无二的序号。除此之外，这个拷贝构造函数从给定对象拷贝所有其他数据成员。这个类还需要自定义拷贝赋值运算符来**避免**将序号赋予目的对象。但是，这个类不需要自定义析构函数。

**所有五个拷贝控制成员应该看作一个整体：一般来说，如果一个类定义了任何一个拷贝操作，它就应该定义所有五个操作**(详见后面的移动操作)

***

**=default, =delete**

见cpp2.0第九节

***

### 2.拷贝控制和资源管理

**行为像值的类**

> 对于行为像值的类，需要额外定义拷贝构造函数、拷贝赋值运算符及析构函数

每个对象都应拥有一份自己的拷贝，如HasPtr类：

* 定义一个拷贝构造函数，完成string的拷贝，而不是拷贝指针
* 定义一个拷贝赋值运算符来释放对象当前的string，并从右侧运算对象拷贝string
* 定义一个析构函数来释放string

```cpp
class HasPtr {
public:
    HasPtr(const string &s = string()) : ps(new string(s)), i(0) {}
    HasPtr(const HasPtr&p) : ps(new string(*p.ps)), i(p.i) {}
    HasPtr& operatror=(const HasPtr &);
    ~HasPtr() {delete ps;}
private:
    string *ps;
    int i;
};

// 通过先拷贝右侧运算对象，可以处理自赋值情况
HasPtr& HasPtr::operator=(const HasPtr &rhs) {
    // 先将右侧运算对象拷贝到一个局部临时对象中，当拷贝完成后，销毁左侧运算对象的现有成员就是安全的了，因为像自赋值的话，先销毁左侧资源时再拷贝，就会访问一个指向无效内存的指针。
    auto newp = new string(*rhs.ps);   // 拷贝底层string
    delete ps;  // 释放旧内存
    ps = newp;  // 从右侧运算对象拷贝数据到本对象
    i = rhs.i;
    return *this;
}
```

**行为像指针的类**

拷贝指针成员本身而不是它指向的string。本例中设计自己的**引用计数**来直接管理资源<img src="E:\MarkDown\picture\image-20220906154714872.png" alt="image-20220906154714872" style="zoom:40%;" />

```cpp
// 计数器应当保存在动态内存中。在创建对象时分配一个新的计数器，在拷贝或赋值对象时，拷贝指向计数器的指针
class HasPtr {
public:
    // 将计数器置为一
    HasPtr(const string &s = string()) : ps(new string(s)), i(0), use(new size_t(1)) {}
    HasPtr(const HasPtr&p) : ps(p.ps), i(p.i), use(p.use) { ++*use; }
    HasPtr& operatror=(const HasPtr &);
    ~HasPtr();
private:
    string *ps;
    int i;
    size_t *use;  // 用来记录有多少个对象共享*ps的成员
};

HasPtr::~HasPtr() {
    if(--*use == 0) {
		delete ps;
        delete use;
    }
}

// 通过先递增rhss中计数再递减左侧运算对象中的计数，可以处理自赋值情况
HasPtr& HasPtr::operator=(const HasPtr &rhs) {
    ++*rhs.use;
    if (--*use == 0) {
        delete ps;
        delete use;
    }
    ps = rhs.ps;
    i = rhs.i;
    use = rhs.use;
    return *this;
}
```

### 3.交换操作

> 除了定义拷贝控制成员，管理资源的类通常还定义一个名为swap的函数。 swap并不是重要的，但对于分配了资源的类，定义swap可能是一种很重要的优化手段。

​	如果一个类定义了自己的swap，那么算法将使用类自定义版本，否则算法将使用标准库定义的swap。理论上，交换两个对象需要进行的一次拷贝和两次赋值所发生的内存分配是不必要的。我们更希望swap交换指针，而不是分配新副本

```cpp
class HasPtr {
    // 定义为friend，便于访问HasPtr的private的数据成员
    friend void swap(HasPtr&, HasPtr&);
    // 在赋值运算符中使用swap
    // 这个技术的有趣之处是它自动处理了自赋值情况且天然就是异常安全的
    // 它通过在改变左侧运算对象之前拷贝右侧运算对象保证了自赋值的正确，这与我们在原来的赋值运算符中使用的方法是一致的(第453页)。它保证异常安全的方法也与原来的赋值运算符实现一样。代码中唯一可能抛出异常的是拷贝构造函数中的new表达式。如果真发生了异常，它也会在我们改变左侧运算对象之前发生
    HasPtr& HasPtr::operator=(HasPtr rhs) {
        swap(*this, rhs);
        return *this;
    }
};

// 重载swap的默认行为
// 由于swap的存在就是为了优化代码，因此声明为inline
inline void swap(HasPtr& lhs, HasPtr& rhs) {
    using std::swap;
    // 每个swap调用应该都是未加限定的，即每个调用都应该是swap，而不是std::swap
    // 这样的话，若存在类型特定的swap版本，其匹配程度会优于std中定义的版本。因此若存在类型特定的swap版本，swap调用会与之匹配，若不存在，则会使用std中的版本
    swap(lhs.ps, rhs.ps);  // 交换指针。这里调用的是std::swap
    swap(lhs.i, rhs.i);
}

// Foo的swap函数
// Foo有一个类型为HasPtr的成员h
void swap(Foo &lhs, Foo &rhs) {
    // 正常来说，在内层作用域中声明了std::swap，将隐藏外层作用域中的HasPtr版本的swap
    // 但对于命名空间中名字的隐藏规则来说有一个重要的例外，它使得我们可以直接访问输出运算符，即当我们给函数传递一个类类型的对象时，除了在常规的作用域查找外，还会查找实参类所属的命名空间。这一例外对于传递类的引用或指针的调用同样有效
    using std::swap;
    swap(lhs.h, rhs.h);  // 这里调用的就是HasPtr版本的swap
}
```

**使用示例**

拷贝控制示例、动态内存管理类见P460

> 需要定义拷贝控制成员的情形：
>
> * 需要分配资源的类，即进行资源管理
> * 帮助进行簿记工作或其他操作

### 4.移动构造函数和移动赋值运算符

见c++2.0 23

> 一般来说，拷贝一个资源会导致一些额外开销。在这种拷贝并非必要的情况下，定义了移动构造函数和移动赋值运算符的类就可以避免此问题。

**合成的移动操作**

​	与拷贝操作不同，编译器根本不会为某些类合成移动操作。特别是，如果一个类定义了自己的拷贝构造函数、拷贝赋值运算符或者析构函数，编译器就不会为它合成移动构造函数和移动赋值运算符了。因此，某些类就没有移动构造函数或移动赋值运算符。如我们将在第477页所见，如果一个类没有移动操作，通过正常的函数匹配，类会使用对应的拷贝操作来代替移动操作。
​	**只有当一个类没有定义任何自己版本的拷贝控制成员，且类的每个非static数据成员都能移动构造或移动赋值时，编译器才会为它合成移动构造函数或移动赋值运算符。**编译器可以移动内置类型的成员。如果一个成员是类类型，且该类有对应的移动操作，编译器也能移动这个成员:<img src="E:\MarkDown\picture\image-20221004105936644.png" alt="image-20221004105936644" style="zoom:47%;" />

​	与拷贝操作不同，移动操作永远不会隐式定义为删除的函数。但是，如果我们显式地要求编译器生成=default的移动操作，且编译器不能移动所有成员，则编译器会将移动操作定义为删除的函数。除了一个重要例外，什么时候将合成的移动操作定义为删除的函数遵循与定义删除的合成拷贝操作类似的原则：

* 与拷贝构造函数不同，移动构造函数被定义为删除的函数的条件是:有类成员定义了自己的拷贝构造函数且未定义移动构造函数，或者是有类成员未定义自己的拷贝构造函数且编译器不能为其合成移动构造函数。移动赋值运算符的情况类似。

* 如果有类成员的移动构造函数或移动赋值运算符被定义为删除的或是不可访问的，则类的移动构造函数或移动赋值运算符被定义为删除的。
* 类似拷贝构造函数，如果类的析构函数被定义为删除的或不可访问的，则类的移动构造函数被定义为删除的。
* 类似拷贝赋值运算符，如果有类成员是const的或是引用，则类的移动赋值运算符被定义为删除的。

> 注意，如果类定义了一个移动构造函数和/或一个移动赋值运算符，则该类的合成拷贝构造函数和拷贝赋值运算符会被定义为删除的。**定义了一个移动构造函数或移动赋值运算符的类必须也定义自己的拷贝操作。否则，这些成员默认地被定义为删除的。**
>
> **如果一个类有一个可用的拷贝构造函数而没有移动构造函数，则其对象是通过拷贝构造函数来“移动”的。拷贝赋值运算符和移动赋值运算符的情况类似：**
> 	正常来说，是移动右值，拷贝左值。如果一个类既有移动构造函数，也有拷贝构造函数，编译器使用普通的函数匹配规则来确定使用哪个构造函数。赋值操作的情况类似。例如我们的StrVec类中，拷贝构造函数接受一个const: StrVec的引用，可以用于任何可以转换为StrVec的类型，比如也可以接受右值，只需要进行一次到const的转换；而移动构造函数接受一个StrVec&&，因此只能用于实参是(非static)右值的情形。**但如果没有移动构造函数，右值也会被拷贝。**即使传入的是std::move()的一个右值，调用的也将是拷贝构造函数，且用拷贝构造函数代替移动构造函数几乎肯定是安全的


<img src="E:\MarkDown\picture\image-20221004143539064.png" alt="image-20221004143539064" style="zoom:70%;" />



### 5.移动迭代器

​	新标准库中定义了一种移动迭代器(move iterator) 适配器，这些专用的迭代器不是拷贝其中的元素，而是移动它们。一个移动迭代器通过改变给定迭代器的解引用运算符的行为来适配此迭代器。一般来说，一个迭代器的解引用运算符返回一个指向元素的左值。与其他迭代器不同，移动迭代器的解引用运算符生成一个右值引用。
​	我们通过调用标准库的make_move_iterator函数将一个普通迭代器转换为一个移动迭代器。此函数接受一个迭代器参数，返回一个移动迭代器。原迭代器的所有其他操作在移动迭代器中都照常工作。由于移动迭代器支持正常的迭代器操作，我们可以将一对移动迭代器传递给算法。特别是，可以将移动迭代器传递给
uninitialized_ copy：

```cpp
auto first = alloc.allocate(newcapacity);
auto last = uninitialized_copy(make_move_iterator(begin()),make_move_iterator(end()),first);
```

uninitialized copy对输入序列中的每个元素调用construct来将元素“拷贝”到目的位置。此算法使用迭代器的解引用运算符从输入序列中提取元素。由于我们传递给它的是移动迭代器，因此解引用运算符生成的是一个右值引用，这意味着construct将使用移动构造函数来构造元素。
值得注意的是，标准库不保证哪些算法适用移动迭代器，哪些不适用。由于移动一个对象可能销毁掉原对象，因此你只有在确信算法在为一个元素赋值或将其传递给一个用户定义的函数后不再访问它时，才能将移动迭代器传递给算法。

### 6.移动和拷贝的重载函数

> 区分移动和拷贝的重载函数通常有一个版本接受一个const T&，而另一个版本接受一个T&&。

​	除了构造函数和赋值运算符之外，如果一个成员函数同时提供拷贝和移动版本，它也能从中受益。这种允许移动的成员函数通常使用与拷贝/移动构造函数和赋值运算符相同的参数模式，即一个版本接受一个指向const的左值引用，第二个版本接受一个指向非const的右值引用

```cpp
void push_back(const X&);  // 拷贝：绑定到任意类型X。这里const是因为从一个对象进行拷贝的操作不应该改变该对象
void push_back(X&&);  // 移动：只能绑定到类型X的可修改(非const)的右值：要窃取数据时，通常传递一个右值引用，因此实参不能是const的。可以vec.push_back("done")，从"done"创建的临时string是个右值
```



### 7.引用限定符

> 通过在参数列表后放置一个**引用限定符**来指出this的左值/右值属性：&来强制左侧运算对象(即返回的*this)是一个左值，&&指出this可以指向一个右值。引用限定符只能用于(非static)成员函数，且必须同时出现在函数的声明和定义中。
>
> 注意，一个函数同时用const和引用限定时，引用限定符必须跟随在const限定符后
>
> ```cpp
> Foo aaa() const &;
> ```
>
> 

<img src="E:\MarkDown\picture\image-20221004190911565.png" alt="image-20221004190911565" style="zoom:45%;" />

```cpp
class Foo {
public:
    Foo &operator=(const Foo&) &;  // 只能向可修改的左值赋值
};
Foo &Foo::operator=(const Foo &rhs) & 
{
    return *this;
}
```



> 引用限定符也可以区分重载版本，编译器会根据调用这个函数的对象的左值/右值属性来确定使用哪个版本。如果一个成员函数有引用限定符，则具有相同参数列表的所有版本都必须有引用限定符

<img src="E:\MarkDown\picture\image-20221004221336342.png" alt="image-20221004221336342" style="zoom:57%;" /><img src="E:\MarkDown\picture\image-20221004221057312.png" alt="image-20221004221057312" style="zoom:40%;" />

## 第十四章 重载运算与类型转换

操作符重载：

1、重载操作符（除了内置操作符）可以使用函数表示法调用

```cpp
std::string str = "Hello,";
str.operator+=("world");  // same as str += "wortd";
operator<<(operator<<(std::cout, str)，'\n'); // same as std::cout<< str << '\n';
```

2.一些限制

作用域解析运算符（::）、成员访问运算符（.）、通过指向成员的指针访问成员（*）不能被重载

不能创造一些新的运算符

&&、||、和逗号（,）失去了他们的特殊属性：短路求值（`&&` 与 `||` 操作符只有当 expr1的值不能确定整个表达式的值时，才会解第二个expr2的值，称为 **短路求值（short-circuit evaluation）**。）和排序。

操作符->的重载必须返回一个原始指针或返回一个对象(通过引用或通过值)，for which operator -> is in turn overloaded

不能更改操作符的优先级、grouping或操作数的数量。

## ==第十五章 面向对象程序设计OOP==

> c++面向对象的三大特性：封装、继承、多态
>
> 基于三个基本概念：数据抽象、继承和动态绑定
>
> 数据抽象：依赖于接口和实现分离的编程技术。
> 继承：可以定义相似的类型并对其相似关系建模
> 动态绑定：可以在一定程度上忽略相似类型的区别，而以统一的方式使用它们的对象



虚函数

> ​	虚函数是一种特殊的成员函数，它被声明为virtual关键字，用于**在子类中重新定义基类中的同名函数**。这样，在调用该函数时，程序会根据实际对象的类型**动态**地选择相应的函数实现。
>
> ​	如果该函数是**虚函数**，则直到**运行时**才会**决定**到底执行哪个版本，判断的依据是引用或指针所绑定的对象的真实类型。另一方面，对**非虚函数**的调用在**编译时**进行**绑定**。类似的，通过对象进行的函数(虚函数或非虚函数)调用也在编译时绑定。对象的类型是确定不变的，我们无论如何都不可能令对象的动态类型与静态类型不一致。因此，通过对象进行的函数调用将在编译时绑定到该对象所属类中的函数版本上。

​	虚函数，即基类希望派生类进行覆盖的函数。关键字virtual只能出现在 **类内部的声明语句之前** 而不能用于类外部的函数定义。
​	基类中的虚函数在派生类中隐含地也是一个虚函数
​	虚函数也可以有默认实参，且实参的值由本次调用的静态类型决定。换句话说，如果我们通过基类的引用或指针调用函数，则使用基类中定义的默认实参，即使实际运行的是派生类中的函数版本也是如此。此时，传入派生类函数的将是基类函数定义的默认实参。因此为了避免混淆，如果虚函数使用默认实参，则基类和派生类中定义的默认实参最好一致

```cpp
// 基类
class Q {
public:
    string isbn() const;
    // 虚函数
    virtual double net_price(size_t n) const;
    
    // 对析构函数进行动态绑定
    // 用于多态的基类通常都应该定义一个虚析构函数，即使该函数不执行任何实际操作
    virtual ~Q() = default;  
};


// 派生类
// 必须通过使用 类派生列表 明确指出是从哪个基类继承而来的
class B : public Q {
public:
    double net_price(size_t n) const override;
};

// 派生类的声明
// 注意，派生类的声明不包含派生列表
class B;

// 派生类构造函数
// 派生类对象中从基类继承而来的成员，需要用基类的构造函数初始化
// 派生类对象的基类部分与派生类对象自己的数据成员都是在构造函数的初始化阶段执行初始化操作的。类似于我们初始化成员的过程，派生类构造函数同样是通过构造函数初始化列表来将实参传递给基类构造函数的
// 除非我们特别指出，否则派生类对象的基类部分会像数据成员一样执行默认初始化。如果想使用其他的基类构造函数，我们需要以类名加圆括号内的实参列表的形式为构造函数提供初始值，如下例所示。这些实参将帮助编译器决定到底应该选用哪个构造函数来初始化派生类对象的基类部分
// 首先初始化基类的部分，然后按照声明的顺序依次初始化派生类的成员
Bulk_quote (const std::string& book, double P, std::size_ t qty, double disc) 
    : Quote (book, p)
    , min_qty(qty)
    , discount (disc)
{}


// 基类中的静态成员
// 对于基类中的静态成员，无论有多少个派生类，只存在唯一的实例。基类和派生类都能访问
// 静态成员遵循通用的访问控制规则
class Base {
public:
	static void statmem();
};
class Derived:public Base {
	void f(const Derived&);
};
void Derived::f (const Derived &derived_obj){
	Base::statmem(); //正确: Base定义了statmem
	Derived::statmem();//正确: Derived继承了statmem :
	//正确:派生类的对象能访问基类的静态成员
	derived_obj.statmem();//通过Derived对象访问
	statmem();//通过this对象(Derived对象)访问
}

// 间接基类
// 在这个继承关系中，Base是D1的直接基类(direct base)，同时是D2的间接基类(indirectbase)。直接基类出现在派生列表中，而间接基类由派生类通过其直接基类继承而来。
class Base { /* ... */ };
class D1: public Base { /* ... */ };
class D2: public D1 { /* ... */ };
```



### 动态绑定

​	当我们通过一个具有**普通类型(非引用非指针)**的表达式调用虚函数时，在**编译时**就会将调用的版本**确定**下来

```cpp
// base类型为基类Quote，derived类型为派生类
base = derived;  // 这里改变了base对象的值，但不能改变其类型。因此仅是把derived的Quote部分拷贝给base
base.net_price(20);//调用Quote::net_price
```

​	当我们使用**基类的引用或指针**调用一个**虚函数**时将发生**动态绑定**(运行时绑定) ：函数的运行版本由实参决定，即在运行时选择函数的版本

```cpp
// const Quote &item是基类的一个引用，因此我们既能用基类的对象调用，也可以使用派生类的对象调用(534页)
double print_total(ostream &os, const Quote &item, size_t n) {
    // net_price为 虚函数 ，是使用引用类型调用的，因此实际传入print_total的对象类型将决定到底执行net_price的哪个版本
	return item.net_price(n);
}
```

​	因此基类的指针或引用的静态类型可能与其动态类型不一致。**静态类型**是变量声明时的类型或表达式生成的类型，在编译时就是已知的；**动态类型**是变量或表达式表示的内存中的对象的类型，直到运行时才可知。

​	一个对象的动态类型依赖于其绑定的实参，传递一个派生类的对象，则其动态类型就会变为派生类的。如这里 print_total 传入的对象是个 bulk_quote 类型的，则 item 的静态类型是 Quote&，动态类型为 bulk_quote 。如果表达式既不是引用也不是指针，则它的动态类型永远与静态类型一致。例如，Quote 类型的变量永远是一个 Quote 对象，我们无论如何都不能改变该变量对应的对象的类型

**动态绑定中的作用域运算符**

​	通过作用域运算符，我们也可以强迫执行虚函数的某个特定版本，而不进行动态绑定。如例中强行调用基类 Quote 中定义的函数版本而不管 baseP 这个对象的动态类型到底是什么，该调用在编译时完成解析。

```cpp
double undiscounted = baseP->Quote::net_price(42);
```

​	通常是当一个派生类的虚函数调用它覆盖的基类的虚函数版本时，我们需要回避虚函数的默认机制。在此情况下，基类的版本通常完成继承层次中所有类型都要做的共同任务，而派生类中定义的版本需要执行一些与派生类本身密切相关的操作。而如果一个派生类虚函数需要调用它的基类版本，但是没有使用作用域运算符，则在运行时该调用将被解析为对派生类版本自身的调用，从而导致无限递归

```cpp
struct B {
public:
    virtual void f1();
}
struct D2:B{
	void f1 (){
        // 先调用父类的f1完成一些操作
        // 若没加作用域运算符，将在此函数中无限递归
        B::f1();
        // 在完成自己的一些操作
        ....
    }
};
```

### 派生类与基类的类型转换

> 只对指针或引用类型有效
>
> 和内置指针一样，智能指针类(参见12.1节，第400页)也支持派生类向基类的类型转换，这意味着我们可以将一个派生类对象的指针存储在一个基类的智能指针内

​	即基类指向派生类(的基类部分)，即将基类的指针或引用绑定到派生类对象中的基类部分上
​	派生类对象包含多个组成部分：一个含有派生类自己定义的(非静态)成员的子对象，以及一个与该派生类继承的基类对应的子对象，如果有多个基类，那么这样的子对象也有多个
​	因为在派生类对象中含有与其基类对应的组成部分，所以我们能把派生类的对象当成基类对象来使用，而且我们也能将基类的指针或引用绑定到派生类对象中的基类部分上

```cpp
Quote item;  // 基类对象
Bulk_quote bulk;  // 派生类对象
Quote *p = &item;  // 基类对象P指向基类对象
P = &bulk;  // p指向派生类对象的基类部分，可以将基类指针或引用绑定到派生类上
// bulk = &p;  // 错误，p为基类，不能将基类转换成派生类
Quote &r = bulk;  // r绑定到派生类对象的基类部分
```



​	派生类向基类的转换是否可访问由使用该转换的代码决定，同时派生类的派生访问说明符也会有影响。假定D继承自B:

* 只有当 `class D : public B` 时，用户代码才能使用派生类向基类的转换；如果D继承B的方式是受保护的或者私有的，则用户代码不能使用该转换。
* 不论D以什么方式继承B，**派生类的成员函数和友元都能使用派生类向基类的转换**；派生类向其直接基类的类型转换对于派生类的成员和友元来说永远是可访问的。
* 如果D继承B的方式是公有的或者受保护的，则D的派生类的成员和友元可以使用D向B的类型转换;反之，如果D继承B的方式是私有的，则不能使用。



**基类向派生类的转换不存在隐式类型转换**

​	正常来说，不能将基类转换成派生类，因为可能会使用派生类对象访问基类中不存在的成员（缺少了派生类的那部分成员）。即使一个基类指针或引用绑定在一个派生类对象上，我们也不能执行从基类向派生类的转换：

```cpp
Quote base;
Bulk_quote* bulkP = &base;  // 错误:不能将基类转换成派生类
Bulk_quote& bulkRef = base;  // 错误:不能将基类转换成派生类

Bulk_quote bulk;
Quote *itemP = &bulk;  // 正确：动态类型是Bulk_quote
Bulk_quote *bulkP = itemP;  // 错误：不能将基类转换成派生类
```

### dynamic_ cast

​	dynamic_cast，用于将基类的指针或引用安全地转换成派生类的指针或引用。如果在基类中有一个或多个虚函数，可以使用dynamic_cast请求一个类型转换，该转换的安全检查将在运行时执行

```cpp
// 730页19.2 运行时类型识别
// dynamic_cast<派生类*>(基类)
if (Derived *dp = dynamic_cast<Derived*>(bp)) {
	// 使用dp指向的Derived对象
} else { // bp指向一个Base对象
	// 使用bp指向的Base对象
}
```

​	同样，如果我们已知某个基类向派生类的转换是安全的，则我们可以使用 **static_cast** (参见4.11.3节，第144页)来强制覆盖掉编译器的检查工作。

​	尽管自动类型转换只对指针或引用类型有效，但是继承体系中的大多数类仍然(显式或隐式地)定义了拷贝控制成员(参见第13章)。因此，我们通常**能够将一个派生类对象拷贝、移动或赋值给一个基类对象**。不过需要注意的是，这种操作**只处理派生类对象的基类部分**
​	当我们用一个派生类对象为一个基类对象初始化或赋值时，只有该派生类对象中的基类部分会被拷贝、移动或赋值，它的派生类部分将被忽略掉，即将一个派生类对象赋值给基类对象后，这个基类对象的构造函数只处理了基类部分，派生类部分被切掉了

```cpp
Bulk_quote bulk;  // 派生类对象
Quote item(bulk);  // 使用Quote::Quote (const Quote&)构造函数
item = bulk;  // 调用Quote::operator= (const Quote&)
```



### 纯虚函数和抽象基类

> ​	含有（或者未经覆盖直接继承）纯虚函数的类是抽象基类，也就是不能被实体化的类。抽象基类负责定义接口，而后续的其他类可以覆盖该接口。我们不能（直接）创建一个抽象基类的对象，可以定义其派生类对象，**前提是这些类覆盖了纯虚函数**
>
> ​	**一个纯虚函数无须定义**。我们通过在函数体的位置(即在声明语句的分号之前)书写=0就可以将一个虚函数说明为纯虚函数。其中，=0只能出现在类内部的虚函数**声明**语句处
>
> （而例外就是纯虚析构，需要有具体实现。详见虚析构小节）

```cpp
// 用于保存折扣值和购买量的类，派生类使用这些数据可以实现不同的价格策略
class Disc_quote : public Quote {
public:
	Disc_quote() = default;
	Disc_quote (const std::string& book, double price, std::size t qty, double disc):Quote(book, price), quantity(qty), discount (disc) { }
    // 纯虚函数
	double net_price (std::size_t) const = 0;
protected:
	std::size_t quantity = 0;
	//折扣适用的购买量
	double discount = 0.0;
	//表示折扣的小数值
};
```

### 访问控制与继承

继承的语法：`class 子类 : 继承方式  父类`

> 继承方式(继承权限)一共有三种：
> 公共权限  public         类内可以访问  类外可以访问
> 保护权限  protected  类内可以访问  类外不可以访问  继承的派生类可以访问
> 私有权限  private       类内可以访问  类外不可以访问  继承的派生类不可以访问
>
> 构造与析构：继承中 先调用父类构造函数，再调用子类构造函数，析构顺序与构造相反

![image-20221007120703264](E:\MarkDown\picture\image-20221007120703264.png)

​	父类中所有非静态成员属性都会被子类继承，如父类中的私有成员只是被编译器隐藏了，但是还是会继承下去，因此从父类继承过来的成员，均属于子类对象

**默认的继承保护级别**

> ​	struct关键字和class关键字定义的类之间唯一的差别就是默认成员访问说明符及默认派生访问说明符。struct默认public继承，class默认private继承

```cpp
class Base{/*...*/};
struct D1:Base{/*...*/};  // struct关键字定义的派生类默认是public继承
class D2:Base{/*...*/};  // class默认private继承
```





### 友元与继承

​	友元关系不能继承。基类的友元在访问派生类成员时不具有特殊性，类似的，派生类的友元也不能随意访问基类的成员。例如A类是B类的友元，C类继承了B类，则A类能访问B类的成员，而A类与C类没有关系。

```cpp
class Base {
    friend class Pal;
protected:
	int prot_mem; 
    
};
class Sneaky : public Base {
	friend void clobber(Sneaky&);
	//能访问Sneaky: :prot mem
	friend void clobber(Base&);
	//不能访问Base: :prot_ mem
	int j;
	// j默认是private
};
class Pal {
public:
	int f (Base b) { return b.prot_mem; } // 正确: Pal是Base的友元
	int f2(Sneaky s) { return s.j; } // 错误: Pal不是Sneaky的友元
    // 注意，每个类负责控制自己的成员的访问权限，因此尽管看起来有点儿奇怪，但f3确实是正确的。Pal是Base的友元，所以Pal能够访问Base对象的成员，这种可访问性包括了Base对象内嵌在其派生类对象中的情况
    // 对基类的访问权限由基类本身控制，即使对于派生类的基类部分也是如此
	int f3(Sneaky s) { return s.prot_mem; } // 正确: Pal 是Base的友元
};
```

### 继承中的类作用域

> ​	当存在继承关系时，派生类的作用域嵌套在其基类的作用域之内。如果一个名字在派生类的作用域内无法正确解析，则编译器将继续在外层的基类作用域中寻找该名字的定义。
>
> ​	派生类的成员将隐藏同名的基类成员：和其他作用域一样，派生类也能重用定义在其直接基类或间接基类中的名字，此时定义在内层作用域(即派生类)的名字将隐藏定义在外层作用域(即基类)的名字。
>
> ​	一个对象、引用或指针的静态类型决定了该对象的哪些成员是可见的。即使静态类型与动态类型可能不一致(当使用基类的引用或指针时会发生这种情况)，但是我们能使用哪些成员仍然是**由静态类型决定的**。

> **调用p->mem()(或者obj .mem() )，则依次执行以下4个步骤:**
>
> ●首先确定p(或obj)的静态类型。因为我们调用的是一个成员，所以该类型必然是类类型。
> ●在p(或obj)的静态类型对应的类中查找mem。如果找不到，则依次在直接基类中不断查找直至到达继承链的顶端。如果找遍了该类及其基类仍然找不到，则编译器将报错。
> ●一旦找到了mem，就进行常规的类型检查(参见6.1节，第183页)以确认对于当前找到的mem,本次调用是否合法。
> ●假设调用合法，则编译器将根据调用的是否是虚函数而产生不同的代码：
> ——如果mem是虚函数且我们是通过引用或指针进行的调用，则编译器产生的代码将在运行时确定到底运行该虚函数的哪个版本，依据是对象的动态类型。
> ——反之，如果mem不是虚函数或者我们是通过对象(而非引用或指针)进行的调用，则编译器将产生一个常规函数调用。

> 名字查找先于类型检查，因此如果派生类(即内层作用域)的成员与基类(即外层作用域)的某个成员同名，则派生类将在其作用域内隐藏该基类成员。即使派生类成员和基类成员的形参列表不一致，基类成员也仍然会被隐藏掉。注意，这里派生类的同名不同参函数**也会覆盖基类的虚函数**，就无法通过基类的引用或指针调用派生类的虚函数了，所以有些情况下看似是对虚函数的动态绑定，但实际上因为函数的覆盖，其实为静态绑定
>
> 可以通过使用using声明语句，指定一个名字而不指定形参列表，如using Base::size;来将基类中该函数的所有重载实例添加到派生类作用域中。此时，派生类只需要定义其特有的函数就可以了，而无须为继承而来的其他函数重新定义。

### 虚析构

> **对于设计目的是为了多态用途的base class，才需要虚析构函数**
>
> ​	如前所述，当我们delete一个动态分配的对象的指针时将执行析构函数。如果该指针指向继承体系中的某个类型，则有可能出现指针的静态类型与被删除对象的动态类型不符的情况。例如**一个指向派生类对象的基类指针**，如果我们delete一个Quote*类型的指针，则该指针有可能实际指向了一个Bulk_quote类型的对象。**当derived class对象经由一个base class指针被删除，而该base class带着一个non-virtual析构函数，其结果未有定义**——实际执行时通常发生的是对象的derived成分没被销毁。因此编译器就必须清楚它应该执行的是Bulk_quote 的析构函数。和其他函数一样，我们通过在基类中将析构函数定义成虚函数以确保执行正确的析构函数版本。
> ​	而对于任何不带virtual析构函数的class，包括所有STL容器如vector, list, set, tr1: :unordered_ map (见条款54)等等。禁止继承一个标准容器或任何其他“带有non-virtual析构函数”的class
>
> ​	如果基类的析构函数不是虚函数，则delete一个指向派生类对象的基类指针将产生未定义的行为。
>
> ​	析构函数的虚属性也会被继承，因此无论派生类使用合成的析构函数还是定义自己的析构函数，都将是虚析构函数。只要基类的析构函数是虚函数，就能确保当我们delete基类指针时将运行正确的析构函数版本
>
> ```cpp
> Quote *itemP = new Quote;  // 静态类型与动态类型一致
> delete itemP;  // 调用Quote的析构函数
> itemP = new Bulk_quote;  // 静态类型与动态类型不一致
> delete itemP;  // 调用Bulk quote的析构函数
> ```
>
> 注意：如果一个类定义了析构函数，即使它通过=default的形式使用了合成的移动版本，**编译器也不会为这个类合成移动操作**（见笔记十三.4）。大多数基类都会定义一个虚析构函数。因此在默认情况下，基类通常不含有合成的移动操作，而且在它的派生类中也没有合成的移动操作。因此当我们确实需要执行移动操作时，应该首先在基类中进行定义

```cpp
// 基类
class Q {
public:
    string isbn() const;
    virtual double net_price(size_t n) const;
    // 基类(作为继承关系中根节点的类)通常都应该定义一个虚析构函数，即使该函数不执行任何实际操作
    virtual ~Q() = default;  
};
```

构造由内而外，析构由外而内



> **只有当clas内含至少一个virtual函数，才为它声明virtual析构函数**
>
> 若一个class不企图被当作base class，令其析构函数为virtual往往是愚蠢的
>
> 欲实现出virtual 函数，对象必须携带某些信息，主要用来在运行期决定哪一个virtual函数该被调用。这份信息通常是由一个所谓vptr (virtual table pointer)指针指出。vptr指向一个由函数指针构成的数组，称为vtbl(virtual table)；每一个带有virtual函数的class都有一个相应的vtbl。当对象调用某一virtual函数，实际被调用的函数取决于该对象的vptr所指的那个vtbl——编译器在其中寻找适当的函数指针。
>
> 而这就会导致对象的体积增加，在32-bit计算机体系结构中将额外占用32bit的vptr，在64-bit计算机体系结构中将额外占用64bit的vptr
>
> > 例如一个Point类中有两个int成员变量。如果int占用32 bits,那么Point对象可塞入一个64-bit 缓存器中。更有甚者，这样一个Point对象可被当做一个“64-bit 量”传给以其他语言如C或FORTRAN撰写的函数。然而当Point的析构函数是virtual，为Point添加一个vptr会增加其
> > 对象大小达50%~100%，使得Point对象不再能够塞入一个64-bit缓存器，而C++的Point
> > 对象也不再和其他语言(如C)内的相同声明有着一样的结构(因为其他语言的对应物并没有vptr) ，因此也就不再可能把它传递至(或接受自)其他语言所写的函数，除非你明确补偿vptr——那属 于实现细节，也因此不再具有移植性。



> 纯虚析构
>
> 当希望一个类是抽象基类，类中又没有纯虚函数，可以为其声明一个纯虚析构来使其成为抽象类
>
> 注意，虽然纯虚函数无需定义，但对于纯虚析构函数，必须为其提供一份定义。因为纯虚析构函数最终需要被调用，以析构基类对象，虽然是抽象类没有实体，但若不提供该析构函数的实现，将使得在析构过程中，析构无法完成而导致析构异常的问题
>
> ```cpp
> class AWOV {  // AWOV = "Abstract w/o Virtuals"
> public:
> 	virtual ~AWOV() = 0;  // 声明pure virtual析构函数
> };
> AWOV::~AWOV() {}  // pure virtual析构函数的定义
> ```
>
> 

### 派生类中删除的拷贝控制与基类的关系
​	就像其他任何类的情况一样，基类或派生类也能出于同样的原因将其合成的默认构造函数或者任何一个拷贝控制成员定义成被删除的函数。此外，某些定义基类的方式也可能导致有的派生类成员成为被删除的函数:

​	●如果基类中的默认构造函数、拷贝构造函数、拷贝赋值运算符或析构函数是被删除的函数或者不可访问(参见15.5节，第543页)，则派生类中对应的成员将是被删除的，原因是编译器不能使用基类成员来执行派生类对象基类部分的构造、赋值或销毁操作。
​	●如果在基类中有一个不可访问或删除掉的析构函数,则派生类中合成的默认和拷贝构造函数将是被删除的，因为编译器无法销毁派生类对象的基类部分。
​	●和过去一样，编译器将不会合成一个删除掉的移动操作。当我们使用=default请求一个移动操作时，如果基类中的对应操作是删除的或不可访问的，那么派生类中该函数将是被删除的，原因是派生类对象的基类部分不可移动。同样，如果基类的析构函数是删除的或不可访问的，则派生类的移动构造函数也将是被删除的。

### 派生类的拷贝控制成员

> 当派生类定义了拷贝或移动操作时，该操作负责拷贝或移动包括基类部分成员在内的整个对象。

​	派生类构造函数在其初始化阶段中不但要初始化派生类自己的成员，还负责初始化派生类对象的基类部分（通过基类的构造函数实现）。因此，派生类的拷贝和移动构造函数在拷贝和移动自有成员的同时，也要拷贝和移动基类部分的成员。类似的，派生类赋值运算符也必须为其基类部分的成员赋值。和构造函数及赋值运算符不同的是，析构函数只负责销毁派生类自己分配的资源。如前所述，对象的成员是被隐式销毁的(参见13.1.3节，第445页);类似的，派生类对象的基类部分也是自动销毁的。

```cpp
// 无论基类的构造函数或赋值运算符是自定义的版本还是合成的版本，派生类的对应操作都能使用它们
class Base { /* ...*/ };
class D: public Base {
public:
// 派生类的拷贝和移动构造函数
    // 默认情况下，基类的默认构造函数初始化对象的基类部分
    // 要想使用拷贝或移动构造函数，或者说我们想拷贝或移动基类部分，我们必须在构造函数初始值列表中显式地调用该构造函数
	D(const D& d) : Base (d)  // 拷贝基类成员
	/* D的成员的初始值*/ { /* ... */ }
	D(D&& d) : Base (std: :move(d))  // 移动基类成员
	/* D的成员的初始值*/ { /* ... */ }

// 派生类的析构函数
    // 对象的基类部分也是隐式销毁的，因此派生类析构函数只负责销毁由派生类自己分配的资源
    ~D() {}
};

// 派生类的赋值运算符
// Base::operator=(const Base&)不会被自动调用
D &D::operator= (const D &rhs)
{
	Base::operator=(rhs); // 显式地调用基类赋值运算符，为基类部分赋值
	//按照过去的方式为派生类的成员赋值
	//酌情处理自賦值及释放已有资源等情况
	return *this;
}

```

> 注意，如果构造函数或析构函数调用了某个虚函数，则我们应该执行与构造函数或析构函数所属类型相对应的虚函数版本。这里可以参考effective c++条款九，不要在构造和析构期间使用虚函数

### 继承的有参构造函数(c++11)

在C++11新标准中，派生类能够重用其**直接基类**定义的有参构造函数。这里称之为继承。类不能继承默认、拷贝和移动构造函数，只能继承有参构造函数。如果派生类没有直接定义这些构造函数，则编译器将为派生类合成它们。

> 派生类继承基类构造函数的方式是提供一条注明了(直接)基类名的using声明语句：
> 	通常情况下，using声明语句只是令某个名字在当前作用域内可见。而当作用于构造函数时，using声明语句将令编译器产生代码。对于基类的每个构造函数，编译器都生成一个与之对应的派生类构造函数。换句话说，对于基类的每个构造函数，编译器都在派生类中生成一个形参列表完全相同的构造函数。
> 	* 构造函数的using声明不会改变该构造函数的访问级别，即基类的私有构造函数在派生类中仍为私有；且using声明语句不能指定explicit或constexpr，继承的构造函数是否有这些属性取决于继承的基类的构造函数
>  * 当一个基类构造函数含有默认实参，这些实参并不会被继承。相反，派生类将获得多个继承的构造函数，其中每个构造函数分别省略掉一个含有默认实参的形参。例如，如果基类有一个接受两个形参的构造函数，其中第二个形参含有默认实参,则派生类将获得两个构造函数：一个构造函数接受两个形参(没有默认实参)，另一个构造函数只接受一个形参，它对应于基类中最左侧的没有默认值的那个形参。
>  * 如果基类含有几个构造函数，则除了两个例外情况，大多数时候派生类会继承所有这些构造函数。第一个例外是**派生类可以继承一部分构造函数，而为其他构造函数定义自己的版本**。如果派生类定义的构造函数与基类的构造函数具有相同的参数列表，则该构造函数将不会被继承。定义在派生类中的构造函数将替换继承而来的构造函数。
>    第二个例外是**默认、拷贝和移动构造函数不会被继承**。这些构造函数按照正常规则被**合成**。继承的构造函数不会被作为用户定义的构造函数来使用，因此，如果一个类只含有继承的构造函数，则它也将拥有一个合成的默认构造函数。
>
> > 编译器生成的构造函数形如：derived(parms):base(args)
> > 等价于：Bulk_ quote (const std: :string& book, double price,std: :size_ t qty， double disc) :Disc_ quote (book, price, qty， disc) { }
> >
> > > 注意，这里将派生类构造函数的形参全部传递给了基类的构造函数。因此如果派生类含有自己的数据成员，则这些成员将被默认初始化

```cpp
class Bulk_ quote : public Disc_ quote {
public:
// 派生类的构造函数
    // 继承Disc_ quote 的有参构造函数
    // 
	using Disc_ quote: :Disc_ quote;
	double net_ price (std: :size_ t) const;
};
```

### 容器与继承

> ​	当我们使用容器存放继承体系中的对象时，通常必须采取间接存储的方式。因为不允许在容器中保存不同类型的元素，所以我们不能把具有继承关系的多种类型的对象直接存放在容器当中。（比如在vector< Quote >容器中放入Bulk_quote对象，则这个对象只有基类部分被放入容器中，派生类部分将被忽略掉）
>
> ​	因此，当我们希望在容器中存放具有继承关系的对象时，实际上存放的通常是基类的指针(更好的选择是智能指针。和往常一样，这些指针所指对象的动态类型可能是基类类型，也可能是派生类类型
>
> ```cpp
> vector<shared_ ptr<Quote>> basket;
> basket.push_back (make_shared<Quote> ("0-201-82470-1". 50));
> // 将派生类的智能指针转换成基类的智能指针
> basket.push_back(make_shared<Bulk_quote> ("0-201-54848-8"，50，10，.25));  // make_shared<Bulk_quote>返回一个shared_ ptr<Bulk_quote>对象，当我们调用push_back时该对象被转换成shared_ ptr<Quote>
> //调用Quote定义的版本;打印562.5，即在15*&50中扣除掉折扣金额
> cout << basket.back()->net_price(15) << endl;
> ```
>
> 





# Ⅳ 高级主题



## 第十七章 标准库特殊设施

### tuple类型





## 第十八章 用于大型程序的工具（没记笔记）

### 异常处理

- **throw:** 当问题出现时，程序会抛出一个异常。这是通过使用 **throw** 关键字来完成的。
- **catch:** 在您想要处理问题的地方，通过异常处理程序捕获异常。**catch** 关键字用于捕获异常。
- **try:** **try** 块中的代码标识将被激活的特定异常。它后面通常跟着一个或多个 catch 块。

​	异常处理(exception handling)机制为程序中**异常检测**和**异常处理**这两部分的协作提供支持。在C++语言中，异常处理包括:
●throw 表达式(throw expression)，异常检测部分使用throw表达式来表示它遇到了无法处理的问题。我们说throw引发(raise) 了异常。
●try语句块(try block)，异常处理部分使用try语句块处理异常。try语句块以关键字try开始，并以一个或多个catch子句(catch clause)结束。try语句块中代码抛出的异常通常会被某个catch子句处理。因为catch子句“处理”异常，所以它们也被称作异常处理代码(exception handler)。
●一套异常类(exception class)，用于在throw表达式和相关的catch子句之间传
递异常的具体信息。

throw表达式：

```cpp
// 首先检查两条数据是否是关于同一种书籍的
if (iteml.isbn() != item2.isbn())
	throw runtime_error("Data must refer to same ISBN");
// 如果程序执行到了这里，表示两个ISBN是相同的
cout << iteml + item2 << end1 ;
在这段代码中，如果ISBN不一样就抛出一个异常，该异常是类型runtime_error的对象。抛出异常将终止当前的函数，并把控制权转移给能处理该异常的代码。
```

try语句块：

```CPP
try {
	program-statements
} catch (exception-declaration) {
	handler-statements
} catch (exception-declaration) {
	handler-statements
} //

while (cin>>item1>>item2) {
	try {
		//执行添加两个Sales_ item 对象的代码
		//如果添加失败，代码抛出一个runtime_ error异常
	} catch (runtime_error err) {
		//提醒用户两个ISBN必须一致，询问是否重新输入
		cout << err.what () << "\nTry Again? Enter y or n" << endl;
		char C;
		cin >> C;
		if (!cin || C=='n' )
			break; // 跳出while循环
	}
}

```



异常类：

> 注意，定义在stdexcept 头文件中的类型必须使用 string 对象或 C 风格字符串来初始化他们。不允许使用默认初始化方式。
>
> 其他 3 个头文件中的 3 中类型则只能默认初始化，不能提供初始值。

●exception头文件定义了最通用的异常类exception. 它只报告异常的发生，不提供任何额外信息。
●**stdexcept**头文件定义了几种常用的异常类，详细信息在表5.1中列出。
●new头文件定义了bad_ alloc 异常类型，这种类型将在12.1.2节(第407页)详细介绍。
●type_ info头文件定义了bad_ cast异常类型，这种类型将在19.2节(第731页)详细介绍。

![image-20221017161302318](E:\MarkDown\picture\image-20221017161302318.png)

异常类型只定义了一个名为what的成员函数，该函数没有任何参数，返回值是一个指向C风格字符串(参见3.5.4节，第109页)的const char*。 该字符串的目的是提供关于异常的一些文本信息。what函数返回的C风格字符串的内容与异常对象的类型有关。如果异常类型有一个字符串初始值，则what返回该字符串。对于其他无初始值的异常类型来说，what返回的内容由编译器决定。

```cpp
// 通过继承和重载exception类来定义新的异常
// 这里throw ()也可以写为noexcept，表示这里的what函数将不会抛出异常
#include <iostream>
#include <exception>
using namespace std;
 
struct MyException : public exception
{
  const char * what () const throw ()
  {
    return "C++ Exception";
  }
};
 
int main()
{
  try
  {
    throw MyException();
  }
  catch(MyException& e)
  {
    std::cout << "MyException caught" << std::endl;
    std::cout << e.what() << std::endl;
  }
  catch(std::exception& e)
  {
    //其他的错误
  }
}

```







```cpp
// 实例
#include <iostream>
using namespace std;
 
double division(int a, int b)
{
   if( b == 0 )
   {
      throw "Division by zero condition!";
   }
   return (a/b);
}
 
int main ()
{
   int x = 50;
   int y = 0;
   double z = 0;
 
   try {
     z = division(x, y);
     cout << z << endl;
   }catch (const char* msg) {  // throw了一个const char*的异常，因此要捕获这样类型
     cerr << msg << endl;
   }
 
   return 0;
}

```



# c++中其他的库

## c++11 chrono时间库

 std::chrono是在C++11中引入的，是一个模板库，用来处理时间和日期的Time library。要使用chrono库,需要#include< chrono>

duration的部分源码

```cpp
// duration是chrono中用来记录时间长度的类别，本质为一个模板类
// 单位换算是1000
typedef duration<long long, nano> nanoseconds;  // ns
typedef duration<long long, micro> microseconds;  // us
typedef duration<long long, milli> milliseconds;  // ms
typedef duration<long long> seconds;
typedef duration<int, ratio<60> > minutes;
typedef duration<int, ratio<3600> > hours;
```

使用示例

```cpp
#include<iostream>
#include<thread>
#include<chrono>

using namespace std;

int main() {
	// duration时间段，用来记录时间长度的类别，本质为一个模板类下
	std::chrono::minutes t1( 10 );  // t1代表十分钟
	std::chrono::seconds t2( 60 );  // t2代表60秒
	std::chrono::seconds t3 = t1 - t2;
	std::cout << t3.count() << " second" << std::endl;
	// 强制的时间单位转换 duration_cast<>() 
	cout << chrono::duration_cast<chrono::minutes>( t3 ).count() << endl;
	
	
	
	// time_point时间点，是一个记录特定时间点的模板类
	// chrono中有两种clock:分别是：system_clock 和 steady_clock
	//  system_clock 是直接去抓系统的时间，有可能在使用中会被被修改
	// steady_clock 则是确实地去纪录时间的流逝，所以不会出现时间倒退的状况
	
	// 用法是通过clock类别提供的now()函数分别记录开始和结束的时间，然后做差
	std::chrono::steady_clock::time_point t4 = std::chrono::steady_clock::now();
	int i = 100;
	int add;
	while(i--) {
		add++;
		std::cout << "Hello World\n";
	}
	std::chrono::steady_clock::time_point t5 = std::chrono::steady_clock::now();
	std::cout << "Printing took " << std::chrono::duration_cast<std::chrono::nanoseconds>(t5 - t4).count() << "ns.\n";

	// time_point 的输出
	// STL 的 chrono 并没有定义 time_point 的输出方式，所以我们并不能直接透过 output stream 来输出 time_point 的资料
	// 如果想要输出的话，一个方法是透过 clock 提供的 to_time_t() 这个函式，把 time_point 先把他转换成 C-style 的 time_t，然后再透过 ctime() 这类的函式做输出
	// 输出：Fri Dec 09 10:29:15 2022
	std::chrono::system_clock::time_point now = std::chrono::system_clock::now();
	std::time_t now_c = std::chrono::system_clock::to_time_t( now );
	std::cout << std::ctime( &now_c ) << std::endl;

}
```







# 一些疑惑

1.这里D类中重载的()是常函数，对于类内的常函数，将this（常量指针）约束为指向常量的指针，因此常函数不能改变类中变量。但这里通过<<运算符将给定的值写入到os中？？

```cpp
#include<iostream>

class D {
public:
	D(std::ostream &s = std::cerr) : os(s) {}
	template <typename T> void operator()(T *p) const {
		os << "aa" << std::endl;
		delete p;
	}
private:
	std::ostream &os;
};

int main() {
	int *p = new int;
	D d;
	d(p);
}
```

解答：

​	注意，这里os是引用，```std::ostream &os;```。对于类内的引用变量，是不会改变这个引用变量，但引用变量绑定的值是可以改变的。将引用看作指针，只能保证这个指针变量保存的值不变，保证不了这个指针指向的值不可被修改。

​	可以将std::ostream &os;改为const std::ostream &os;，就可以看出区别了。

简单来说，成员函数的const只能针对成员变量，引用并不属于成员变量，对引用的赋值可以简单理解成*p=xxx.

**从每条语句对应的汇编代码，可以发现引用 是 保存 所引用对象的地址。当给引用赋值时，改变的不是引用本身，而是 地址指向的内存，即所引用的对象。**



2

![image-20221111104201226](E:\MarkDown\picture\image-20221111104201226.png)

比如向一个形参T&的函数传递一个常量，此时将绑定到这个常量上，保持了const属性，是底层的

引用的const是底层const，指不变的是引用绑定的值









# 面试题

米哈游
<img src="E:\MarkDown\picture\image-20220926122501011.png" alt="image-20220926122501011" style="zoom:50%;" />

```cpp
#include<iostream>
using namespace std;

class A
{
    public:
        int &x;
};

int main() {
	// 直接写这个，会报错， error:use of deleted function 'A::A()'
	// 因为这里没有提供构造函数，编译器提供的默认构造函数是有问题的，无法初始化这个变量
	// A a;
	
	// 可以使用初始化列表
	// 由于A是聚合类型，因此从初始化列表中的对应项执行拷贝初始化（先生成临时对象再拷贝
	int i = 3;
	A a = {i};
}
```









## 大疆

偏特化相关
函数模板不能被偏特化



auto &&var = func();
var的类型一定是引用？可能不是引用？右值？左值？、



extern c 的作用




<img src="E:\MarkDown\picture\image-20220829184859805.png" alt="image-20220829184859805" style="zoom:67%;" />

```cpp
#include<iostream>
using namespace std;

size_t func(char a[]) {
	return sizeof(a);
}
int main() {
	char a[16];
	cout << sizeof(a) + func(a);
}
```

在32位系统下输出20，在64位系统下输出24






<img src="E:\MarkDown\picture\image-20220829185109616.png" alt="image-20220829185109616" style="zoom:67%;" />




<img src="E:\MarkDown\picture\image-20220829185141835.png" alt="image-20220829185141835" style="zoom:67%;" />




<img src="E:\MarkDown\picture\image-20220829185156167.png" alt="image-20220829185156167" style="zoom:67%;" />






<img src="E:\MarkDown\picture\image-20220829185244479.png" alt="image-20220829185244479" style="zoom:40%;" />
<img src="E:\MarkDown\picture\image-20220829185310732.png" alt="image-20220829185310732" style="zoom:40%;" />





![image-20220829185344889](E:\MarkDown\picture\image-20220829185344889.png)<img src="E:\MarkDown\picture\image-20220829185354115.png" alt="image-20220829185354115" style="zoom:67%;" />
<img src="E:\MarkDown\picture\image-20220829185426612.png" alt="image-20220829185426612" style="zoom:67%;" /><img src="E:\MarkDown\picture\image-20220829185440863.png" alt="image-20220829185440863" style="zoom:67%;" />











# 较好的cpp文章



[1.良好的编程习惯与编程要点](https://mp.weixin.qq.com/s/qtgnScMQbalO1-U2vNnkBA)

[2.Copy/move elision: C++ 17 vs C++ 11](https://mp.weixin.qq.com/s?__biz=MzI4ODE4NTkxMw==&mid=2649441370&idx=1&sn=b8f4d13501cb1a4bf376c72779f716fb&chksm=f3ddaf0cc4aa261a76a50b15745573e148962b17444d99ff1a7162534aefcf0ce67c547ac4fd&token=2061556422&lang=zh_CN#rd)

[3.对 int 变量赋值的操作是原子的吗？为什么？(没看懂)](https://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651171979&idx=1&sn=5fb16906bcf2bbbdf99898c705fcf8c8&chksm=80647dd4b713f4c24738dc2729f117ede2b8b81a7170e2afe0c168ef1030aff894495f13c084&mpshare=1&scene=23&srcid=0726AyQN24EzbCBKWgDLXyAz&sharer_sharetime=1658826217723&sharer_shareid=cfabe99fece2ab59be8d039f11a2fdff#rd)

[4.图解函数调用过程(汇编、函数栈)](https://mp.weixin.qq.com/s/VX13nGppBblm3zNFkiEfLQ)

[5.C++临时变量的生命周期，纯右值实体化(涉及概念太多，没看懂)](https://mp.weixin.qq.com/s?__biz=MzI4ODE4NTkxMw==&mid=2649441386&idx=1&sn=c1fda1d9f053c96b08ac0aa7a889636e&chksm=f3ddaf3cc4aa262af62dc89733a0b950951aeaa604db851b6fb3e44bb84c41b1f86b3c7fd0eb&cur_album_id=1950101107037241345&scene=189#wechat_redirect)



























&取地址运算符

*解引用运算符

::a;//使用**作用域运算符**来覆盖默认的作用域规则。全局作用域本身没有名字，因此::左侧为空时，向全局作用域发出请求获取作用域操作符右侧名字对应的变量



问题1：

引用传递时正常计算sizeof()，输出len为5，sizeof(array)为5，内容为{'2','9','3','7','5'}，正常

**不建议这么写，只有在函数模板里才会不报错。**

```c++
char arr[5] = {'2','9','3','7','5'};//传入数组

//将地址运算符&用于数组名时，将返回整个数组的地址。
void mySwap(T& array,int select) {
	int len = sizeof(array)/sizeof(array[0]);}
```





指针传递时，提示用另一个值除指针的sizeof值，且len的输出变为4

sizeof(array)为4，sizeof(array[0])为1，这里array为一个指针，array[0]为'2'，数据类型为char，占一个字节

```c++
//这写法不建议，写在普通函数中就报错了。
void mySwap(T array,int select) {
	int len = sizeof(array)/sizeof(array[0]);}

//地址传递，T array[]和T* array本质相同，array都为指针
void mySwap(T array[],int select) {
	int len = sizeof(array)/sizeof(array[0]);}

void mySwap(T* array,int select) {
	int len = sizeof(array)/sizeof(array[0]);}

//当数组名传入到函数作为参数时，被退化为指向首元素的指针
```







问题2：

在函数模板==引用传参==，用显式类型转换时报错  “没有与参数列表匹配的函数模板”，而参数列表是(int,char)

原因：

类型转换时是产生的临时变量

结论：

少用显式类型转换，尤其是在形参为引用的情况下



函数形参为引用时，无法进行显示类型转换`

![image-20211005103232861](E:\MarkDown\picture\image-20211005103232861.png)

函数形参为值传递时候，则可以

![image-20211005103342292](E:\MarkDown\picture\image-20211005103342292.png)

为什么呢？因为==类型转换是产生临时量==，这个临时量在第二个例子中值传递进入，然后直接返回，所以不报错。在第一个例子中，a 作为 char 传入给 int& 时需要进行类型转换，产生 int 型的临时量，这种临时量无法被一般的左值引用所绑定。但可以被 const 左值引用和 右值引用绑定，测试如下：

![image-20211005103748893](E:\MarkDown\picture\image-20211005103748893.png)

![img](E:\MarkDown\picture\3T[L`T[R(KSHT``H)Q@@MX2.png)

在函数模板中，也同样可以用const等来实现类型转换的传入，但是，因为类型转换毕竟是临时变量，因此还是无法实现函数内交换等操作，只能进行打印输出之类

![img](E:\MarkDown\picture\{EWDAI]QQJA@RQIV06N]5F.png)



![image-20211005104104805](E:\MarkDown\picture\image-20211005104104805.png)



关于右值引用：

![在这里插入图片描述](E:\MarkDown\picture\cf7a8a2e13ca402ea6cf246b175878c8.png)















