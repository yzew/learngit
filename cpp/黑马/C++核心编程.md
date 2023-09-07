# C++核心编程

本阶段主要针对C++**面向对象**编程技术做详细讲解，探讨C++中的核心和精髓。



## 1 内存分区模型

C++程序在执行时，将内存大方向划分为**4个区域**

- 代码区：存放函数体的二进制代码，由操作系统进行管理的
- 全局区：存放**全局变量**和**静态变量**以及**常量**
- 栈区：由编译器自动分配释放, 存放**函数的参数值**，**局部变量**，**局部常量**等
- 堆区：由程序员分配和释放,若程序员不释放,程序结束时由操作系统回收

**内存四区意义：**

不同区域存放的数据，赋予不同的生命周期, 给我们更大的灵活编程



### 1.1 程序运行前

​	在程序编译后，生成了exe可执行程序，**未执行该程序前**分为两个区域

​	**代码区：**

​		存放 CPU 执行的机器指令

​		代码区是**共享**的，共享的目的是对于频繁被执行的程序，只需要在内存中有一份代码即可

​		代码区是**只读**的，使其只读的原因是防止程序意外地修改了它的指令

​	**全局区：**

​		全局变量和静态变量存放在此.

​		全局区还包含了常量区, 字符串常量和其他常量也存放在此.

​		**该区域的数据在程序结束后由操作系统释放**.



**示例：**

```c++
//全局变量
int g_a = 10;
int g_b = 10;

//全局常量
const int c_g_a = 10;
const int c_g_b = 10;

int main() {
    
    //全局区
    //全局变量、静态变量、常量

	//局部变量
	int a = 10;
	int b = 10;

	//打印地址
	cout << "局部变量a地址为： " << (int)&a << endl;
	cout << "局部变量b地址为： " << (int)&b << endl;

	cout << "全局变量g_a地址为： " <<  (int)&g_a << endl;
	cout << "全局变量g_b地址为： " <<  (int)&g_b << endl;

	//静态变量
	static int s_a = 10;
	static int s_b = 10;

	cout << "静态变量s_a地址为： " << (int)&s_a << endl;
	cout << "静态变量s_b地址为： " << (int)&s_b << endl;

	cout << "字符串常量地址为： " << (int)&"hello world" << endl;
	cout << "字符串常量地址为： " << (int)&"hello world1" << endl;

	cout << "全局常量c_g_a地址为： " << (int)&c_g_a << endl;
	cout << "全局常量c_g_b地址为： " << (int)&c_g_b << endl;

	const int c_l_a = 10;
	const int c_l_b = 10;
	cout << "局部常量c_l_a地址为： " << (int)&c_l_a << endl;
	cout << "局部常量c_l_b地址为： " << (int)&c_l_b << endl;

	system("pause");

	return 0;
}
```

打印结果：

![1545017602518](assets/1545017602518.png)

总结：

* C++中在程序运行前分为全局区和代码区
* 代码区特点是共享和只读
* 全局区中存放全局变量、静态变量、常量
* 常量区中存放 const修饰的全局常量  和 字符串常量




### 1.2 程序运行后

​	**栈区：**

​		由编译器自动分配释放，存放函数的参数值，局部变量，局部常量等

​		注意事项：**不要返回局部变量的地址**，栈区开辟的数据由编译器自动释放

**示例：**

```c++
#include<iostream>
using namespace std;
int * func()
{
	int a = 10;//局部变量
	return &a;
}

int main() {

	int *p = func();//接收func函数的返回值

	cout << *p << endl;//输出10，可以打印局部变量
    //第一次可以打印正确的数字，是因为编译器做了保留
	cout << *p << endl;//输出2058590600，乱码，因为执行完后栈上的数据清空了

	system("pause");

	return 0;
}
```



​	**堆区：**	

​	由程序员分配释放,若程序员不释放,程序结束时由操作系统回收

​	在C++中主要利用new在堆区开辟内存

**示例：**

```c++
//在堆区开辟数据

int* func()
{
	//指针本质也是局部变量，放在栈区，指针保存的数据是放在堆区
    //new返回的是该数据类型的指针
    int* a = new int(10);//用new开辟一块初始值为10的内存，将堆区的地址传递给指针
	return a;
}

int main() {

	int *p = func();

	cout << *p << endl;//输出10
	cout << *p << endl;
    
	system("pause");

	return 0;
}
```

**总结：**

堆区数据由程序员管理开辟和释放

堆区数据利用new关键字进行开辟内存



### 1.3 new操作符



​	C++中利用**new**操作符在堆区开辟数据

​	堆区开辟的数据，由程序员手动开辟，手动释放，释放利用操作符**delete**

​	语法：` new 数据类型`

​	利用new创建的数据，会返回该数据对应的类型的指针



**示例1： 基本语法**

```c++
int* func()
{
	//指针本质也是局部变量，放在栈区，指针保存的数据是放在堆区
	//new返回的是该数据类型的指针
	int* a = new int(10);//用new开辟一块初始值为10的内存，将堆区的地址传递给指针
	return a;
}

int main() {

	int *p = func();

	cout << *p << endl;
	cout << *p << endl;

	//利用delete释放堆区数据
	delete p;

	//cout << *p << endl; //报错，释放的空间不可访问

	system("pause");

	return 0;
}
```



**示例2：开辟数组**

```c++
//堆区开辟数组
int main() {
    // 返回该数据对应的类型的指针
	int* arr = new int[10];//和int* a = new int(10);区别，是[]，表示数组
	for (int i = 0; i < 10; i++)
	{
		arr[i] = i + 100;
	}
	for (int i = 0; i < 10; i++)
	{
		cout << arr[i] << endl;
	}
	//释放数组 delete 后加 []
	delete[] arr;

	system("pause");
	return 0;
}
```



## 2 引用

### 2.1 引用的基本使用

**作用： **给变量起别名

**语法：** `数据类型 &别名 = 原名`，其中数据类型需要和原名的一致

起别名后，如a和b，相当于两个变量都指向一个内存，任何一个变量的赋值都会对内存的内容进行改变，则两个变量同时变化



**示例：**

```C++
int main() {

	int a = 10;
	int &b = a;

	cout << "a = " << a << endl;
	cout << "b = " << b << endl;

	b = 100;

	cout << "a = " << a << endl;
	cout << "b = " << b << endl;

	system("pause");

	return 0;
}
```



### 2.2 引用注意事项

* 引用必须初始化，即必须int& b = a; 不能先int& b;，后进行赋值
* 引用在初始化后，不可以改变，即

示例：

```C++
int main() {

	int a = 10;
	int b = 20;
	//int &c; //错误，引用必须初始化
	int &c = a; //一旦初始化后，就不可以更改，即c指向a后，不能更改成指向别的
	c = b; //这是赋值操作，不是更改引用

	cout << "a = " << a << endl;
	cout << "b = " << b << endl;
	cout << "c = " << c << endl;

	system("pause");

	return 0;
}
```



### 2.3 引用做函数参数

**作用：**函数传参时，可以利用引用的技术让形参修饰实参

**优点：**可以简化指针修改实参

原理：还是通过起别名，如这里是相当于int& a=a;因此第一个a变化就可以影响到第二个a了

**示例：**

```C++
//1. 值传递，形参不会修饰实参，主函数实参并无交换
void mySwap01(int a, int b) {
	int temp = a;
	a = b;
	b = temp;
}

//2. 地址传递
void mySwap02(int* a, int* b) {
	int temp = *a;
	*a = *b;
	*b = temp;
}

//3. 引用传递，形参也会修饰实参
void mySwap03(int& a, int& b) {
	int temp = a;
	a = b;
	b = temp;
}

int main() {

	int a = 10;
	int b = 20;

	mySwap01(a, b);
	cout << "a:" << a << " b:" << b << endl;

	mySwap02(&a, &b);
	cout << "a:" << a << " b:" << b << endl;

	mySwap03(a, b);
	cout << "a:" << a << " b:" << b << endl;

	system("pause");

	return 0;
}

```

> 总结：通过引用参数产生的效果同按地址传递是一样的。引用的语法更清楚简单



### 2.4 引用做函数返回值

作用：引用是可以作为函数的返回值存在的

注意：**不要返回局部变量引用**

用法：函数调用作为**左值**



**示例：**

```C++
//1、不要返回局部变量引用

//引用做函数的返回值
int& test01() {//在返回值类型int后加一个&，就相当于是以引用的方式返回
	int a = 10; //局部变量，存放在栈区，
	return a;
}

//2、函数调用作为左值 test02() = 1000;
//返回静态变量引用，放在全局区，在程序结束后系统才释放
int& test02() {
	static int a = 20;
	return a;
}

int main() {

	//不能返回局部变量的引用，因为引用指向的地址在栈区，会被自动释放
	int& ref = test01();//int& 接收的是别名，即ref指向a
    //若是改为int ref = test01();则是用ref接收a的值？反正没有被释放
    
	cout << "ref = " << ref << endl;//输出10
	cout << "ref = " << ref << endl;//输出乱码，局部变量被释放

	//如果函数做左值，那么必须返回引用
	int& ref2 = test02();
	cout << "ref2 = " << ref2 << endl;//20
	cout << "ref2 = " << ref2 << endl;//20

	test02() = 1000;//如果函数的返回值是引用，函数的调用可以作为左值
    //test02()相当于把a这个变量进行了返回，因此相当于是给a赋值
    //而这里ref2为a的别名，因此随之改变

	cout << "ref2 = " << ref2 << endl;//1000
	cout << "ref2 = " << ref2 << endl;//1000

	system("pause");

	return 0;
}
```



### 2.5 引用的本质【重要】

本质：**引用的本质在c++内部实现是一个常量指针.**

常量指针，即int* const p，p不可以修改，即指针指向不可以修改，但指针指向的值可以修改

讲解示例：

