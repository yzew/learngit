[参考1](https://normaluhr.github.io/2020/12/31/Effective-C++/)

## 条款09：绝不在构造和析构过程中调用`virtual`函数

> 在构造和析构期间不要调用virtual函数，因为这类调用从不下降至derived class（如base class构造期间virtual函数绝不会下降到derived class阶层）

​	在多态环境中，我们需要重新理解构造函数和析构函数的意义，这两个函数在执行过程中，涉及到了对象类型从基类到子类，再从子类到基类的转变。

​	一个子类对象开始创建时，首先调用的是基类的构造函数，在调用子类构造函数之前，该对象**将一直保持着“基类对象”的身份而存在**，因此在基类的构造函数中调用的虚函数将会是基类的虚函数版本，在子类的构造函数中，**原先的基类对象变成了子类对象**，这时子类构造函数里调用的是子类的虚函数版本。这是一件有意思的事情，这说明在**构造函数中虚函数并不是虚函数**，在不同的构造函数中，调用的虚函数版本并不同，因为随着不同层级的构造函数调用时，对象的类型在实时变化。那么相似的，析构函数也是如此

​	因此，为了避免调用到错误版本的虚函数，确保不要在构造函数和析构函数中用到虚函数。但很遗憾的是，你可能并没有意识到自己做出了这样的设计，例如将构造函数的主要工作抽象成一个`init()`函数以防止不同构造函数的代码重复是一个很常见的做法，但是在`init()`函数中是否调用了虚函数，就要好好注意一下了，同样的情况在析构函数中也是一样。

​		若想确保每次继承体系上的对象被创建，都会调用适当版本的虚函数，可以采用这个方案：在base class中将需要的函数改为non-virtual，然后要求derived class构造函数传递必要信息给base class构造函数。即 **令derived classes将必要的构造信息向上传递至base class构造函数**

**Example**

原：
![image-20221113150846870](E:\MarkDown\picture\image-20221113150846870.png)
![image-20221113150907329](E:\MarkDown\picture\image-20221113150907329.png)

改进：

![image-20221113150421308](E:\MarkDown\picture\image-20221113150421308.png)

注意本例中private static函数createLogString的运用。比起在成员初值列内给予base class所需数据，利用辅助函数创建一个值传给base class构造函数往往比较方便(也比较可读)。令此函数为static ，静态成员函数不能访问普通成员变量，只能访问静态成员变量，因此也就不可能意外指向“初期未成熟之BuyTransaction对象内尚未初始化的成员变量”。这很重要，正是因为“那些成员变量处于未定义状态”，所以“在baseclass构造和析构期间调用的virtual函数不可下降至derived classes”

## 条款10：令operator= 返回一个reference to *this

> 令赋值( assignment) 操作符及其他赋值相关操作（+=、*=等）返回一个reference to *this.

这样做可以让你的赋值操作符实现“连等”的效果，即写成连锁形式：`x = y = z = 10;`

在设计接口时一个重要的原则是，**让自己的接口和内置类型相同功能的接口尽可能相似**，所以如果没有特殊情况，就请让你的赋值操作符的返回类型为`ObjectClass&`类型并在代码中返回`*this`。

```cpp
class Weight {
public:
    Widget& operator= (const widget& rhs) {
        return* this;
    }
};
```

## 条款11：在operator=中处理“自我赋值”

> * 确保当对象自我赋值时operator=有良好行为。其中技术包括比较“来源对象”和“目标对象”的地址、精心周到的语句顺序、以及copy-and-swap
> * 确定任何函数如果操作一个以上的对象，而其中多个对象是同一个对象时，其行为仍然正确

自我赋值指的是将自己赋给自己。这是一种看似愚蠢无用但却在代码中出现次数比任何人想象的多得多的操作，这种操作常常需要假借指针来实现：

```cpp
Wight w;
w = w;
// 潜在的自我赋值
*pa = *pb;		 			//pa和pb指向同一对象，便是自我赋值
arr[i] = arr[j];		//i和j相等，便是自我赋值
class Base { ... };
class Derived: public Base { ....};
void doSomething (const Base& rb, Derived* pd);
//rb和*pd有可能其实是同一对象
```

对于管理一定资源的对象重载的operator=中，一定要对是不是自我赋值格外小心并且增加预判，因为无论是深拷贝还是资源所有权的转移，原先的内存或所有权一定会被清空才能被赋值，如果不加处理，这套逻辑被用在自我赋值上会发生——先把自己的资源给释放掉了，然后又把已释放掉的资源赋给了自己导致出错。

```cpp
// 不安全的自我赋值
Widget&
Widget::operator=(const Widget& rhs)  // 一份不安全的 operator=实现版本.
{
	delete pb;  // 停止使用当前的bitmap,
	pb = new Bitmap (*rhs.pb);  // 使用rhs's bitmap的副本(复件)。
	return *this;
}
```

**做法一：在赋值前增加预判**

在最前面增加一个证同测试(identity test)

问题：无异常安全性，试想如果在删除掉原指针指向的内存后，在赋值之前任何一处跑出了异常，那么原指针就指向了一块已经被删除的内存；自我赋值的频率比较小，将 证同测试 放到起始处，多次的测试会降低执行速度

```cpp
Widget& Widget::operator=(const Widget& rhs)
{
	if (this == &rhs) return *this;  //证同测试(identity test) :
	delete pb;
	pb = new Bitmap(*rhs.pb);  // 如果此处抛出异常，ptr将指向一块已经被删除的内存。
	return *this;
}
```

**做法二：考虑异常安全性(exception safety)**

问题：对原对象做了一份附件，没那么高效。

```cpp
Widget& Widget::operator=(const Widget& rhs)
{
	Bitmap* pOrig = pb;  // 记住原先的pb
    // 如果new处抛出异常，ptr仍然指向之前的内存
	pb = new Bitmap (*rhs.pb);  // 令pb指向*pb的一个复件(副本)
	delete pOrig;  // 删除原先的pb
	return *this;
}

```

**做法三：copy and swap**

copy and swap策略的原则很简单：为你打算修改的对象(原件)做出一份副本，然后在那副本身上做一切必要修改。若有任何修改动作抛出异常，原对象仍保持未改变状态。待所有改变都成功后，再将修改过的那个副本和原对象在一个不抛出异常的操作中置换(swap) 

若形参是引用，则创建一份副本后交换；若形参是值传递，则自动传入一个副本，可以直接进行交换操作，这就是将copy动作从函数本体移动到函数参数构造阶段，令编译器生成更高效的代码

```cpp
Widget& Widget::operator=(const widget& rhs)
{
	Widget temp(rhs);  // 为rhs数据制作一份复件(副本)
	swap(temp);  // 将*this数据和上述复件的数据交换。
	return *this;
}

// 写法二
// 注意这里是pass by value.
widget& widget::operator= (Widget rhs) //rhs是被传对象的一份复件(副本)
{
    swap (rhs);  //将*this的数据和复件/副本的数据互换
	return *this;
}

```

## 条款12：复制对象时勿忘其每一个成分

> * Copying函数应该确保复制“对象内的所有成员变量"及“所有base class成分”
>
> * 不要尝试以某个copying函数实现另一个copying 函数。应该将共同机能放进第三个函数中，并由两个coping函数共同调用

设计良好之面向对象系统(OO-systems) 会将对象的内部封装起来，只留两个函数负责对象拷贝(复制)，那便是带着适切名称的copy构造函数和copy assignment操作符，我称它们为copying函数。

所谓“每一个成分”，作者在这里其实想要提醒大家两点：

- 当你给类多加了成员变量时，请不要忘记在拷贝构造函数和赋值操作符中对新加的成员变量进行处理。如果你忘记处理，编译器也不会报错。

- 如果你的类有继承，那么在你为子类编写拷贝构造函数时要注意复制基类的每一个成分，这些成分往往是private的，所以你无法访问它们，你应该让子类使用子类的拷贝构造函数和拷贝赋值运算符去调用相应基类的拷贝构造函数和拷贝赋值运算符：

  ```cpp
  // 在成员初始化列表显式的调用基类的拷贝构造函数
  PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs):Customer (rhs), priority (rhs. priority)
  {
  	1ogCall("PriorityCustomer copy constructor");
  }
  
  PriorityCustomer&
  PriorityCustomer::operator=(const PriorityCustomer& rhs)
  {
  	1ogCa1l ("PriorityCustomer copy assignment operator") ;
  	Customer::operator=(rhs);  //对base class成分进行赋值动作
  	priority = rhs .priority;
  	return *this;
  }
  ```

​		除此之外，拷贝构造函数和拷贝赋值操作符，他们两个中任意一个不要去调用另一个。虽然这两个函数有近似相同的实现，看似可以避免代码重复，但是是荒谬的。其根本原因在于拷贝构造函数在构造一个对象——这个对象在调用之前并不存在；而赋值操作符在改变一个对象——这个对象是已经构造好了的。因此前者调用后者是在给一个还未构造好的对象赋值；而后者调用前者就像是在构造一个已经存在了的对象.

​		若想消除两个函数的重复代码，应该建立一个新的成员函数（通常为private且常被命名为init）给两者调用

# 三、资源管理

> RAII（Resource Acquisition Is Initialization），也称为“资源获取就是初始化”，是c++等编程语言常用的管理资源、避免内存泄露的方法。它保证在任何情况下，使用对象时先构造对象，最后析构对象。

​		资源使用后必须归还系统。c++系统中最常使用的资源为动态分配内存，其他常见的资源还包括文件描述器(file descriptors)、互斥锁(mutex locks)、图形界面中的字型和笔刷、数据库连接、以及网络sockets。不论哪一种资源， 重要的是，当你不再使用它时，必须将它还给系统。

​		本章正是在考虑异常、函数内多重回传路径、程序员不当维护软件的背景下尝试和资源管理打交道。本章除了介绍基于对象的资源管理办法，也专门对内存管理提出了更深层次的建议。

## 条款13：以对象管理资源

> * 为防止资源泄漏，请使用RAII对象，它们在构造函数中获得资源并在析构函数中释放资源。
> * 常被使用的RAII classes 分别是shared_ptr、unique_ptr和weak_ptr
>
> 注意，std::auto_ptr，deprecated in C++11，removed in C++17。现在为shared_ptr、unique_ptr和weak_ptr

​		本条款的核心观点在于：以面向流程的方式管理资源（的获取和释放），总是会在各种意外出现时，丢失对资源的控制权并造成资源泄露。以面向过程的方式管理资源意味着，资源的获取和释放都分别被封装在函数中。这种管理方式意味着资源的索取者肩负着释放它的责任，但此时我们就要考虑以下几个问题：调用者是否总是会记得释放呢？调用者是否有能力保证合理地释放资源呢？不给调用者过多义务的设计才是一个良好的设计。

首先我们看一下哪些问题会让调用者释放资源的计划付诸东流：

- 一句简单的`delete`语句并不会一定执行，例如一个**过早的`return`语句**（如资源的使用和delete位于某循环内，而该循环由于某个continue或goto语句过早退出）或是**在`delete`语句之前某个语句抛出了异常**。
- 谨慎的编码可能能在这一时刻保证程序不犯错误，但无法保证软件接受**维护**时，其他人在delete语句之前加入的return语句或异常重复第一条错误。

​		为了保证资源的获取和释放一定会合理执行，我们把获取资源和释放资源的任务**封装在一个对象中**。当构造这个对象时资源自动获取，当不需要资源时，让对象的析构函数自动释放那些资源。这便是RAII的想法，因为我们总是在获得一笔资源后于同一语句内初始化某个管理对象。无论控制流如何离开区块，一旦对象被销毁（比如离开对象的作用域）其析构函数会自动被调用。

以对象管理资源的两个关键想法：

* 获得资源后立即放进管理对象内（如返回的资源被当作其管理者的初值`shared_ptr<aa> p(createaa());`

* 管理对象运用机构函数确保资源被释放

具体实践请参考C++11的`shared_ptr<T>`。

## 条款14：在资源管理类中小心coping行为

> * 复制RAII对象必须一并复制它所管理的资源，所以资源的copying行为决定RAII对象的copying行为。
> * 普遍而常见的RAII class copying行为是：抑制copying、施行引用计数法(reference counting)。不过其他行为也都可能被实现。

并非所有资源都是heap-based，因此有时需要**自己建立资源管理类**

```cpp
void lock (Mutex* pm);  // 锁定pm所指的互斥器
void unlock (Mutex* pm);  // 将互斥器解除锁定

class Lock {
public:
    explicit Lock(Mutex* pm) : mutexPtr(pm) {
        lock(mutexPtr);
    }
    ~Lock() { unlock(mutexPtr); }
private:
    Mutex *mutexPtr;
};

// Lock的用法符合RAII方式
Mutex m;  // 定义你需要的互斥器
...
{	// 建立一一个区块用来定义critical section.
Lock ml (&m);  // 锁定互斥器.
...  // 执行crtical section内的操作.
}  // 在区块最末尾，自动解除互斥器锁定.
```

若Lock对象，即我们的RAII对象被复制

```cpp
Lock ml1(&m);  // 锁定m
Lock ml2(ml1);  // 将ml1复制到ml2上
```

我们的做法一般有两种

* 禁止复制。有时对RAII对象进行复制并不合理，可以用=delete禁止
* 对底层资源使用”引用计数法“。有时希望保有资源，直到最后一个使用者被销毁，此时复制RAII对象时应该将资源的被引用数递增。通过shared_ptr实现引用计数coping行为。shared_ptr的缺省行为是”**当引用次数为0时删除其所指物**“，这里可以通过**指定删除器**来改变

```cpp
class Lock {
pub1ic:
	explicit Lock(Mutex* pm)  // 以某个Mutex初始化shared_ptr
	 : mutexPtr(pm, unlock)  // 以unlock函数来代替delete，当引用次数为0时被调用
    {
         lock(mutexPtr.get());  // 返回mutexPtr中保存的指针
	}
private:
	std::shared_ptr<Mutex> mutexPtr;  // 使用shared_ptr
};										   // 替换raw pointer
```

## 条款15：在资源管理类中提供对原始资源的访问

> * APIs往往要求访问原始资源(raw resources)，所以每一个RAIIclass应该提供一个”取得其所管理之资源“的办法。（比如智能指针可以通过get()显式的得到智能指针对象中保存的指针，也可以通过*和->等运算符访问所指对象中的资源）
> * 对原始资源的访问可能经由显式转换或隐式转换。一般而言显式转换比较安全，但隐式转换对客户比较方便。

## 条款16：成对使用new和delete时要采取相同形式

> 如果你在new表达式中使用[]，必须在相应的delete表达式中也使用[]。如果你在new表达式中不使用[]，一定不要在相应的delete表达式中使用[]。

见c++笔记/第十二章/new和delete

## 条款17：以独立语句将newed对象置入智能指针

> 以独立语句将newed对象存储于(置入)智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄漏。

```cpp
// 假设有这么两个函数
int priority();
void processWidget(std::shared_ptr<Widget> pw, int priority);

// 调用processWidget
processWidget(new Widget, priority());  // 错误，shared_ptr的构造函数为explicit的，传入new Widget不会自动转换为shared_ptr<Widget>的形式

// 可以通过编译，但这种调用方式可能造成资源泄露
processWidget (std::shared_ptr<Widget> (new widget), priority());
/*
在调用processWidget之前，编译器必须创建代码，做以下三件事:
■调用priority
■执行"new widget"
■调用shared_ptr构造函数
调用priority的次序是不固定的，若在第二顺位执行：
1.执行"new Widget"
2.调用priority
3.调用shared ptr构造函数
此时若对priority的调用导致异常，在此情况下"new Widget"返回的指针将会遗失，因为它尚未被置入shared_ptr内，后者是我们期盼用来防卫资源泄漏的武器。在对processWidget的调用过程中可能引发资源泄漏，因为在“资源被创建(经由"new widget") ”和“资源被转换为资源管理对象”两个时间点之间有可能发生异常干扰。
*/

// 解决方法：使用分离语句,分别写出(1) 创建widge,(2)将它置入一个智能指针内，然后再把那个智能指针传给processWidget:
std::shared_ptr<Widget> pw(new Widget);  // 在单独语句内以智能指针存储newed所得对象
processWidget(pw, priority());  // 这个调用动作绝不至于造成泄漏。

```

# 四、设计与声明

## 条款18：让接口容易被正确使用，不易被误用

> * 好的接口很容易被正确使用，不容易被误用。你应该在你的所有接口中努力达成这些性质。
> * “ 促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容。
> * “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任。
> * shared_ ptr支持定制型删除器(custom deleter)。这可防范DLL问题，可被用来自动解除互斥锁(mutexes; 见条款14)等等。

本条款告教你如何**帮助你的客户在使用你的接口时避免他们犯错误**。

在设计接口时，我们常常会错误地假设，接口的调用者**拥有某些必要的知识来规避一些常识性的错误**。但事实上，接口的调用者并不总是像正在设计接口的我们一样“聪明”或者知道接口实现的”内幕信息“，结果就是，我们错误的假设使接口表现得不稳定。这些不稳定因素可能是由于调用者缺乏某些先验知识，也有可能仅仅是代码上的粗心错误。一个合理的接口，应该尽可能的从**语法层面**并在**编译之时运行之前**，帮助接口的调用者规避可能的风险。

- 使用**外覆类型（wrapper）**提醒调用者传参错误检查，将参数的附加条件限制在**类型本身**

当调用者试图传入数字“13”来表达一个“月份”的时候，你可以在函数内部做运行期的检查，然后提出报警或一个异常，但这样的做法更像是一种责任转嫁——调用者只有在尝试过后才发现自己手残把“12”写成了“13”。如果在设计参数类型时就把“月份”这一类型抽象出来，比如使用enum class（强枚举类型），就能帮助客户在**编译**时期就发现问题，把参数的附加条件限制在类型本身，可以让接口更易用。

另一个例子：

![image-20221115121010017](E:\MarkDown\picture\image-20221115121010017.png)

![image-20221115121045682](E:\MarkDown\picture\image-20221115121045682.png)

再对月份进行限制

![image-20221115121222346](E:\MarkDown\picture\image-20221115121222346.png)

- 从**语法层面**限制调用者**不能做的事**

接口的调用者往往无意甚至没有意识到自己犯了个错误，所以接口的设计者必须在语法层面做出限制。一个比较常见的限制是加上`const`，比如在`operate*`的返回类型上加上`const`修饰，可以防止无意错误的赋值`if (a * b = c)`。

- 接口应表现出与内置类型的一致性

让自己的类型和内置类型的一致性，比如自定义容器的接口在命名上和STL应具备一致性，可以有效防止调用者犯错误。或者你有两个对象相乘的需求，那么你最好重载`operator*`而并非设计名为”multiply”的成员函数。

- 从语法层面限制调用者**必须做的事**

**别让接口的调用者总是记得做某些事情**，接口的设计者应在假定他们**总是忘记**这些条条框框的前提下设计接口。比如用智能指针代替原生指针就是为调用者着想的好例子，为了防止客户忘记使用智能指针，我们可以令factory函数返回一个智能指针`shared_ptr<aa> factory();`。如果一个核心方法需要在使用前后设置和恢复环境（比如获取锁和归还锁），更好的做法是将设置和恢复环境设置成纯虚函数并要求调用者继承该抽象类，强制他们去实现。在核心方法前后对设置和恢复环境的调用，则应由接口设计者操心。

当方法的调用者（我们的客户）责任越少，他们可能犯的错误也就越少



## 条款19：设计class犹如设计type

> Class的设计就是type的设计。在定义一个新type之前，请确定你已经考虑过本条款覆盖的所有讨论主题。

本条款提醒我们设计class需要注意的细节，但并没有给每一个细节提出解决方案，只是提醒而已。每次设计class时最好在脑中过一遍以下问题：

- 对象该如何创建销毁：包括构造函数、析构函数以及new和delete操作符的重构需求。
- 对象的构造函数与赋值行为应有何区别：构造函数和赋值操作符的区别，重点在资源管理上。
- 对象被拷贝时应考虑的行为：拷贝构造函数。
- 对象的合法值是什么？最好在语法层面、至少在编译前应对用户做出监督。
- 新的类型是否应该配合某个继承体系，这就包含虚函数的覆盖问题。
- 新类型和已有类型之间的隐式转换问题，这意味着类型转换函数和非explicit函数之间的取舍。
- 那些操作符和函数该是成员函数
- 新类型是否需要重载操作符。
- 什么样的接口应当暴露在外，而什么样的技术应当封装在内（public和private）
- 新类型的效率、资源获取归还、线程安全性和异常安全性如何保证。
- 这个类是否具备template的潜质，如果有的话，就应改为模板类。



## 条款20：宁以pass-by-reference-to-const替换pass-by-value

> * 尽量以pass-by-reference-to-const替换pass-by-value。前者通常比较高效，并可避免切割问题( slicing problem)
> * 以上规则并不适用于内置类型，以及STL的迭代器和函数对象。对它们而言，pass-by-value往往比较适当。

函数接口应该以`const`引用的形式传参，而不应该是按值传参，否则可能会有以下问题：

- 按值传参涉及大量参数的复制，这些副本大多是没有必要的。
- 如果拷贝构造函数设计的是深拷贝而非浅拷贝，那么拷贝的成本将远远大于拷贝某几个指针。
- 对于多态而言，将父类设计成按值传参，如果传入的是子类对象，仅会对子类对象的父类部分进行拷贝，即部分拷贝，而所有属于子类的特性将被丢弃，造成不可预知的错误，同时虚函数也不会被调用。
- 小的类型并不意味着按值传参的成本就会小。首先，类型的大小与编译器的类型和版本有很大关系，某些类型在特定编译器上编译结果会比其他编译器大得多。小的类型也无法保证在日后代码复用和重构之后，其类型始终很小。

尽管如此，面对内置类型和STL的迭代器与函数对象，我们通常还是会选择按值传参的方式设计接口。

详见c++笔记/黑马/值传递和地址传递

## 条款21：必须返回对象时，别妄想返回其reference

> 绝不要返回pointer或reference指向一个local stack（栈）对象，或返回reference指向一个heap-allocated对象，或返回pointer或reference指向一个local static 对象而有可能同时需要多个这样的对象。**对于一个“必须返回新对象”的函数的正确写法是：就让那个函数返回一个新对象object，而不是返回一个reference**。虽然这会造成构造和析构函数的额外调用，但在C++11之后，我们可以采用给类型编写“转移构造函数”以及使用`std::move()`函数更加优雅地**消除由于拷贝造成的时间和空间的浪费。**

局部对象local stack在函数退出前就会被销毁，返回指向局部对象是一个未定义行为

对于heap-allocated：在heap内new一个对象，然后返回，会付出一个“构造函数调用”的代价，并且容易造成无法析构的情况，导致资源泄露
![image-20221115213435652](E:\MarkDown\picture\image-20221115213435652.png)

对于local static：如让operator*返回的reference指向一个被定义于函数内部的static Rational(有理数)对象，这不仅可能存在多线程安全性问题，且存在可能需要多个这样的对象的问题

![image-20221115214212677](E:\MarkDown\picture\image-20221115214212677.png)



因此**对于一个“必须返回新对象”的函数的正确写法是：就让那个函数返回一个新对象**。虽然会要承受这个返回值的构造成本和析构成本，但编译器在某些情况下也会进行优化。
![image-20221115214506953](E:\MarkDown\picture\image-20221115214506953.png)

而在C++11之后，我们可以采用给类型编写“转移构造函数”以及使用`std::move()`函数更加优雅地**消除由于拷贝造成的时间和空间的浪费。**



## 条款22：将成员变量声明为private

> * 切记将成员变量声明为private。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供class作者以充分的实现弹性。
> * protected并不比public更具封装性。

先说结论——**请对class内所有成员变量声明为`private`，`private`意味着对变量的封装。**但本条款提供的更有价值的信息在于不同的属性控制——`public`,` private`和`protected`——**代表的设计思想**。

简单的来说，把所有成员变量声明为private的好处有两点。首先，将所有的变量都设为private了，那么所有的public和protected成员都是函数了，用户在使用的时候也就无需区分，在访问class成员时就不用再去想是否该使用小括号（调用函数），这就是语法一致性；其次，对变量的封装意味着，**可以尽量减小因类型内部改变造成的类外代码的必要改动。**

一旦所有变量都被封装了起来，外部无法直接获取，那么所有类的使用者（我们称为客户，客户也可能是未来的自己，也可能是别人）想利用私有变量实现自己的业务功能时，就**必须通过我们留出的接口**，这样的接口便充当了一层缓冲，将类型内部的升级和改动尽可能的对客户不可见——**不可见就是不会产生影响**，不会产生影响就不会要求客户更改类外的代码。因此，一个设计良好的类在内部产生改动后，对整个项目的影响只应是**需要重新编辑而无需改动类外部的代码**。

我们接着说明，**`public`和`protected`属性在一定程度上是等价的**。成员变量的封装性与“成员变量的内容改变时可能造成的代码破坏量”成反比。一个自定义类型被设计出来就是供客户使用的，那么客户的使用方法无非是两种——**用这个类创建对象**或者**继承这个类以设计新的类**——以下简称为第一类客户和第二类客户。那么从封装的角度来说，一个`public`的成员说明了**类的作者决定对类的第一种客户不封装此成员**，而一个`protected`的成员说明了**类的作者对类的第二种客户不封装此成员**。也就是说，当我们把类的两种客户一视同仁了以后，`public`、`protected`和`private`三者反应的即类设计者对类成员封装特性的不同思路——对成员封装还是不封装，如果不封装是对第一类客户不封装还是对第二类客户不封装。

## 条款23：宁以non-member, non-friend替换member函数

> 宁可拿non-member、non-friend函数替换member函数（不直接访问private成员而是通过若干public函数集成而来）。这样做可以增加封装性、包裹弹性(packaging flexibility)和机能扩充性。

​	在一个类里，我愿把**需要直接访问private成员的public和protected成员函数**称为**功能颗粒度较低的函数**，原因很简单，他们涉及到对private成员的直接访问，说明他们处于封装表面的第一道防线。由若干其他public（或protected）函数集成而来的public成员函数，我愿称之为**颗粒度高的函数**，因为他们集成了若干颗粒度较低的任务，这就是本条款所针对的对象——那些**无需直接访问private成员**，而只是**若干public函数集成而来的member函数**。本条款告诉我们：这些函数应该尽可能放到类外。

```cpp
// member函数
class WebBrowser {							//	一个浏览器类
public:
  	void clearCache();					// 清理缓存，直接接触私有成员
  	void clearHistory();				// 清理历史记录，直接接触私有成员
  	void clearCookies();				// 清理cookies，直接接触私有成员
  
  	void clear();								// 颗粒度较高的函数，在内部调用上边三个函数，不直接接触私有成员，本条款告诉我们这样的函数应该移至类外
}

// 也可以由non-member函数调用
void clear(WebBrowser& wb) {
    wb.clearCache();
    wb.clearHistory();
    wb.clearCookies();
}
```

如果高颗粒度函数设置为类内的成员函数，那么一方面他会破坏类的封装性，另一方面降低了函数的包裹弹性。

1. 类的封装性

封装的作用是尽可能减小被封装成员的改变对类外代码的影响——我们希望类内的改变只影响有限的客户。一个量化某成员封装性好坏的简单方法是：看类内有多少（public或protected）函数直接访问到了这个成员，这样的函数越多，该成员的封装性就越差——该成员的改动对类外代码的影响就可能越大。回到我们的问题，高颗粒度函数在设计之时，设计者的本意就是**它不应直接访问任何私有成员**，而只是公有成员的简单集成，这样会最大程度维护封装性，但很可惜，**这样的愿望并没有在代码层面体现出来**。这个类未来的维护者（有可能是未来的你或别人）很可能忘记了这样的原始设定，而在此本应成为“高颗粒度”函数上大肆添加对私有成员的直接访问，这也就是为什么封装性可能会被间接损坏了。member函数带来的封装性比non-member函数要低。但设计为非成员函数就从语法上避免了这种可能性。

2. 函数的包裹弹性与设计方法

将高颗粒度函数提取至类外部可以允许我们从**更多维度组织代码结构**，并**优化编译依赖关系**。我们用上边的例子说明什么是“更多维度”。`clear（）`函数是代码的设计者最初从浏览器的角度对低颗粒度函数做出的集成，但是如果从“cache”、“history”、和“cookies”的角度，我们又能够做出其他的集成。比如将“搜索历史记录”和“清理历史记录”集成为“定向清理历史记录”函数，将“导出缓存”和“清理缓存”集成为“导出并清理缓存”函数，这时，我们在浏览器类外做这样的集成会有更大的自由度。通常利用一些工具类如`class CacheUtils`、`class HistoryUtils`中的static函数来实现；又或者采用不同namespace来明确责任，将不同的高颗粒度函数和浏览器类纳入不同namespace和头文件，当我们使用不同功能时就可以include不同的头文件，而不用在面对cache的需求时不可避免的将cookies的工具函数包含进来，降低编译依存性。这也是`namespace`可以跨文件带来的好处。

```cpp
// 头文件 webbrowser.h		针对class WebBrowserStuff自身
namespace WebBrowserStuff {
class WebBrowser { ... };		//核心机能
}

// 头文件 webbrowsercookies.h		针对WebBrowser和cookies相关的功能
namespace WebBrowserStuff {
	...												//与cookies相关的工具函数
}

// 头文件 webbrowsercache.h		针对WebBrowser和cache相关的功能、
namespace WebBrowserStuff {
	...												//与cache相关的工具函数
}
```

最后要说的是，本条款讨论的是那些**不直接接触私有成员的函数**，如果你的public(或protected)函数必须直接访问私有成员，那请忘掉这个条款，因为把那个函数移到类外所需做的工作就比上述情况远大得多了。

friend函数对class private成员的访问权力和member函数相同，因此对封装的冲击力度也相同

## 条款24：若所有参数皆需类型转换，请为此采用non-member函数

> 如果你需要为某个函数的所有参数(包括被this指针所指的那个隐喻参数)进行类型转换，那么这个函数必须是个non-member

这个条款告诉了我们**操作符重载被重载为成员函数和非成员函数的区别**。作者想给我们提个醒，如果我们在使用操作符时**希望操作符的任意操作数都可能发生隐式类型转换**，**那么应该把该操作符重载成非成员函数。**

我们首先说明：如果一个操作符是成员函数，那么它的**第一个操作数（即调用对象）不会发生隐式类型转换**。因为只有当参数被列于参数列内，这个参数才是隐式类型转换的合格参与者。

首先简单讲解一下当操作符被重载成员函数时，第一个操作数特殊的身份。操作符一旦被设计为成员函数，它在被使用时的特殊性就显现出来了——单从表达式你无法直接看出**是类的哪个对象在调用这个操作符函数**，不是吗？例如下方的有理数类重载的操作符”+”，当我们在调用`Rational z = x + y;`时，调用操作符函数的对象并没有直接显示在代码中——这个操作符的`this`指针指向`x`还是`y`呢？

```cpp
class Rational {
public:
  //...
  Rational operator+(const Rational rhs) const; 
private:
  //...
}
```

作为成员函数的操作符的第一个隐形参数”`this`指针”总是指向第一个操作数，所以上边的调用也可以写成`Rational z = x.operator+(y);`，这就是操作符的**更像函数的调用方法**。那么，做为成员函数的操作符默认操作符的第一个操作数应当是正确的类对象——**编译器正式根据第一个操作数的类型来确定被调用的操作符到底属于哪一个类的**。因而第一个操作数是不会发生隐式类型转换的，第一个操作数是什么类型，它就调用那个类型对应的操作符。

我们举例说明：当`Ratinoal`类的构造函数允许`int`类型隐式转换为`Rational`类型时，`Rational z = x + 2;`是可以通过编译的，因为操作符是被`Rational`类型的`x`调用，同时将`2`隐式转换为`Ratinoal`类型，完成加法。但是`Rational z = 2 + x;`却会引发编译器报错，因为由于操作符的第一个操作数不会发生隐式类型转换，所以加号“+”实际上调用的是`2`——一个`int`类型的操作符，因此编译器会试图将`Rational`类型的`x`转为`int`，这样是行不通的。

因此在你编写诸如加减乘除之类的（但不限于这些）操作符、并假定允许每一个操作数都发生隐式类型转换时，**请不要把操作符函数重载为成员函数**。因为当第一个操作数不是正确类型时，可能会引发调用的失败。解决方案是，**请将操作符声明为类外的非成员函数**，你可以选择友元让操作符内的运算更便于进行，也可以为私有成员封装更多接口来保证操作符的实现，这都取决于你的选择。



题外话：如果你想禁止隐式类型转换的发生，请把你每一个单参数构造函数后加上关键字`explicit`。



## 条款25：考虑写出一个不抛出异常的swap函数

> * 当std::swap对你的类型效率不高时，提供一个swap成员函数，并确定这个函数不抛出异常。
> * 如果你提供一个member swap，也该提供一个non-member swap用来调用前者。对于classes(而非templates)，也请特化std: :swap。
> * 调用swap时应针对std: :swap使用using声明式，然后调用swap并且不带任何“命名空间资格修饰”。
> * 为“用户定义类型”进行std templates全特化是好的，但千万不要尝试在std内加入某些对std而言全新的东西。



对于标准库提供的swap算法，只要类型T支持copying（通过copy构造函数和copy assignment操作符完成），缺省的swap即可实现swap功能

```cpp
namespace std {
    template<typename T>
    void swap(T& a, T& b) {
        T temp(a);
        a = b;
        b = temp;
    }
}
```



而对于“指针指向一个对象，内含真正数据”的类型，缺省的swap不止复制三个Widget，还会复制三个WidgetImpl对象

1、在class内提供一个non-member swap，令其调用swap成员函数

```cpp
class WidgetImpl {
public:
    ...
private:
	int a, b, c;
	std::vector<double> v;  // 可能有很多数据，意味着复制时间会很长
    ...
};

class Widget {  //与前同，唯一差别是增加swap函数
public:
	void swap (Widget& other)
	{
		using std::swap;  // pImpl成员为private，需要声明后swap函数才能正常访问
		swap(pImpl, other.pImpl);  //若要置换Widgets就置换其plmpl指针。
	}
private:
    WidgetImpl* pImpl;
};
namespace std {
    // 通常不被允许改变std命名空间内的任何东西，不可以增加新的templates，但可以为标准templates制造特化版本
	template<>  //修订后的std::swap特化版本
	void swap<Widget>(Widget& a, Widget& b)
    {
		a.swap(b);  // 若要置换Widgets,调用其swap成员函数
	}
}
```



2、在命名空间widgetStuff内声明一个non-member swap，让它调用member swap

同时也要在std命名空间内写一个std::swap特化版本，因为使用者有时会用std修饰swap调用式。

```cpp
namespace widgetStuff {
	...  // 模板化的WidgetImpl等等。
	template<typename T>  // 同前，内含swap成员函数。
	class Widget { ... };
    ...
	template<typename T>  // non-member swap函数;
	void swap(Widget<T>& a, Widget<T>& b)  //这里并不属于std命名空间。
	{
		a.swap(b);
	}
}
```

还要注意，成员版swap绝不可抛出异常，因为swap的一个应用就是帮助class提供异常安全性。对于非成员版如swap缺省版本是以copy构造函数和copy assignment操作符为基础，而一般情况下两者都允许抛出异常。因此自定义版本的swap所提供的不仅是高效置换对象值的方法，而且不抛出异常

# 五、实现



## 条款26:尽可能延后变量定义式的出现时间

> 尽可能延后变量定义式（变量的类型带有一个构造函数或析构函数）的出现，至能给其初值实参为止。这样做不仅能够避免构造(和析构)非必要对象，还可以避免无意义的default 构造行为，可增加程序的清晰度并改善程序效率。



```cpp
// 这个函数过早定义变量"encrypted"
std::string encryptPassword(const std::string& password)
{
	using namespace std;
	// string encrypted;定义在这，在有异常抛出时就是未被使用的，此时多花费了构造和析构成本
	if (password.length() < MinimumPasswordLength) {
		throw logic_error("Password is too short");
	}
    // string encrypted;放在这，但仍有问题。虽定义但无初值，调用的是default构造函数，效率差
    
    // 正确做法，通过copy构造函数直接赋值
    string encrypted(password);
	encrypt(encrypted);  // 必要动作，将加密后的密码置入变量encrypted内
	return encrypted;
}
```



对于循环

```cpp
//方法A:定义于循环外
Widget w;
for (int i = 0; i < n; ++i) {
	w=取决于i的某个值;
}
//方法B:定义于循环内
for (int i = 0; i < n; ++i) {
	Widget w(取决于i的某个值);
}
```

* 做法A：1个构造函数 + 1个析构函数 + n个赋值操作
* 做法B：n个构造函数 + n个析构函数

​	如果classes的一个赋值成本低于一组构造+析构成本，做法A大体而言比较高效。尤其当n值很大的时候。否则做法B或许较好。此外做法A造成名称w的作用域(覆盖整个循环)比做法B更大，有时那对程序的可理解性和易维护性造成冲突。因此除非
​	(1) 你知道赋值成本比“构造+析构”成本低
​	(2) 你正在处理代码中效率高度敏感( performance -sensitive)的部分
​	否则你应该使用做法B.



## 条款27：尽量少做转型动作

> * 如果可以，尽量避免转型，特别是在注重效率的代码中避免dynamic_casts。如果有个设计需要转型动作，试着发展无需转型的替代设计。
> * 如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需将转型放进他们自己的代码内。
> * 宁可使用C++-style（新式）转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分门别类的职掌。

**1、如果可以，尽量避免转型，特别是在注重效率的代码中避免dynamic_casts。如果有个设计需要转型动作，试着发展无需转型的替代设计**



**1.1 转型可能会引发一些问题**

```cpp
class Widget {
public:
	explicit widget(int size);
};

void doSomeWork (const Widget& w);
// 以一个int加上“函数风格”的转型动作创建一个widget
doSomeWork(Widget (15));
// 以一个int加上“C++风格”的转型动作创建一个Widget
doSomeWork (static_cast<Widget>(15));
```

​	对于单一对象，可能拥有一个以上的地址（如基类指针和派生类指针都指向一个派生类对象，此时对于基类指针，是在派生类指针上加了个偏移量（注意，对象的布局方式和其地址计算方式随编译器的不同而不同））

```cpp
class Window {  // base class
public:
	virtual void onResize() { ... }  // base onResize实现代码
    ...
};

class SpecialWindow: public Window {  // derived class
public:
	virtual void onResize() {  // derived onResize实现代码
		static_cast<Window>(*this).onResize();  // 将*this 转型为Window,然后调用其onResize;这不可行!
		...  // 这里进行SpecialWindow专属行为。
	}
};
```

​	这段程序将 * this转型为Window，对函数onResize的调用也因此调用了Window: :onResize。但它调用的并不是当前对象上的函数，而是稍早转型动作所建立的一个 “ *this对象之base class成分 ” 的**临时副本**身上的onResize。上述代码是在“当前对象之base class成分”的副本上调用Window::onResize，然后在当前对象身上执行SpecialWindow专属动作。如果window: :onResize修改了对象内容，当前对象其实没被改动，改动的是副本。然而SpecialWindow: :onResize内如果也修改对象，当前对象真的会被改动。这使当前对象进入一种“伤残”状态：其base class 成分的更改没有落实，而derived class成分的更改倒是落实了。

正确写法

```cpp
class SpecialWindow: public Window {
public:
	virtual void onResize() {
        // 调用Window::onResize作用于*this身上
        // 而不是哄骗编译器将*this视为一个base class对象
		window::onResize();
        ...
	}
...
};
```

**1.2 效率问题**

​	dynamic_cast的许多实现版本执行速度相当慢。例如至少有一个很普遍的实现版本基于“class名称之字符串比较”，如果你在四层深的单继承体系内的某个对象身上执行dynamic_cast，刚才说的那个实现版本所提供的每一次dynamic_cast可能会耗用多达四次的strcmp调用，用以比较class名称。深度继承或多重继承的成本更高。某些实现版本这样做有其原因(它们必须支持动态连接)。但应该在注重效率的代码中对dynamic_casts 保持机敏与猜疑。





之所以需要dynamic_ cast, 通常是因为你想在一个你认定为derived class对象身上执行derived class操作函数，但你的手上却只有一个“指向base"的pointer或reference，你只能靠它们来处理对象`dynamic_cast<派生类*>(基类)`。有两个一般性做法可以避免这个问题。

1.直接存储指向派生类对象的指针

![image-20221121150615989](E:\MarkDown\picture\image-20221121150615989.png)

2.通过调用虚函数

![image-20221121151616526](E:\MarkDown\picture\image-20221121151616526.png)

![image-20221121151628596](E:\MarkDown\picture\image-20221121151628596.png)

***

**3、宁可使用C++-style（新式）转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分门别类的职掌**

**旧式转型**

C风格：(T) expression

函数风格：T(expression)

**新式转型**

![image-20221121115620095](E:\MarkDown\picture\image-20221121115620095.png)



## 条款28：避免返回handles指向对象内部成分

> 避免返回handles（包括references、指针、迭代器）指向对象内部。遵守这个条款可增加封装性，帮助const成员函数的行为像个const，并将发生“虚吊号码牌”(dangling handles)，即指向一个不存在的对象的可能性降至最低。

如一个const成员函数传出一个reference指向一个类，这个类的内容又和对象自身有关联，则函数的调用者就可以修改这个数据。因此，返回一个”代表对象内部数据“的handle，会降低对象的封装性，且造成”虽然调用const成员函数却令对象状态被更改“。

因此，**绝对不该令成员函数返回一个指针指向private或protected的成员函数/成员变量，因为这样后者的访问级别也会跟着修改**

就算加了const，有些场景下也不行，如

![image-20221206182601082](E:\MarkDown\picture\image-20221206182601082.png)



这并不意味你绝对不可以让成员函数返回handle。有时候你必须那么做。例如operator[]就允许你“摘采”strings和vectors的个别元素，而这些operator[]就是返回references 指向“容器内的数据”(见条款3)，那些数据会随着容器被销毁而销毁。尽管如此，这样的函数毕竟是例外，不是常态。

## 条款29：为“异常安全”而努力是值得的

> * 异常安全函数(Exception-safefunctions)即使发生异常也不会泄漏资源或允许任何数据结构败坏。这样的函数区分为三种可能的保证：基本型、强烈型、不抛异常型。
> * 基本承诺：即使抛出异常，也不会存在对象或数据结构败坏的情况。但程序的现实状态是难以预知的，比如在更改背景图的例子中我们编写程序使得即使抛出异常，也能保存原背景图，但我们无法得知背景图的情况。需要调用某个成员函数以得知现状。
> * 强烈保证：程序只有两种可能：若函数成功，就是完全成功，若函数失败，程序会回复到调用函数前的状态（即若异常被抛出，程序状态不改变）。往往以copy-and-swap 实现出来，但“强烈保证”并非对所有函数都可实现或具备现实意义。
> * 不抛掷(nothrow)保证：承诺绝不抛出异常，总能够完成功能。如int、指针等内置类型的所有操作都提供Nothrow保证
> * 函数提供的“异常安全保证”通常最高只等于其所调用之各个函数的“异常安全保证”中的最弱者。

当异常被抛出时，带有异常安全性的函数会：

* 不泄露任何资源
* 不允许数据败坏

```cpp
// 错误例子
class PrettyMenu {
public:
	void changeBackground(std::istream& imgSrc);  // 改变背景图像
private:
	Mutex mutex;  // 互斥器
	Image* bgImage;  // 目前的背景图像
	int imageChanges;  // 背景图像被改变的次数
};
void PrettyMenu::changeBackground(std::istream& imgSrc)
{
	lock (&mutex);  // 取得互斥器(见条款14)
	delete bgImage;  // 摆脱旧的背景图像
	++imageChanges;  // 修改图像变更次数
	bgImage = new Image(imgSrc);  // 安装新的背景图像
	unlock(&mutex);  //释放互斥器
}
```

不泄漏任何资源：上述代码没有做到这一点，因为一旦"new Image (imgSrc)"导致异常，对unlock的调用就绝不会执行，于是互斥器就永远被把持住了。
不允许数据败坏：如果"new Image (imgSrc)"抛出异常，bgImage 就是指向一个已被删除的对象，imageChanges也已被累加，而其实并没有新的图像被成功安装起来。

对于资源泄漏：

```cpp
// 通过资源管理类解决资源泄漏
void PrettyMenu::changeBackground(std::istream& imgSrc)
{
    Lock ml(&mutex);  // 来自条款14:获得互斥器并确保它稍后被释放。作为资源管理类，自动释放。并且可以使得函数更短
	delete bgImage;
	++imageChanges; 
	bgImage = new Image(imgSrc);
}
```

对于数据败坏：

```cpp
class PrettyMenu {
	...
	std::shared_ptr<Image> bgImage;
	...
};

void PrettyMenu::changeBackground(std::istream& imgSrc)
{
    Lock ml(&mutex);
    // 以new的执行结果设定bgImage这个智能指针
    // 这样删除动作只发生在新图像被创建之后
    bgImage.reset(new Image(imgSrc));
	++imageChanges; 
}
```



































































































