```C++
//发现是引用，转换为 int* const ref = &a;
void func(int& ref){
	ref = 100; // ref是引用，转换为*ref = 100
}
int main(){
	int a = 10;
    
    //自动转换为 int* const ref = &a; 常量指针是指针指向不可改，也说明为什么引用不可更改
	int& ref = a; 
	ref = 20; //内部发现ref是引用，自动帮我们转换为: *ref = 20;
    
	cout << "a:" << a << endl;
	cout << "ref:" << ref << endl;
    
	func(a);
	return 0;
}
```

结论：C++推荐用引用技术，因为语法方便，引用本质是常量指针，但是所有的指针操作编译器都帮我们做了



如果返回值是引用，返回值就是本身；如果返回值是一个值，实际上返回的是一个值的副本

![image-20210921222626523](D:\MarkDown\picture\image-20210921222626523.png)



### 2.6 常量引用

**作用：**常量引用主要用来修饰形参，防止误操作

在函数形参列表中，可以加**const修饰形参**，防止形参改变实参



**示例：**

```C++
//引用使用的场景，通常用来修饰形参
void showValue(const int& v) {
	//v += 10;，const后就不能对v进行修改了，相当于是对v的一个保护，防止在函数中对参数误操作
	cout << v << endl;
}

int main() {

	//int& ref = 10;  引用本身需要一个合法的内存空间，因此这行错误。引用本质是给变量取别名，这种操作是非法的。
	//加入const就可以了，编译器优化代码，int temp = 10; const int& ref = temp;相当于我们操作的ref为一个别名
	const int& ref = 10;

	//ref = 100;  //加入const后不可以修改变量
	cout << ref << endl;

	//函数中利用常量引用防止误操作修改实参
	int a = 10;
	showValue(a);

	system("pause");

	return 0;
}
```



## 3 函数提高

### 3.1 函数默认参数

在C++中，函数的形参列表中的形参是可以有默认值的。

语法：` 返回值类型  函数名 （参数= 默认值）{}`

**示例：**

```C++
int func(int a, int b = 10, int c = 10) {
	return a + b + c;
}

//1. 如果某个位置参数有默认值，那么从这个位置往后，从左向右，必须都要有默认值
int func3(int a,int b=10,int c){}//报错，不可以
//2. 如果函数声明有默认值，函数实现的时候就不能有默认参数，因为如果声明和实现的默认值不一样，会产生二义性
//声明和实现只能有一个有默认参数
int func2(int a = 10, int b = 10);//声明
int func2(int a, int b) {
	return a + b;
}

int main() {

	cout << "ret = " << func(20, 20) << endl;
	cout << "ret = " << func(100) << endl;

	system("pause");

	return 0;
}
```



### 3.2 函数占位参数

C++中函数的形参列表里可以有占位参数，用来做占位，调用函数时必须填补该位置

**语法：** `返回值类型 函数名 (数据类型){}`

在现阶段函数的占位参数存在意义不大，但是后面的课程中会用到该技术

**示例：**

```C++
//函数占位参数 
void func(int a, int) {
	cout << "this is func" << endl;
}
//占位参数也可以有默认参数
void func2(int a, int=10) {
	cout << "this is func" << endl;
}
int main() {
	func(10,10); //占位参数必须填补
    func2(10);//有默认参数可以不填补

	system("pause");
	return 0;
}
```



### 3.3 函数重载

#### 3.3.1 函数重载概述

**作用：**函数名可以相同，提高复用性

**函数重载满足条件：**

* 同一个作用域下
* 函数名称相同
* 函数参数**类型不同**  或者 **个数不同** 或者 **顺序不同**

**注意:**  函数的返回值不可以作为函数重载的条件



**示例：**

```C++
//函数重载需要函数都在同一个作用域下，如这里都在全局作用域下
void func()
{
	cout << "func 的调用！" << endl;
}
void func(int a)
{
	cout << "func (int a) 的调用！" << endl;
}
void func(double a)
{
	cout << "func (double a)的调用！" << endl;
}
void func(int a ,double b)
{
	cout << "func (int a ,double b) 的调用！" << endl;
}
void func(double a ,int b)
{
	cout << "func (double a ,int b)的调用！" << endl;
}

//函数返回值不可以作为函数重载条件
//int func(double a, int b)
//{
//	cout << "func (double a ,int b)的调用！" << endl;
//}


int main() {

	func();
	func(10);
	func(3.14);
	func(10,3.14);
	func(3.14 , 10);
	
	system("pause");

	return 0;
}
```



#### 3.3.2 函数重载注意事项

* 引用作为重载条件，实参为变量则调用形参为int& a的函数，实参为数值则调用const形参(自动优化)
* 函数重载碰到函数默认参数会报错



**示例：**

```C++
//函数重载注意事项
//1、引用作为重载条件

void func(int &a)
{
	cout << "func (int &a) 调用 " << endl;
}

void func(const int &a)
{
	cout << "func (const int &a) 调用 " << endl;
}


//2、函数重载碰到函数默认参数。会报错，因为产生歧义

void func2(int a, int b = 10)
{
	cout << "func2(int a, int b = 10) 调用" << endl;
}

void func2(int a)
{
	cout << "func2(int a) 调用" << endl;
}

int main() {
	
	int a = 10;
	func(a); //调用无const
	func(10);//调用有const，因为对于void func(int &a)相当于int &a = 10；不合法
    //对于void func(const int &a)，相当于const int &a=10，这时如前面所讲的，编译器自动进行优化，创建temp，所以合法


	//func2(10); //碰到默认参数产生歧义，需要避免

	system("pause");

	return 0;
}
```



## **4** 类和对象

C++认为**万事万物都皆为对象**，对象上有其属性和行为

​	具有相同性质的**对象**，我们可以抽象称为**类**，人属于人类，车属于车类







### 4.2 对象的初始化和清理

#### 4.2.1 构造函数和析构函数

对象的**初始化和清理**也是两个非常重要的安全问题

​	一个对象或者变量没有初始状态，对其使用后果是未知
​	同样的使用完一个对象或变量，没有及时清理，也会造成一定的安全问题



c++利用了**构造函数**和**析构函数**解决上述问题，这两个函数将会被编译器**自动调用**，完成对象初始化和清理工作。

对象的初始化和清理工作是编译器强制要我们做的事情，因此如果**我们不提供构造和析构，编译器会提供**

**编译器提供的构造函数和析构函数是空实现。**



* 构造函数：主要作用在于创建对象时为对象的成员属性赋值，构造函数由编译器自动调用，无须手动调用。
* 析构函数：主要作用在于对象**销毁前**系统自动调用，执行一些清理工作。



**构造函数语法：**`类名(){}`

1. 构造函数，没有返回值也不写void
2. 函数名称与类名相同
3. 构造函数可以有参数，因此可以发生重载
4. 程序在调用对象时候会自动调用构造，无须手动调用,而且只会调用一次



**析构函数语法：** `~类名(){}`

1. 析构函数，没有返回值也不写void
2. 函数名称与类名相同，在名称前加上符号  ~
3. 析构函数不可以有参数，因此不可以发生重载
4. 程序在**对象销毁前**会自动调用析构，无须手动调用，而且只会调用一次



```C++
class Person
{
public:
	//构造函数，自动调用
	Person()
	{
		cout << "Person的构造函数调用" << endl;
	}
	//析构函数
	~Person()
	{
		cout << "Person的析构函数调用" << endl;
	}

};

void test01()
{
	Person p;//在栈上的数据，test01执行完毕后，会释放这个对象，因此会自动调用析构函数
}

int main() {
	test01();
    Person p2;//在主函数创建对象的时候，在执行完后system("pause");程序结束，因此看不到析构函数的显示

	system("pause");
	return 0;
}
```



#### 4.2.2 构造函数的分类及调用

两种分类方式：

​	按参数分为： 有参构造和无参构造

​	按类型分为： 普通构造和拷贝构造

三种调用方式：

​	括号法

​	显示法

​	隐式转换法



**示例：**

```C++
//1、构造函数分类
// 按照参数分类分为 有参和无参构造   无参又称为默认构造函数
// 按照类型分类分为 普通构造和拷贝构造

class Person {
public:
    // 按照参数分类
	//无参构造（默认构造）函数
	Person() {
		cout << "无参构造函数!" << endl;
	}
	//有参构造函数
	Person(int a) {
		age = a;
		cout << "有参构造函数!" << endl;
	}
    
    
    // 按照类型分类
	//拷贝构造函数，因为是拷贝，不能把原来的给改变了，因此const；其次，拷贝时要用引用传进来
	Person(const Person& p) {
		age = p.age;//将传入对象的属性拷贝
		cout << "拷贝构造函数!" << endl;
	}
    
    
	//析构函数
	~Person() {
		cout << "析构函数!" << endl;
	}
    
public:
	int age;
};


//2、构造函数的调用
//调用无参构造函数
void test01() {
	Person p; //调用无参构造函数
}

//调用有参的构造函数
void test02() {

	//2.1  括号法，常用
    //Person p2;//无参构造函数
	Person p1(10);//有参构造函数
    //Person p2(p1)；y拷贝构造函数
    
	//注意1：调用无参构造函数不能加括号，如果加了编译器认为这是一个函数声明
	//Person p2();

	//2.2 显示法
	Person p2 = Person(10); 
	Person p3 = Person(p2);//拷贝构造函数
	//Person(10)单独写就是匿名对象  当前行结束之后，马上析构

	//2.3 隐式转换法
	Person p4 = 10; // Person p4 = Person(10); 
	Person p5 = p4; // Person p5 = Person(p4); 

	//注意2：不能利用 拷贝构造函数 初始化匿名对象 编译器认为是对象声明
	//Person (p4);
}

int main() {
	test01();
	//test02();

	system("pause");
	return 0;
}
```



#### 4.2.3 拷贝构造函数自动调用时机

C++中拷贝构造函数调用时机通常有三种情况

* 使用一个已经创建完毕的对象来初始化一个新对象
* 值传递的方式给函数参数传值
* 以值方式返回局部对象



**示例：**

```C++
class Person {
public:
	Person() {
		cout << "无参构造函数!" << endl;
		mAge = 0;
	}const   
	Person(int age) {
		cout << "有参构造函数!" << endl;
		mAge = age;
	}
	Person(const Person& p) {
		cout << "拷贝构造函数!" << endl;
		mAge = p.mAge;
	}
	//析构函数在释放内存之前调用
	~Person() {
		cout << "析构函数!" << endl;
	}
public:
	int mAge;
};

//1. 使用一个已经创建完毕的对象来初始化一个新对象
void test01() {

	Person man(100); //p对象已经创建完毕
	Person newman(man); //调用拷贝构造函数
	Person newman2 = man; //拷贝构造

	//Person newman3;
	//newman3 = man; //不是调用拷贝构造函数，赋值操作
}


//2. 值传递的方式给函数参数传值
//值传递相当于Person p1 = p;调用拷贝构造函数
void doWork(Person p1) {}//值传递
void test02() {
	Person p; //无参构造函数
	doWork(p);//拷贝构造函数
}

//3. 以值方式返回局部对象
Person doWork2()
{
	Person p1;//默认构造
	cout << (int *)&p1 << endl;
	return p1;//根据p1拷贝一个新的对象再返回，拷贝构造
}

void test03()//调用默认构造和拷贝构造
{
	Person p = doWork2();
	cout << (int *)&p << endl;//p和p1的返回地址不一样，不是同一个对象
}


int main() {
	//test01();
	//test02();
	test03();

	system("pause");
	return 0;
}
```

这里的第三种情况用g++编译会出现没调用拷贝构造函数，且两个地址相同的问题，是因为被编译器优化了，会直接用对象p存放dowork2()的返回值。称为copy ellision，即复制省略



#### 4.2.4 构造函数调用规则

默认情况下，c++编译器至少给一个类添加3个函数

**其中构造函数只提供无参构造和拷贝构造，有参构造需要自己写**

1．默认构造函数(无参，函数体为空)

2．默认析构函数(无参，函数体为空)

3．默认拷贝构造函数，对属性进行值拷贝

构造函数调用规则如下：

* 如果用户定义有参构造函数，c++不在提供默认无参构造，但是会提供默认拷贝构造，因此定义有参构造后需要用户再定义无参构造，否则调用无参构造时会出错


* 如果用户定义拷贝构造函数，c++不会再提供其他构造函数



示例：

```C++
class Person {
public:
	//无参（默认）构造函数
	Person() {
		cout << "无参构造函数!" << endl;
	}
	//有参构造函数
	Person(int a) {
		age = a;
		cout << "有参构造函数!" << endl;
	}
	//拷贝构造函数
	Person(const Person& p) {
		age = p.age;
		cout << "拷贝构造函数!" << endl;
	}
	//析构函数
	~Person() {
		cout << "析构函数!" << endl;
	}
public:
	int age;
};

void test01()
{
	Person p1(18);
	//如果不写拷贝构造，编译器会自动添加拷贝构造，并且做浅拷贝操作
	Person p2(p1);
	cout << "p2的年龄为： " << p2.age << endl;
}

void test02()
{
	//如果用户提供有参构造，编译器不会提供默认构造，会提供拷贝构造
	Person p1; //此时如果用户自己没有提供默认构造，会出错
	Person p2(10); //用户提供的有参
	Person p3(p2); //此时如果用户没有提供拷贝构造，编译器会提供

	//如果用户提供拷贝构造，编译器不会提供其他构造函数
	Person p4; //此时如果用户自己没有提供默认构造，会出错
	Person p5(10); //此时如果用户自己没有提供有参，会出错
	Person p6(p5); //用户自己提供拷贝构造
}

int main() {
	test01();
	system("pause");
	return 0;
}
```







**示例：**

```C++
class Person {
public:
	//无参（默认）构造函数
	Person() {
		cout << "无参构造函数!" << endl;
	}
	//有参构造函数
	Person(int age ,int height) {
		
		cout << "有参构造函数!" << endl;

		m_age = age;
		m_height = new int(height);//创建到堆区，用m_height这个指针接收
		
	}
    
	//自写拷贝构造函数，来解决浅拷贝带来的问题
	Person(const Person& p) {
		cout << "拷贝构造函数!" << endl;
		//如果不利用深拷贝在堆区创建新内存，会导致浅拷贝带来的重复释放堆区问题
		m_age = p.m_age;
        //m_height = p.m_height;//编译器的默认实现就是这行代码
        
        //深拷贝操作    *p.m_height为解引用，取出指针存的数值，再来开辟新空间
		m_height = new int(*p.m_height);//堆区开辟的内存，需要程序员手动释放
		
	}

	//析构函数，可以通过一些操作将堆区开辟数据做释放操作
	~Person() {
		cout << "析构函数!" << endl;
		if (m_height != NULL)
		{
			delete m_height;//将堆区开辟数据做释放操作
            m_height = NULL;//防止野指针的出现，做一个置空的操作
		}
	}
public:
	int m_age;
	int* m_height;//指针，将数据开辟到堆区
};

void test01()
{
	Person p1(18, 180);

	Person p2(p1);

	cout << "p1的年龄： " << p1.m_age << " 身高： " << *p1.m_height << endl;

	cout << "p2的年龄： " << p2.m_age << " 身高： " << *p2.m_height << endl;
}

int main() {

	test01();

	system("pause");

	return 0;
}
```

> 总结：如果属性有在堆区开辟的，一定要自己提供拷贝构造函数，防止浅拷贝带来的问题



#### 4.2.6 初始化列表

**作用：**

C++提供了初始化列表语法，用来初始化属性

**语法：**`构造函数(....)：属性1(值1),属性2（值2）... {}`

**示例：**

```C++
class Person {
public:

	////传统方式初始化
	//Person(int a, int b, int c) {
	//	m_A = a;
	//	m_B = b;
	//	m_C = c;
	//}

	//初始化列表方式初始化
	Person(int a, int b, int c) :m_A(a), m_B(b), m_C(c) {}
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


	system("pause");

	return 0;
}
```



#### 4.2.7 类对象作为类成员

C++类中的成员可以是另一个类的对象，我们称该成员为 对象成员

构造的顺序是 ：先调用对象成员的构造，再调用本类构造

析构顺序与构造相反



例如：

```C++
class A {}
class B
{
    A a;
}
```



B类中有对象A作为成员，A为对象成员

那么当创建B对象时，A与B的构造和析构的顺序是谁先谁后？



**示例：**

```C++
class Phone
{
public:
	Phone(string name)
	{
		m_PhoneName = name;
		cout << "Phone构造" << endl;
	}

	~Phone()
	{
		cout << "Phone析构" << endl;
	}

	string m_PhoneName;
};


class Person
{
public:

	//初始化列表可以告诉编译器调用哪一个构造函数
    //Phone m_Phone = pName，创建对象时的隐式转换法
	Person(string name, string pName) :m_Name(name), m_Phone(pName)
	{
		cout << "Person构造" << endl;
	}

	~Person()
	{
		cout << "Person析构" << endl;
	}

	void playGame()
	{
		cout << m_Name << " 使用" << m_Phone.m_PhoneName << " 牌手机! " << endl;
	}

	string m_Name;
	Phone m_Phone;

};
void test01()
{
	//当类中成员是其他类对象时，我们称该成员为 对象成员
	//构造的顺序是 ：先调用对象成员的构造，再调用本类构造
	//析构顺序与构造相反
	Person p("张三" , "苹果X");
	p.playGame();

}


int main() {

	test01();

	system("pause");

	return 0;
}
```



#### 4.2.8 静态成员

成员：出现在类中，类中的属性为成员属性，函数为成员函数

静态成员就是在成员变量和成员函数前加上关键字**static**，称为静态成员

静态成员分为：

*  静态成员变量
   *  所有对象共享同一份数据
   *  在编译阶段分配内存
   *  在类内声明，**在类外初始化**
*  静态成员函数
   *  所有对象共享同一个函数
   *  **静态成员函数只能访问静态成员变量**



**示例1 ：**静态成员变量

```C++
class Person
{
public:
	static int m_A; //静态成员变量，类内声明

	//静态成员变量特点：
	//1 在编译阶段分配内存
	//2 类内声明，类外初始化
	//3 所有对象共享同一份数据

private:
	static int m_B; //静态成员变量也是有访问权限的，也可以设为私有
};

int Person::m_A = 10;//类外初始化，初始化时不需要标示为static
int Person::m_B = 10;

void test01()
{
	//因为所有对象共享同一个函数
    //因此静态成员变量两种访问方式，分别是通过任意一个对象，和通过类名Person::

	//1、通过对象
	Person p1;
	p1.m_A = 100;
	cout << "p1.m_A = " << p1.m_A << endl;

	Person p2;
	p2.m_A = 200;
	cout << "p1.m_A = " << p1.m_A << endl; //共享同一份数据，200
	cout << "p2.m_A = " << p2.m_A << endl;

	//2、通过类名
	cout << "m_A = " << Person::m_A << endl;


	//cout << "m_B = " << Person::m_B << endl; //私有权限访问不到
}

int main() {

	test01();

	system("pause");

	return 0;
}
```



**示例2：**静态成员函数

```C++
class Person
{

public:

	//静态成员函数特点：
	//1 程序共享一个函数
	//2 静态成员函数只能访问静态成员变量
	
	static void func()
	{
		cout << "func调用" << endl;
		m_A = 100;//静态成员函数可以访问静态成员变量
		//m_B = 100; //错误，不可以访问非静态成员变量，因为静态成员变量是所有对象共享的，而你要是想在这里改变非静态成员变量，会不清楚是改变哪个对象的这个变量，因为每个对象都有一个对应的这个变量
	}

	static int m_A; //静态成员变量
	int m_B; // 
private:

	//静态成员函数也是有访问权限的
	static void func2()
	{
		cout << "func2调用" << endl;
	}
};

int Person::m_A = 10;


void test01()
{
	//静态成员变量两种访问方式

	//1、通过对象
	Person p1;
	p1.func();

	//2、通过类名
	Person::func();


	//Person::func2(); //私有权限访问不到
}

int main() {

	test01();

	system("pause");

	return 0;
}
```



[C++中const和static的作用](https://interviewguide.cn/#/Doc/Knowledge/C++/基础语法/基础语法?id=24、c中const和static的作用)

**static**

- 不考虑类的情况
  - 隐藏。所有不加static的全局变量和函数具有全局可见性，可以在其他文件中使用，加了之后只能在该文件所在的编译模块中使用
  - 默认初始化为0，包括未初始化的全局静态变量与局部静态变量，都存在全局未初始化区
  - 静态变量在函数内定义，始终存在，且只进行一次初始化，具有记忆性，其作用范围与局部变量相同，函数退出后仍然存在，但不能使用
- 考虑类的情况
  - static成员变量：只与类关联，不与类的对象关联。定义时要分配空间，不能在类声明中初始化，必须在类定义体外部初始化，初始化时不需要标示为static；可以被非static成员函数任意访问。
  - static成员函数：不具有this指针，无法访问类对象的非static成员变量和非static成员函数；**不能被声明为const、虚函数和volatile**；可以被非static成员函数任意访问

**const**

- 不考虑类的情况
  - const常量在定义时必须初始化，之后无法更改
  - const形参可以接收const和非const类型的实参，例如// i 可以是 int 型或者 const int 型void fun(const int& i){    //...}
- 考虑类的情况
  - const成员变量：不能在类定义外部初始化，只能通过构造函数初始化列表进行初始化，并且必须有构造函数；不同类对其const数据成员的值可以不同，所以不能在类中声明时初始化
  - const成员函数：const对象不可以调用非const成员函数；非const对象都可以调用；不可以改变非mutable（用该关键字声明的变量可以在const成员函数中被修改）数据的值



### 4.3 C++对象模型和this指针

#### 4.3.1 成员变量和成员函数分开存储

在C++中，类内的成员变量和成员函数分开存储

只有**非静态成员变量**才属于类的对象上，**占对象空间**

静态成员变量、成员函数、静态成员函数都不占对象空间



```c++
class person {
public:
};

void test01() {
	person p;
	cout << sizeof(p) << endl;//空对象占用内存空间是一个字节
	//编译器会给每个空对象也分配一个字节空间，是为了区分不同的空对象占内存的位置
    cout<<&p<<endl;
    //每个对象都有一个独一无二的内存地址
    person p3(p);
	cout<<&p3<<endl;  // 地址与P不同，拷贝构造不是拷贝对象，只是把参数复制过去
}

int main() {
	test01();
}
```



```C++
class Person {
public:
	Person() {
		mA = 0;
	}
	// 非静态成员变量占对象空间，这里占用4，直接按int分配
	int mA;
	// 静态成员变量不占对象空间，sizeof不变
	static int mB; 
	// 函数也不占对象空间，所有函数共享一个函数实例，只有一份
	void func() {
		cout << "mA:" << this->mA << endl;
	}
	// 静态成员函数也不占对象空间
	static void sfunc() {
	}
};

int main() {
	cout << sizeof(Person) << endl;

	system("pause");
	return 0;
}
```



#### 4.3.2 this指针概念

通过4.3.1我们知道在C++中成员变量和成员函数是分开存储的

每一个非静态成员函数只会诞生一份函数实例，也就是说多个同类型的对象会共用一块代码

那么问题是：这一块代码是如何区分哪个对象调用自己的呢？



c++通过提供特殊的对象指针，this指针，解决上述问题。**this指针指向被调用的成员函数所属的对象**

this指针是隐含每一个非静态成员函数内的一种指针

this指针不需要定义，直接使用即可



this指针的用途：

*  当形参和成员变量同名时，可用this指针来区分
*  **在类的非静态成员函数中返回对象本身，可使用return *this**(this为一个指向p1(即一个对象)的指针，*this解引用得到p1这个对象) ，可以实现链式编程



```c++
//形参和成员变量同名
class person {
public:
	person(int age) {
		age = age;
        //修改方法：this->age = age;
	}
	int age;
};
void test01() {
	person p1(18);
	cout << p1.age << endl;//输出-566525而不是18，因为形参和成员变量同名，person(int age){age = age;}中将三个age都视为了形参
}
int main() {
	test01();
}
```



```c++
class person {
public:
	person(int age) {
		this->age = age;
	}
	int age;

	void padd(person& p) {
		this->age += p.age;
	}
};
void test01() {
	person p1(10);
	person p2(10);
    //如果p2.padd(p1)返回一个对象的话，就可以实现链式编程
	p2.padd(p1).padd(p1).padd(p1).padd(p1);//报错，因为padd函数的返回值为void
	cout << p2.age << endl;
}

int main() {
	test01();
}
```

如果将padd函数的返回值修改为p2，则就可以继续调用：

```C++
class Person
{
public:

	Person(int age)
	{
		//1、当形参和成员变量同名时，可用this指针来区分
        //this指针指向被调用的成员函数所属的对象
		this->age = age;
	}

	////////////////////////////////////////////////////////////////
    //用引用的方式返回
    Person& PersonAddPerson(Person &p)
	{
		this->age += p.age;
		//返回对象本身
		return *this;//this指向p2的指针，则*this指向p2本体
	}
     ///////////////////////////////////////////////////////////////

	int age;
};

void test01()
{
	Person p1(10);
	cout << "p1.age = " << p1.age << endl;

	Person p2(10);
    ///////////////////////////////////////////////////////////////
    //链式编程思想
	p2.PersonAddPerson(p1).PersonAddPerson(p1).PersonAddPerson(p1);
    ///////////////////////////////////////////////////////////////
	cout << "p2.age = " << p2.age << endl;
}

int main() {
	test01();
	system("pause");
	return 0;
}
```

问题：如果函数 Person& PersonAddPerson(Person &p)的返回为值而不是对象，即 Person PersonAddPerson(Person &p)，结果如何？

调用完第一次p2.PersonAddPerson(p1)后，p2.age变为20，此时返回值为值，因此调用了拷贝构造函数创建一个新的对象，复制一份新的数据出来，即p2'，之后再次调用，又复制出一个p2"。因此最后输出20



```c++
//拷贝构造函数  自动调用时机
//3. 以值方式返回局部对象
Person doWork2()
{
	Person p1;//默认构造
	cout << (int *)&p1 << endl;
	return p1;//根据p1拷贝一个新的对象再返回，拷贝构造
}

void test03()//调用默认构造和拷贝构造
{
	Person p = doWork2();
	cout << (int *)&p << endl;//p和p1的返回地址不一样，不是同一个对象
}
```



#### 4.3.3 空指针访问成员函数

C++中空指针也是可以调用成员函数的，但是也要注意有没有用到this指针

如果用到this指针，需要加以判断保证代码的健壮性



**示例：**

```C++
//空指针访问成员函数
class Person {
public:
	//成员函数
    void ShowClassName() {
		cout << "我是Person类!" << endl;
	}

	void ShowPerson() {
        //因此加了下列代码，如果传的是空指针，则返回，程序就不会报错了，增强代码的健壮性
		if (this == NULL) {
			return;
		}
		cout << mAge << endl;//在属性的前面默认加了个this->mAge，指当前对象的属性，而这里为空指针，因此报错。所以在可能用到this指针的情况下，需要加以判断保证代码的健壮性
	}

public:
	int mAge;
};

void test01()
{
	Person * p = NULL;
	p->ShowClassName(); //空指针，可以调用成员函数
	p->ShowPerson();  //但是如果成员函数中用到了this指针，就不可以了
}

int main() {
	test01();
	system("pause");
	return 0;
}
```



#### 4.3.4 const修饰成员函数

**常函数：**

* 成员函数**后**加const后我们称为这个函数为**常函数**
* 常函数内**不可以修改成员属性**
* 成员属性声明时加关键字mutable后，在常函数中依然可以修改



**常对象：**

* 声明对象前加const称该对象为常对象
* 常对象只能调用常函数



**示例：**

```C++
class Person {
public:
	Person() {
		m_A = 0;
		m_B = 0;
	}

	//this指针的本质是一个常量指针，Person* const this，指针的指向不可修改
    //若在前面再加一个const，则为const Person* const this，指针指向的值也不可以修改。因此这里把前面这个const加到了成员函数后面
	//如果想让指针指向的值也不可以修改，需要声明常函数
	void ShowPerson() const {
		//this = NULL; //不能修改指针的指向 Person* const this;
		//this->mA = 100; //但是this指针指向的对象的数据是可以修改的

		//const修饰成员函数，表示指针指向的内存空间的数据不能修改，除了mutable修饰的变量
		this->m_B = 100;
	}

	void MyFunc() const {
		//mA = 10000;
	}

public:
	int m_A;
	mutable int m_B; //可修改 可变的
};



// const修饰对象  常对象
void test01() {
	const Person person; // 常量对象  
	cout << person.m_A << endl;
	// person.m_A = 100; // 常对象不能修改成员变量的值,但是可以访问
	person.m_B = 100; // 但是常对象可以修改mutable修饰成员变量

	// 常对象访问成员函数
	person.MyFunc(); // 常对象不能调用const的函数

}

int main() {

	test01();

	system("pause");

	return 0;
}
```




### 4.4 友元

在程序里，有些私有属性 也想让类外特殊的一些函数或者类进行访问，就需要用到友元的技术



友元的目的就是**让一个函数或者类 访问另一个类中私有成员**

友元的关键字为  **friend**，**友元不属于类的成员，无所谓访问权限，可以放在类的任意位置**



友元的三种实现

* 全局函数做友元
* 类做友元
* 成员函数做友元 

#### 4.4.1 全局函数做友元

```C++
// 全局函数类外实现
class Building
{
	// 全局函数做友元
    // 告诉编译器 goodGay全局函数 是 Building类的好朋友，可以访问类中的私有内容
	friend void goodGay(Building * building);

public:
	Building()
	{
		this->m_SittingRoom = "客厅";
		this->m_BedRoom = "卧室";
	}

public:
	string m_SittingRoom; //客厅

private:
	string m_BedRoom; //卧室
};

void goodGay(Building * building)
{
	cout << "好基友正在访问： " << building->m_SittingRoom << endl;
	cout << "好基友正在访问： " << building->m_BedRoom << endl;
}

void test01()
{
	Building b;
	goodGay(&b);
}

int main(){
	test01();
	system("pause");
	return 0;
}
```



#### 4.4.2 类做友元

```C++
class Building;
class goodGay
{
public:

	goodGay();//构造函数
	void visit();

private:
	Building *building;
};


class Building
{
	//告诉编译器 goodGay类是Building类的好朋友，可以访问到Building类中私有内容
	friend class goodGay;

public:
	Building();

public:
	string m_SittingRoom; //客厅
private:
	string m_BedRoom;//卧室
};

Building::Building()//类外写构造函数
{
	this->m_SittingRoom = "客厅";
	this->m_BedRoom = "卧室";
}

goodGay::goodGay()//类外写构造函数
{
	building = new Building;//堆区创建B的对象，并用指针维护
}

void goodGay::visit()
{
	cout << "好基友正在访问" << building->m_SittingRoom << endl;
	cout << "好基友正在访问" << building->m_BedRoom << endl;
}

void test01()
{
	goodGay gg;
	gg.visit();

}

int main(){

	test01();

	system("pause");
	return 0;
}
```



#### 4.4.3 成员函数做友元



```C++
class Building;
class goodGay
{
public:

	goodGay();
	void visit(); //只让visit函数作为Building的好朋友，可以发访问Building中私有内容
	void visit2(); 

private:
	Building *building;
};


class Building
{
	//告诉编译器  goodGay类中的visit成员函数 是Building好朋友，可以访问私有内容
	friend void goodGay::visit();

public:
	Building();

public:
	string m_SittingRoom; //客厅
private:
	string m_BedRoom;//卧室
};

Building::Building()
{
	this->m_SittingRoom = "客厅";
	this->m_BedRoom = "卧室";
}

goodGay::goodGay()
{
	building = new Building;
}

void goodGay::visit()
{
	cout << "好基友正在访问" << building->m_SittingRoom << endl;
	cout << "好基友正在访问" << building->m_BedRoom << endl;
}

void goodGay::visit2()
{
	cout << "好基友正在访问" << building->m_SittingRoom << endl;
	//cout << "好基友正在访问" << building->m_BedRoom << endl;
}

void test01()
{
	goodGay  gg;
	gg.visit();
}

int main(){
    
	test01();

	system("pause");
	return 0;
}
```



### 4.5 运算符重载

运算符重载概念：对已有的运算符重新进行定义，赋予其另一种功能，以适应不同的数据类型

#### 4.5.1 加号运算符重载

作用：实现两个自定义数据类型相加的运算

> 注意，局部对象只能返回值，返回引用是非法操作

```C++
class Person {
public:
	Person() {};
	Person(int a, int b)
	{
		this->m_A = a;
		this->m_B = b;
	}
	//成员函数实现 + 号运算符重载
    //这里返回的是值，因为temp为局部对象，当前函数运行完后会被释放，返回引用是非法操作
	Person operator+(const Person& p) {
		Person temp;
		temp.m_A = this->m_A + p.m_A;
		temp.m_B = this->m_B + p.m_B;
		return temp;
	}


public:
	int m_A;
	int m_B;
};

//通过全局函数重载+
//Person operator+(const Person& p1, const Person& p2) {
//	Person temp(0, 0);
//	temp.m_A = p1.m_A + p2.m_A;
//	temp.m_B = p1.m_B + p2.m_B;
//	return temp;
//}

//通过全局函数重载+
//运算符重载，也可以发生函数重载，这里修改数据类型，让person和int的数值相加
Person operator+(const Person& p2, int val)  
{
	Person temp;
	temp.m_A = p2.m_A + val;
	temp.m_B = p2.m_B + val;
	return temp;
}

void test() {

	Person p1(10, 10);
	Person p2(20, 20);

	//成员函数方式
	Person p3 = p2 + p1;  //相当于 p2.operaor+(p1)
	cout << "mA:" << p3.m_A << " mB:" << p3.m_B << endl;


	Person p4 = p3 + 10; //相当于 operator+(p3,10)
	cout << "mA:" << p4.m_A << " mB:" << p4.m_B << endl;

}

int main() {

	test();
	system("pause");
	return 0;
}
```



> 总结1：对于内置的数据类型，如1+1的表达式的的运算符是不可能改变的

> 总结2：不要滥用运算符重载



#### 4.5.2 左移运算符<<重载

作用：可以输出自定义数据类型

一般不会利用成员函数重载<<运算符，只能利用全局函数



```C++
class Person {
    //一般会将成员变量设为私有，因此通过友元来赋予访问权限
	friend ostream& operator<<(ostream& out, Person& p);
public:
	Person(int a, int b)
	{
		this->m_A = a;
		this->m_B = b;
	}
private:
	int m_A;
	int m_B;
};

//疑问一 为什么不能再类内部？若是写在类内部，会报错此运算符函数的参数太多
//原因：在类内部void operator<<(Person& p){}，这个函数因为在类内部，因此需要person调用，而调用这个函数又需要传进去一个person,就成了p.operator<<(p)。但又说，可以void operator<<(cout){}，那就是p.operator<<(cout)，但这时cout在p的右边了，实现不了p << cout，不是我们想要的效果
//疑问二 返回值，引用还是值？引用，需要多次调用，比如cout<<p<<endl;
//全局函数实现左移重载
//ostream对象全局只能有一个，因此必须必须用引用的方式传入，不能值传入创建一个新的出来。值传入的话直接报错
//本质是operator<<(cout,p)，简化为cout<<p
ostream& operator<<(ostream& out, Person& p) {
	out << "a:" << p.m_A << " b:" << p.m_B;
	return out;
}

void test() {
	Person p1(10, 20);
	cout << p1 << "hello world" << endl; //链式编程
}

int main() {
	test();
	system("pause");
	return 0;
}
```

> 总结：重载左移运算符配合友元可以实现输出自定义数据类型



#### 4.5.3 递增运算符重载



作用： 通过重载递增运算符，实现自己的整型数据

递增有前置和后置两种

```C++
class MyInteger {
	friend ostream& operator<<(ostream& out, MyInteger myint);

public:
	MyInteger() {
		m_Num = 0;
	}
    
	//重载前置++运算符
	//返回的需要是一个引用MyInteger&，而不是值MyInteger
	//因为返回是值的话，像这种++(++a)是无法实现的，在++a后返回值，在进行++，会拷贝一个副本进行自增操作
	//返回引用是为了一直对一个数据进行递增操作
	MyInteger& operator++() {
		//先++
		m_Num++;
		//再把自身做一个返回，*this指对象本身
		return *this;
	}

	//后置++
    //这里int代表占位参数，可以用于区分前置和后置递增，编译器会自动标识为后置。这里规定的只能用int，其他float等都不行
    //这里返回的是值，因为temp为局部对象，当前函数运行完后会被释放，返回引用是非法操作
	MyInteger operator++(int) {
		//先 记录当时结果
		MyInteger temp = *this; //记录当前本身的值，然后让本身的值加1，但是返回的是以前的值，达到先返回后++；
        //后递增
		m_Num++;
        //最后将记录的结果进行返回
		return temp;
	}

private:
	int m_Num;
};

ostream& operator<<(ostream& out, MyInteger myint) {
	out << myint.m_Num;
	return out;
}


//前置++ 先++ 再返回
void test01() {
	MyInteger myInt;
	cout << ++myInt << endl;
	cout << myInt << endl;
}

//后置++ 先返回 再++
void test02() {

	MyInteger myInt;
	cout << myInt++ << endl;
	cout << myInt << endl;
}

int main() {

	test01();
	//test02();

	system("pause");

	return 0;
}
```

> 总结： 前置递增返回引用，后置递增返回值



//问题：多次后置递增会报错

```c++
//自写递增
//和视频区别：写在了类外
//缺点：需要友元，且需要传参，没有成员函数用this方便，成员函数的话*this就可以直接返回对象


#include<iostream>
using namespace std;

class MyInteger {
	friend MyInteger& operator++(MyInteger& p);
	friend MyInteger operator++(MyInteger& p, int);
	friend ostream& operator<<(ostream& out, MyInteger myint);
public:
	MyInteger() {
		m_num = 0;
	}
	void prints() {
		cout << m_num << endl;
	}
	
private:
	int m_num;
};
	
//前置自增++a，先自增后返回，
	//疑问一  类内类外？????
	//疑问二  返回引用还是值？引用，需要能重复调用
MyInteger& operator++(MyInteger& p) {
	p.m_num += 1;
	return p;
}

//后置，先返回后自增
//疑问二  返回引用还是值？值，因为这里返回的是局部变量了
//而这就造成无法重复引用
MyInteger operator++(MyInteger& p,int) {
	MyInteger temp= p;
	p.m_num += 1;
	return temp;
}

ostream& operator<<(ostream& out, MyInteger myint) {
	out << myint.m_num;
	return out;
}

int main() {
	MyInteger m;
	//cout << ++(++m) << endl;
	cout << m++ << endl;
	cout << m << endl;

	system("pause");
	return 0;
}
```



#### 4.5.4 赋值运算符重载

c++编译器至少给一个类添加4个函数

1. 默认构造函数(无参，函数体为空)
2. 默认析构函数(无参，函数体为空)
3. 默认拷贝构造函数，对属性进行值拷贝
4. 赋值运算符 operator=, 对属性进行值拷贝



**如果类中有属性指向堆区，做赋值操作时也会出现深浅拷贝问题**



**示例：**



//疑问：为什么operator=必须为成员函数？写在类外就报错，说必须为成员函数
应该是规定

```C++
class Person
{
public:
	Person(int age)
	{
		//将年龄数据开辟到堆区
		m_Age = new int(age);
	}

	//重载赋值运算符 
	Person& operator=(Person &p)
	{
        //编译器提供的代码是浅拷贝
		//m_Age = p.m_Age;
        
        //应该先判断是否有属性在堆区，有的话先释放干净，后进行深拷贝
		if (m_Age != NULL)
		{
			delete m_Age;
			m_Age = NULL;
		}
		
		//提供深拷贝 解决浅拷贝的问题
		m_Age = new int(*p.m_Age);

		//返回自身
		return *this;
	}


	~Person()
	{
		if (m_Age != NULL)
		{
			delete m_Age;
			m_Age = NULL;
		}
	}

	//年龄的指针
	int *m_Age;

};


void test01()
{
	Person p1(18);

	Person p2(20);

	Person p3(30);

	p3 = p2 = p1; //赋值操作，若没有自写拷贝构造函数，则会出现浅拷贝的问题，即开辟在堆区的内存被重复释放。这里直接重写赋值运算，让在赋值的时候自动开辟新内存

	cout << "p1的年龄为：" << *p1.m_Age << endl;

	cout << "p2的年龄为：" << *p2.m_Age << endl;

	cout << "p3的年龄为：" << *p3.m_Age << endl;
}

int main() {

	test01();

	//int a = 10;
	//int b = 20;
	//int c = 30;

	//c = b = a;
	//cout << "a = " << a << endl;//10
	//cout << "b = " << b << endl;//10
	//cout << "c = " << c << endl;//10

	system("pause");

	return 0;
}
```



#### 4.5.5 关系运算符重载

**作用：**重载关系运算符，可以让两个自定义类型对象进行对比操作

**示例：**

```C++
class Person
{
public:
	Person(string name, int age)
	{
		this->m_Name = name;
		this->m_Age = age;
	};

	bool operator==(Person & p)
	{
		if (this->m_Name == p.m_Name && this->m_Age == p.m_Age)
		{
			return true;
		}
		else
		{
			return false;
		}
	}

	bool operator!=(Person & p)
	{
		if (this->m_Name == p.m_Name && this->m_Age == p.m_Age)
		{
			return false;
		}
		else
		{
			return true;
		}
	}

	string m_Name;
	int m_Age;
};

void test01()
{
	//int a = 0;
	//int b = 0;

	Person a("孙悟空", 18);
	Person b("孙悟空", 18);

	if (a == b)
	{
		cout << "a和b相等" << endl;
	}
	else
	{
		cout << "a和b不相等" << endl;
	}

	if (a != b)
	{
		cout << "a和b不相等" << endl;
	}
	else
	{
		cout << "a和b相等" << endl;
	}
}

int main() {
	test01();
	system("pause");
	return 0;
}
```



#### 4.5.6 函数调用运算符()重载

* 函数调用运算符 ()  也可以重载
* 由于重载后使用的方式非常像函数的调用，因此称为**仿函数**
* 仿函数没有固定写法，非常灵活，可以实现各种功能



**示例：**

```C++
class MyPrint
{
public:
	void operator()(string text)
	{
		cout << text << endl;
	}

};

void test01()
{
	//重载的（）操作符 也称为仿函数
	MyPrint myFunc;
	myFunc("hello world");//由于使用起来非常类似于函数调用，因此成为仿函数
}

//加法类
class MyAdd
{
public:
	int operator()(int v1, int v2)
	{
		return v1 + v2;
	}
};

void test02()
{
	MyAdd madd;
	int ret = madd(10, 10);
	cout << "ret = " << ret << endl;

	//匿名对象调用，通过类名加()创建一个匿名对象
	cout << "MyAdd()(100,100) = " << MyAdd()(100, 100) << endl;
}

int main() {

	test01();
	test02();

	system("pause");
	return 0;
}
```



### 4.6  继承

**继承是面向对象三大特性之一**

有些类与类之间存在特殊的关系，例如下图中：

![1544861202252](assets/1544861202252.png)

我们发现，定义这些类时，下级别的成员除了拥有上一级的共性，还有自己的特性。

这个时候我们就可以考虑利用继承的技术，**减少重复代码**







利用工具查看：



![1545881904150](assets/1545881904150.png)



打开工具窗口后，定位到当前CPP文件的盘符

然后输入： cl /d1 reportSingleClassLayout查看的类名   所属文件名



效果如下图：



![1545882158050](assets/1545882158050.png)

> 结论： 父类中私有成员也是被子类继承下去了，只是由编译器给隐藏后访问不到

 



#### 4.6.5 继承同名成员处理方式

问题：当子类与父类出现同名的成员，如何通过子类对象，访问到子类或父类中同名的数据呢？

* 访问子类同名成员   直接访问即可
* 访问父类同名成员   需要加作用域



**示例：**

```C++
class Base {
public:
	Base()
	{
		m_A = 100;
	}
	void func()
	{
		cout << "Base - func()调用" << endl;
	}
	void func(int a)
	{
		cout << "Base - func(int a)调用" << endl;
	}
public:
	int m_A;
};

class Son : public Base {
public:
	Son()
	{
		m_A = 200;
	}

	//当子类与父类拥有同名的成员函数，子类会隐藏父类中所有版本的同名成员函数
	//如果想访问父类中被隐藏的同名成员函数，需要加父类的作用域
	void func()
	{
		cout << "Son - func()调用" << endl;
	}
public:
	int m_A;
};

void test01()
{
	Son s;
	cout << "Son下的m_A = " << s.m_A << endl;
	cout << "Base下的m_A = " << s.Base::m_A << endl;

	s.func();
	s.Base::func();
	s.Base::func(10);

}
int main() {

	test01();

	system("pause");
	return EXIT_SUCCESS;
}
```

总结：

1. 子类对象可以直接访问到子类中同名成员
2. 子类对象加作用域可以访问到父类同名成员
3. 当子类与父类拥有同名的成员函数，子类会隐藏父类中同名成员函数，加作用域可以访问到父类中同名函数



#### 4.6.6 继承同名静态成员处理方式

问题：继承中同名的静态成员在子类对象上如何进行访问？

静态成员和非静态成员出现同名，处理方式一致

- 访问子类同名成员   直接访问即可
- 访问父类同名成员   需要加作用域

**示例：**

```C++
class Base {
public:
	static void func()
	{
		cout << "Base - static void func()" << endl;
	}
	static void func(int a)
	{
		cout << "Base - static void func(int a)" << endl;
	}
	static int m_A;
};

int Base::m_A = 100;

class Son : public Base {
public:
	static void func()
	{
		cout << "Son - static void func()" << endl;
	}
	static int m_A;
};

int Son::m_A = 200;

//同名成员属性
void test01()
{
	//通过对象访问
	cout << "通过对象访问： " << endl;
	Son s;
	cout << "Son  下 m_A = " << s.m_A << endl;
	cout << "Base 下 m_A = " << s.Base::m_A << endl;

	//通过类名访问
	cout << "通过类名访问： " << endl;
	cout << "Son  下 m_A = " << Son::m_A << endl;
	cout << "Base 下 m_A = " << Son::Base::m_A << endl;
}

//同名成员函数
void test02()
{
	//通过对象访问
	cout << "通过对象访问： " << endl;
	Son s;
	s.func();
	s.Base::func();

	cout << "通过类名访问： " << endl;
	Son::func();
	Son::Base::func();
	//出现同名，子类会隐藏掉父类中所有同名成员函数，需要加作作用域访问
	Son::Base::func(100);
}
int main() {

	//test01();
	test02();

	system("pause");
	return 0;
}
```

> 总结：同名静态成员处理方式和非静态处理方式一样，只不过有两种访问的方式（通过对象 和 通过类名）



#### 4.6.7 多继承语法

C++允许**一个类继承多个类**

语法：` class 子类 ：继承方式 父类1 ， 继承方式 父类2...`

多继承可能会引发父类中有同名成员出现，需要加作用域区分

**C++实际开发中不建议用多继承**

**示例：**

```C++
class Base1 {
public:
	Base1()
	{
		m_A = 100;
	}
public:
	int m_A;
};

class Base2 {
public:
	Base2()
	{
		m_A = 200;  //开始是m_B 不会出问题，但是改为mA就会出现不明确
	}
public:
	int m_A;
};

//语法：class 子类：继承方式 父类1 ，继承方式 父类2 
class Son : public Base2, public Base1 
{
public:
	Son()
	{
		m_C = 300;
		m_D = 400;
	}
public:
	int m_C;
	int m_D;
};


//多继承容易产生成员同名的情况
//通过使用类名作用域可以区分调用哪一个基类的成员
void test01()
{
	Son s;
	cout << "sizeof Son = " << sizeof(s) << endl;
	cout << s.Base1::m_A << endl;
	cout << s.Base2::m_A << endl;
}

int main() {
	test01();
	system("pause");
	return 0;
}
```

> 总结： 多继承中如果父类中出现了同名情况，子类使用时候要加作用域







### 4.7  多态

#### 4.7.1 多态的基本概念

**多态是C++面向对象三大特性之一**

//多态满足条件： 
//1、有继承关系
//2、子类重写父类中的虚函数，即子类中的同名函数的函数返回值类型、函数名称、参数列表完全相同。父类函数前需要加virtual变为虚函数
//多态使用：
//父类指针或引用指向子类对象



下面通过案例进行讲解多态

```C++
class Animal//原来这个类占1个字节，因为非静态函数不属于这个对象，但加上virtual后变为四个字节，因为多了个指针vfptr，即虚函数指针，指向虚函数表Vftable，表内记录虚函数地址&Animal::speak。子类继承了父类的这些东西，但当子类重写的时候，子类中的虚函数表内部会替换为子类的虚函数地址&cat::speak
{
public:
	//Speak函数就是虚函数
	//函数前面加上virtual关键字，变成虚函数，那么编译器在编译的时候就不能确定函数调用了。
	virtual void speak()
	{
		cout << "动物在说话" << endl;
	}
};

class Cat :public Animal
{
public:
	void speak()
	{
		cout << "小猫在说话" << endl;
	}
};

class Dog :public Animal
{
public:
	void speak()
	{
		cout << "小狗在说话" << endl;
	}
};
//我们希望传入什么对象，那么就调用什么对象的函数
//如果函数地址在编译阶段就能确定，那么静态联编
//如果函数地址在运行阶段才能确定，就是动态联编

//2.为什么输出动物呢？因为在这里地址早绑定，在编译阶段就确定函数地址了
//3.想执行让猫说话，则函数地址不能早绑定，需要在运行阶段绑定
//4.做法：virtual void speak()
void DoSpeak(Animal & animal)
{
	animal.speak();
}
//


void test01()
{
	Cat cat;
	DoSpeak(cat);//1.这里用Animal & animal接收cat，输出 动物在说话
    
	Dog dog;
	DoSpeak(dog);
}


int main() {

	test01();
	system("pause");
	return 0;
}
```

总结：

多态满足条件

* 有继承关系
* 子类重写父类中的虚函数

多态使用条件

* 父类指针或引用指向子类对象

重写：函数返回值类型  函数名 参数列表 完全一致称为重写



#### 4.7.2 多态案例一-计算器类

案例描述：

分别利用普通写法和多态技术，设计实现两个操作数进行运算的计算器类



多态的优点：

* 代码组织结构清晰
* 可读性强
* 利于前期和后期的扩展以及维护



**示例：**

```C++
//普通实现
class Calculator {
public:
	int getResult(string oper)
	{
		if (oper == "+") {
			return m_Num1 + m_Num2;
		}
		else if (oper == "-") {
			return m_Num1 - m_Num2;
		}
		else if (oper == "*") {
			return m_Num1 * m_Num2;
		}
		//问题：如果要提供新的运算，需要修改源码
        //在真实开发中，提倡开闭原则，对扩展进行开放，对修改进行关闭
	}
public:
	int m_Num1;
	int m_Num2;
};

void test01()
{
	//普通实现测试
	Calculator c;
	c.m_Num1 = 10;
	c.m_Num2 = 10;
	cout << c.m_Num1 << " + " << c.m_Num2 << " = " << c.getResult("+") << endl;

	cout << c.m_Num1 << " - " << c.m_Num2 << " = " << c.getResult("-") << endl;

	cout << c.m_Num1 << " * " << c.m_Num2 << " = " << c.getResult("*") << endl;
}


//多态实现
//抽象计算器类
//多态优点：代码组织结构清晰，可读性强，利于前期和后期的扩展以及维护
class AbstractCalculator
{
public :
	virtual int getResult()
	{
		return 0;
	}
	int m_Num1;
	int m_Num2;
};

//加法计算器
class AddCalculator :public AbstractCalculator
{
public:
	int getResult()
	{
		return m_Num1 + m_Num2;
	}
};

//减法计算器
class SubCalculator :public AbstractCalculator
{
public:
	int getResult()
	{
		return m_Num1 - m_Num2;
	}
};

//乘法计算器
class MulCalculator :public AbstractCalculator
{
public:
	int getResult()
	{
		return m_Num1 * m_Num2;
	}
};


void test02()
{
	//多态使用条件
    //父类指针或者引用指向子类对象
    //创建加法计算器
	AbstractCalculator *abc = new AddCalculator;//创建了一个加法计算器的对象，是匿名对象，然后new的话用指针接收
	abc->m_Num1 = 10;
	abc->m_Num2 = 10;
	cout << abc->m_Num1 << " + " << abc->m_Num2 << " = " << abc->getResult() << endl;
	delete abc;  //堆区数据，用完了记得销毁

	//创建减法计算器
	abc = new SubCalculator;
	abc->m_Num1 = 10;
	abc->m_Num2 = 10;
	cout << abc->m_Num1 << " - " << abc->m_Num2 << " = " << abc->getResult() << endl;
	delete abc;  

	//创建乘法计算器
	abc = new MulCalculator;
	abc->m_Num1 = 10;
	abc->m_Num2 = 10;
	cout << abc->m_Num1 << " * " << abc->m_Num2 << " = " << abc->getResult() << endl;
	delete abc;
}

int main() {
	//test01();
	test02();
	system("pause");
	return 0;
}
```

> 总结：C++开发提倡利用多态设计程序架构，因为多态优点很多



#### 4.7.3 纯虚函数和抽象类

在多态中，通常父类中虚函数的实现是毫无意义的，主要都是调用子类重写的内容

因此可以将虚函数改为**纯虚函数**

纯虚函数语法：`virtual 返回值类型 函数名 （参数列表）= 0 ;`

当类中有了纯虚函数，这个类也称为**抽象类**



**抽象类特点**：

 * 无法实例化对象
 * 子类必须重写抽象类中的纯虚函数，否则也属于抽象类



**示例：**

```C++
class Base
{
public:
	//纯虚函数
	//类中只要有一个纯虚函数就称为抽象类
	//抽象类无法实例化对象
	//子类必须重写父类中的纯虚函数，否则也属于抽象类
	virtual void func() = 0;
};

class Son :public Base
{
public:
	virtual void func() 
	{
		cout << "func调用" << endl;
	};
};

void test01()
{
	//Base b;//报错，无法实例化对象
    Base * base = NULL;
	//base = new Base; // 错误，抽象类无法实例化对象
	base = new Son;
	base->func();
	delete base;//记得销毁
}

int main() {
	test01();
	system("pause");
	return 0;
}
```





#### 4.7.4 多态案例二-制作饮品

**案例描述：**

制作饮品的大致流程为：煮水 -  冲泡 - 倒入杯中 - 加入辅料
利用多态技术实现本案例，提供抽象制作饮品基类，提供子类制作咖啡和茶叶

![1545985945198](assets/1545985945198.png)

**示例**

```C++
//抽象制作饮品
class AbstractDrinking {
public:
	//烧水
	virtual void Boil() = 0;
	//冲泡
	virtual void Brew() = 0;
	//倒入杯中
	virtual void PourInCup() = 0;
	//加入辅料
	virtual void PutSomething() = 0;
	//规定流程
	void MakeDrink() {
		Boil();
		Brew();
		PourInCup();
		PutSomething();
	}
};

//制作咖啡
class Coffee : public AbstractDrinking {
public:
	//烧水
	virtual void Boil() {
		cout << "煮农夫山泉!" << endl;
	}
	//冲泡
	virtual void Brew() {
		cout << "冲泡咖啡!" << endl;
	}
	//倒入杯中
	virtual void PourInCup() {
		cout << "将咖啡倒入杯中!" << endl;
	}
	//加入辅料
	virtual void PutSomething() {
		cout << "加入牛奶!" << endl;
	}
};

//制作茶水
class Tea : public AbstractDrinking {
public:
	//烧水
	virtual void Boil() {
		cout << "煮自来水!" << endl;
	}
	//冲泡
	virtual void Brew() {
		cout << "冲泡茶叶!" << endl;
	}
	//倒入杯中
	virtual void PourInCup() {
		cout << "将茶水倒入杯中!" << endl;
	}
	//加入辅料
	virtual void PutSomething() {
		cout << "加入枸杞!" << endl;
	}
};


//业务函数，这里的接收相当于AbstractDrinking* drink=new Coffee，因此之后也需要delete
void DoWork(AbstractDrinking* drink) {
	drink->MakeDrink();
	delete drink;
}

void test01() {
	DoWork(new Coffee);
	cout << "--------------" << endl;
	DoWork(new Tea);
}


int main() {
	test01();
	system("pause");
	return 0;
}
```



#### 4.7.5 虚析构和纯虚析构

​    多态使用时，父类的指针或引用指向子类对象，而父类指针无法释放子类中的析构代码，如果子类中有属性开辟到堆区，那么父类指针在释放时无法调用到子类的析构代码
​	解决方式：将父类中的析构函数改为**虚析构**或者**纯虚析构**

虚析构和纯虚析构共性：

* 可以解决父类指针释放子类对象
* **都需要有具体的函数实现**

虚析构和纯虚析构区别：

* 如果是纯虚析构，该类属于抽象类，无法实例化对象



虚析构语法：

`virtual ~类名(){}`

纯虚析构语法：

` virtual ~类名() = 0;`

`类名::~类名(){}`



**示例：**

```C++
class Animal {
public:

	Animal(){
		cout << "Animal 构造函数调用！" << endl;
	}
	virtual void Speak() = 0;

	//析构函数加上virtual关键字，变成虚析构函数
	//virtual ~Animal()
	//{
	//	cout << "Animal虚析构函数调用！" << endl;
	//}


	//纯虚析构，需要有这条声明，也需要有具体的实现
    virtual ~Animal() = 0;
};

//纯虚析构的实现
Animal::~Animal()
{
	cout << "Animal 纯虚析构函数调用！" << endl;
}

//和包含普通纯虚函数的类一样，包含了纯虚析构函数的类也是一个抽象类。不能够被实例化。

class Cat : public Animal {
public:
	Cat(string name)
	{
		cout << "Cat构造函数调用！" << endl;
		m_Name = new string(name);
	}
	virtual void Speak()
	{
		cout << *m_Name <<  "小猫在说话!" << endl;
	}
	~Cat()
	{
		cout << "Cat析构函数调用!" << endl;
		if (this->m_Name != NULL) {
			delete m_Name;
			m_Name = NULL;
		}
	}

public:
	string *m_Name;
};

void test01()
{
	Animal *animal = new Cat("Tom");
	animal->Speak();

	//父类指针在析构时，不会调用子类的析构函数
    //通过父类指针去释放，会导致子类对象可能清理不干净，造成内存泄漏
	//怎么解决？给基类增加一个虚析构函数
	//虚析构函数就是用来解决通过父类指针释放子类对象
	delete animal;
}

int main() {

	test01();

	system("pause");

	return 0;
}
```

总结：

	1. 虚析构或纯虚析构就是用来解决通过父类指针释放子类对象
	1. 如果子类中没有堆区数据，可以不写为虚析构或纯虚析构
	
	2. 拥有纯虚析构函数的类也属于抽象类



#### 4.7.6 多态案例三-电脑组装



**案例描述：**

电脑主要组成部件为 CPU（用于计算），显卡（用于显示），内存条（用于存储）

将每个零件封装出抽象基类，并且提供不同的厂商生产不同的零件，例如Intel厂商和Lenovo厂商

创建电脑类提供让电脑工作的函数，并且调用每个零件工作的接口

测试时组装三台不同的电脑进行工作



**示例：**

```C++
#include<iostream>
using namespace std;

//抽象CPU类
class CPU
{
public:
	//抽象的计算函数
	virtual void calculate() = 0;
};

//抽象显卡类
class VideoCard
{
public:
	//抽象的显示函数
	virtual void display() = 0;
};

//抽象内存条类
class Memory
{
public:
	//抽象的存储函数
	virtual void storage() = 0;
};

//电脑类
class Computer
{
public:
	Computer(CPU * cpu, VideoCard * vc, Memory * mem)
	{
		m_cpu = cpu;
		m_vc = vc;
		m_mem = mem;
	}

	//提供工作的函数
	void work()
	{
		//让零件工作起来，调用接口
		m_cpu->calculate();

		m_vc->display();

		m_mem->storage();
	}

	//提供析构函数 释放3个电脑零件
	~Computer()
	{

		//释放CPU零件
		if (m_cpu != NULL)
		{
			delete m_cpu;
			m_cpu = NULL;
		}

		//释放显卡零件
		if (m_vc != NULL)
		{
			delete m_vc;
			m_vc = NULL;
		}

		//释放内存条零件
		if (m_mem != NULL)
		{
			delete m_mem;
			m_mem = NULL;
		}
	}

private:

	CPU * m_cpu; //CPU的零件指针
	VideoCard * m_vc; //显卡零件指针
	Memory * m_mem; //内存条零件指针
};

//具体厂商
//Intel厂商
class IntelCPU :public CPU
{
public:
	virtual void calculate()
	{
		cout << "Intel的CPU开始计算了！" << endl;
	}
};

class IntelVideoCard :public VideoCard
{
public:
	virtual void display()
	{
		cout << "Intel的显卡开始显示了！" << endl;
	}
};

class IntelMemory :public Memory
{
public:
	virtual void storage()
	{
		cout << "Intel的内存条开始存储了！" << endl;
	}
};

//Lenovo厂商
class LenovoCPU :public CPU
{
public:
	virtual void calculate()
	{
		cout << "Lenovo的CPU开始计算了！" << endl;
	}
};

class LenovoVideoCard :public VideoCard
{
public:
	virtual void display()
	{
		cout << "Lenovo的显卡开始显示了！" << endl;
	}
};

class LenovoMemory :public Memory
{
public:
	virtual void storage()
	{
		cout << "Lenovo的内存条开始存储了！" << endl;
	}
};

void test01()
{
	//第一台电脑零件
	CPU * intelCpu = new IntelCPU;
	VideoCard * intelCard = new IntelVideoCard;
	Memory * intelMem = new IntelMemory;

	cout << "第一台电脑开始工作：" << endl;
	//创建第一台电脑
	Computer * computer1 = new Computer(intelCpu, intelCard, intelMem);
	computer1->work();
	delete computer1;

	cout << "-----------------------" << endl;
	cout << "第二台电脑开始工作：" << endl;
	//第二台电脑组装
	Computer * computer2 = new Computer(new LenovoCPU, new LenovoVideoCard, new LenovoMemory);;
	computer2->work();
	delete computer2;

	cout << "-----------------------" << endl;
	cout << "第三台电脑开始工作：" << endl;
	//第三台电脑组装
	Computer * computer3 = new Computer(new LenovoCPU, new IntelVideoCard, new LenovoMemory);;
	computer3->work();
	delete computer3;

}
```



## 5 文件操作

程序运行时产生的数据都属于临时数据，程序一旦运行结束都会被释放

通过**文件可以将数据持久化**

C++中对文件操作需要包含头文件 &lt; fstream &gt;



文件类型分为两种：

1. **文本文件**     -  文件以文本的**ASCII码**形式存储在计算机中
2. **二进制文件** -  文件以文本的**二进制**形式存储在计算机中，用户一般不能直接读懂它们



操作文件的三大类:

1. ofstream：写操作
2. ifstream： 读操作
3. fstream ： 读写操作



### 5.1文本文件

#### 5.1.1写文件

   写文件步骤如下：

1. 包含头文件   

     \#include <fstream\>

2. 创建流对象  

   ofstream ofs;

3. 打开文件

   ofs.open("文件路径",打开方式);

4. 写数据

   ofs << "写入的数据";

5. 关闭文件

   ofs.close();

   

文件打开方式：

| 打开方式    | 解释                       |
| ----------- | -------------------------- |
| ios::in     | 为读文件而打开文件         |
| ios::out    | 为写文件而打开文件         |
| ios::ate    | 初始位置：文件尾           |
| ios::app    | 追加方式写文件             |
| ios::trunc  | 如果文件存在先删除，再创建 |
| ios::binary | 二进制方式                 |

**注意：** 文件打开方式可以配合使用，利用|操作符

**例如：**用二进制方式写文件 `ios::binary | ios:: out`



```c++
//清空文件
//打开模式 ios::trunc 如果存在删除文件并重新创建
ofstream ofs(FILENAME, ios::trunc);
ofs.close();
```



**示例：**

```C++
#include <fstream>

void test01()
{
	ofstream ofs;
	ofs.open("test.txt", ios::out);

	ofs << "姓名：张三" << endl;
	ofs << "性别：男" << endl;
	ofs << "年龄：18" << endl;

	ofs.close();
}

int main() {

	test01();

	system("pause");

	return 0;
}
```

总结：

* 文件操作必须包含头文件 fstream
* 读文件可以利用 ofstream  ，或者fstream类
* 打开文件时候需要指定操作文件的路径，以及打开方式
* 利用<<可以向文件中写数据
* 操作完毕，要关闭文件



#### 5.1.2读文件

读文件与写文件步骤相似，但是读取方式相对于比较多



读文件步骤如下：

1. 包含头文件   

     \#include <fstream\>

2. 创建流对象  

   ifstream ifs;

3. 打开文件并判断文件是否打开成功

   ifs.open("文件路径",打开方式);

4. 读数据

   四种方式读取

5. 关闭文件

   ifs.close();



**示例：**

```C++
#include <fstream>
#include <string>
void test01()
{
	ifstream ifs;
	ifs.open("test.txt", ios::in);

	if (!ifs.is_open())
	{
		cout << "文件打开失败" << endl;
		return;
	}

	//第一种方式
	//char buf[1024] = { 0 };
	//while (ifs >> buf)
	//{
	//	cout << buf << endl;
	//}
    //逐词读取while(ifs>>buf)，遇到空格、换行或文件尾结束一次读取，即一次读取连续的内容

	//第二种
	//char buf[1024] = { 0 };
	//while (ifs.getline(buf,sizeof(buf)))
	//{
	//	cout << buf << endl;
	//}

	//第三种
	//string buf;
	//while (getline(ifs, buf))
	//{
	//	cout << buf << endl;
	//}

	//第四种方法，不是很推荐用，效率低
    char c;
	while ((c = ifs.get()) != EOF)
	{
		cout << c;
	}

	ifs.close();


}

int main() {

	test01();
	system("pause");
	return 0;
}
```

总结：

- 读文件可以利用 ifstream  ，或者fstream类
- 利用is_open函数可以判断文件是否打开成功
- close 关闭文件 



### 5.2 二进制文件

以二进制的方式对文件进行读写操作，可以写入自定义的数据类型

打开方式要指定为 ios::binary



#### 5.2.1 写文件

二进制方式写文件主要利用流对象调用成员函数write

函数原型 ：`ostream& write(const char * buffer,int len);`

参数解释：字符指针buffer指向内存中一段存储空间。len是读写的字节数



**示例：**

```C++
#include <fstream>
#include <string>

class Person
{
public:
	char m_Name[64];//不要用string，会出现各种问题
	int m_Age;
};

//二进制文件  写文件
void test01()
{
	//1、包含头文件

	//2、创建输出流对象
	ofstream ofs("person.txt", ios::out | ios::binary);
	
	//3、打开文件
	//ofs.open("person.txt", ios::out | ios::binary);

	Person p = {"张三"  , 18};

	//4、写文件
	ofs.write((const char *)&p, sizeof(p));

	//5、关闭文件
	ofs.close();
}

int main() {
	test01();
	system("pause");
	return 0;
}
```

总结：

* 文件输出流对象 可以通过write函数，以二进制方式写数据



#### 5.2.2 读文件

二进制方式读文件主要利用流对象调用成员函数read

函数原型：`istream& read(char *buffer,int len);`

参数解释：字符指针buffer指向内存中一段存储空间。len是读写的字节数

示例：

```C++
#include <fstream>
#include <string>

class Person
{
public:
	char m_Name[64];
	int m_Age;
};

void test01()
{
	ifstream ifs("person.txt", ios::in | ios::binary);
	if (!ifs.is_open()){
		cout << "文件打开失败" << endl;}

	Person p;
	ifs.read((char *)&p, sizeof(p));

	cout << "姓名： " << p.m_Name << " 年龄： " << p.m_Age << endl;
}

int main() {
	test01();
	system("pause");
	return 0;
}
```

- 文件输入流对象 可以通过read函数，以二进制方式读数据

