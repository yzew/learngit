## JetBrains Rider

一般来说编写C#程序是使用Visual Studio，但公司由于去A化，所以..

双击shift，全局搜索文件，类，函数，字段等啥都可以！

ctrl+shift+f 全局搜索，这个一般用于字符串查找

## c#

* 与C++相比，C#不支持多重继承（**当 B 类从 A 类派生，C 类从 B 类派生**），只能单一继承；
* C#是面向对象的编程语言；基于组件和面向对象概念
* 内存管理被自动处理
* 指针只能在不安全模式（？）下使用
* C#的接口与单一继承，承载着处理逻辑；
* C#编译成中间代码，跨平台运行；这一点，与Java类似；
* C# 编程可用于创建控制台应用程序，Windows应用程序，移动应用程序等。Windows之外很少使用。

**访问修饰符**如下所示：

* public：所有对象都可以访问；
* internal：同一个程序集的对象可以访问（类的默认访问标识符）
* protected：只有该类对象及其子类对象可以访问
* private：对象本身在对象内部可以访问；（成员的默认访问标识符）
* protected internal：访问限于当前程序集或派生自包含类的类型。

**参数传递**

```
值参数、引用参数（形参和输入实参都需要用ref关键字声明）、输出参数（可以返回多个值。out关键字声明
```

数组作为形参时，可以用int[] arr，也可以用params关键字进行修饰，表示将不定参数编译成一个数组提供给coder使用，调用的时候传入的就可以是（1， 5， 93），会被当做一个数组

Array类提供了对数组的一系列方法，可以通过Array.xxx(数组)调用

c#中类和结构有以下几个基本的不同点：

* 结构是值类型，它在栈中分配空间；而类是引用类型，它在堆中分配空间，栈中保存的只是引用。
* 结构不支持继承。
* 结构不能声明默认的构造函数
* 类的对象是存储在堆空间中，结构存储在栈中。堆空间大，但访问速度较慢，栈空间小，访问速度相对更快。故而，当我们描述一个轻量级对象的时候，结构可提高效率，成本更低。当然，这也得从需求出发，假如我们在传值的时候希望传递的是对象的引用地址而不是对象的拷贝，就应该使用类了。

**属性**

 属性将声明信息与C#代码相关联，一旦属性与程序实体关联，即可使用名为反射的技术对属性进行查询。

 属性结合了字段和方法的多个方面。对于对象的用户，属性显示为**字段**，访问该属性需要完全相同的语法。对于类的实现者，属性是一个或两个代码块，表示一个 **get** 访问器和 (或) 一个 set 访问器。
​ 当读取属性时，执行 get 访问器的代码块。当向属性分配一个新值时，执行 set 访问器的代码块。不具有 set 访问器的属性被视为只读属性，不具有 get 访问器的属性被视为只写属性，同时具有这两个访问器的属性为可读可写属性。

 属性可向程序中添加元数据。元数据是嵌入程序中的信息，如编译器指令或数据描述。程序可以使用反射检查自己的元数据

```
class aa
{
    private string id = "";
	public string ID
    {
        get
        {
            return id;
        }
        set
        {
            id = value;
        }
    }
}
class program
{
    static void Main()
    {
        aa a = new aa();
        a.ID = "BH001";
        Console.WriteLine(a.ID +"  +myclass.Name);
    }
}
```

**多态：**

多态是同一个行为具有多个不同表现形式或形态的能力。同样分静态多态（函数重载和运算符重载）和动态多态（通过 **抽象类** 和 **虚方法** 实现)。在**静态多态性**中，函数的响应是在编译时发生的。在**动态多态性**中，函数的响应是在运行时发生的。

在 C# 中，每个类型都是多态的，因为包括用户定义类型在内的所有类型都继承自 Object。

**抽象类**

使用关键字 **abstract** 创建抽象类，用于提供接口的部分类的实现。**抽象类**包含抽象方法(函数前加abstract)，抽象方法不能有方法体，需要被派生类实现（派生类的重载函数前要加 **override**）

* 不能创建一个抽象类的实例。

* 不能在一个抽象类外部声明一个抽象方法。

* 通过在类定义前面放置关键字 **sealed**，可以将类声明为**密封类**。当一个类被声明为 **sealed** 时，它不能被继承。抽象类不能被声明为 sealed。成员函数也可以声明为 **sealed** ，不能被重写，但要与override一起使用

  ```
  class A {} // 普通类
  sealed class B : A {}  // B不能被继承
  
  // 抽象类
  abstract class Shape
  {
     abstract public int area();
  }
  class Rectangle:  Shape
  {
    private int length;
    private int width;
    public Rectangle( int a=0, int b=0)
    {
       length = a;
       width = b;
    }
    public override int area ()
    {
       Console.WriteLine("Rectangle 类的面积：");
       return (width * length);
    }
  }
  ```

**虚方法**

用 virtual 关键字声明。可以在不同的继承类中有不同的实现。

与抽象方法的区别在于，虚方法有方法体，派生类可以重写也可以不重写；抽象方法只在抽象类中定义，派生类必须重写抽象类的抽象方法

* 虚方法仅适用于有继承关系的类对象，所以只有类的成员函数才能说明为虚函数
* 静态成员函数，内联函数，构造函数不能是虚函数
* 析构函数可以是虚函数

```
public class Shape
{
    // 虚方法
    public virtual void Draw()
    {
        Console.WriteLine("执行基类的画图任务");
    }
}

class Circle : Shape
{
    public override void Draw()
    {
        Console.WriteLine("画一个圆形");
        base.Draw();
    }
}
class Rectangle : Shape
{
    public override void Draw()
    {
        Console.WriteLine("画一个长方形");
        base.Draw();
    }
}


class X
{
    protected virtual void F() { Console.WriteLine("X.F"); }
    protected virtual void F2() { Console.WriteLine("X.F2"); }
}

class Y : X
{
    // sealed + override
    sealed protected override void F() { Console.WriteLine("Y.F"); }
    protected override void F2() { Console.WriteLine("Y.F2"); }
}

class Z : Y
{
    // Attempting to override F causes compiler error CS0239.
    // protected override void F() { Console.WriteLine("Z.F"); }

    // Overriding F2 is allowed.
    protected override void F2() { Console.WriteLine("Z.F2"); }
}
```

**接口**

> C#从入门到精通——P152
>
>  由于 C# 中的类不支持多重继承，但是客观世界出现多重继承的情况又比较多，因此为了避免传统的多重继承给程序带来的复杂性等问题, 同时保证多重继承带给程序员的诸多好处, 提出了接口概念。通过接口可以实现多重继承的功能。

接口定义了所有类继承接口时应遵循的语法合同。接口定义了语法合同 **"是什么"** 部分，派生类定义了语法合同 **"怎么做"** 部分。

接口只包含了成员的声明。成员的**定义**是**派生类**的责任。

通常接口命令以 **I** 字母开头

接口可以包含方法、属性、索引器和事件作为成员，但只能定义不能赋值

一个类或结构可以继承多个接口，然后依次实现接口中的方法

```
using System;

interface IMyInterface
{
    // 接口成员
    void MethodToImplement();
}

class InterfaceImplementer : IMyInterface
{
    static void Main()
    {
        InterfaceImplementer iImp = new InterfaceImplementer();
        iImp.MethodToImplement();
    }

    // 接口的实现
    public void MethodToImplement()
    {
        Console.WriteLine("MethodToImplement() called.");
    }
}
```

如果一个接口继承其他接口，那么实现类或结构就需要实现所有接口的成员。

```
using System;

interface IParentInterface
{
    void ParentInterfaceMethod();
}

interface IMyInterface : IParentInterface
{
    void MethodToImplement();
}

class InterfaceImplementer : IMyInterface
{
    static void Main()
    {
        InterfaceImplementer iImp = new InterfaceImplementer();
        iImp.MethodToImplement();
        iImp.ParentInterfaceMethod();
    }

    public void MethodToImplement()
    {
        Console.WriteLine("MethodToImplement() called.");
    }

    public void ParentInterfaceMethod()
    {
        Console.WriteLine("ParentInterfaceMethod() called.");
    }
}
```

## CodeHub

[CodeHub常用操作指导](http://3ms.huawei.com/km/groups/3949280/blogs/details/13341925)

注意，在进行Git Clone操作的时候，勾选“加载Putty秘钥”，选择之前生成的私钥（ppk文件，保存在C:\Users\Administrator.ssh\TortoiseGit）

## 项目构建

**类库**定义的是可以由应用程序调用的类型和方法。定义一些功能函数（DLL类库），编写完并构建后，会生成.dll

在其他程序中，如**控制台应用程序**，我们就可以在左边的依赖项中右键进行引用，添加这个dll，就能在程序中using、创建类对象、调用类方法。此时进行运行，就会调用这个方法。清理一下解决方案然后构建，就会生成.exe文件。

区别就是控制台应用程序会using system，定义main函数进行一些控制台的调用。

## 软件

> codehub代码平台 **已安装** [常用操作指导](http://3ms.huawei.com/km/groups/3949280/blogs/details/13341925)
>
> [skyladder天梯平台源码](https://codehub-y.huawei.com/groups/Automation_Group/Skyladder/detail)，包括私有库、运行包、功能库（已clone，D:\skyladder_addin）

tcp协议与VM通信，

2、需要将自研视觉的这些接口，通过tcp协议执行里面的工具；
3、自己选输出参考点；
4、这个识别功能，没有训练工具。

运行：
启动 D:\HWVision2\TestVision.exe 启动自研视觉软件
启动 D:\HWVision2\NetAssist（网口支持本地IP调试）.exe 进行通信指令测试 发送： 任务1， 任务2，任务3或任务4

这个程序用了很多using，如using AsyncTcp异步tcp;等

程序集位置：

C:\Users\Administrator\AppData\Roaming\JetBrains\Rider2023.1\resharper-host\DecompilerCache\decompiler\8129a077dec44a64b5f6d8e15fb178de4000\54\58b6d19d

HwvCommon、HwvFramework这些程序集是否有使用说明？

比如点一个Task，就出现...会发现这个Task来自于Task.cs

AsyncTcp->

AsyncTcpServer

## 程序

**TestVision**

Program类：

CmdServer类

* InitSocket函数：创建AsyncTcpServer对象，绑定连接回调和断联回调以及server_PlaintextReceived明文接收？？回调，并启动其start函数
* server_PlaintextReceived函数：创建了VisionTask（HwvFramework命名空间）对象并执行其run函数，并通过AsyncTcpServer将VisionTask得到的参数进行发送

## 开发

已经有接口定义的软硬件资源要求按资源库开发，平台功能支持不足的情况可以按功能库开发

引用添加后要求把拷贝到本地选项关闭

## 问题

1、创建工程后无法设置输出目标位置

http://3ms.huawei.com/km/groups/3949280/blogs/details/13009523

2、比如在这个示例程序中，我们要添加一个LibBase.dll以及其他的依赖项，这个是需要我们手动在依赖项中一个个添加的么？在skyladder中，比如LibBase.dll就有若干个，如何选择？

在[资源库（硬件）开发手册](http://3ms.huawei.com/km/groups/3949280/blogs/details/13009179?l=zh-cn)中，示例的目录等和现在的已经不同了



3、“引用添加后要求把拷贝到本地选项关闭”，在哪设置？在.csproj文件中的<Reference那里改为false

4、比如[资源库（硬件）开发手册](http://3ms.huawei.com/km/groups/3949280/blogs/details/13009179?l=zh-cn)，不知道这些程序是要干什么

天梯软件？



## .NET

c#是与.NET平台无缝集成的编程语言。当然.NET还支持其他的编程语言。

* 跨语言：即只要是面向.NET平台的编程语言((C#、Visual Basic、C++/CLI、Eiffel、F#、IronPython、IronRuby、PowerBuilder、Visual COBOL 以及 Windows PowerShell))，用其中一种语言编写的类型可以无缝地用在另一种语言编写的应用程序中的互操作性。

* 跨平台：一次编译，不需要任何代码修改，应用程序就可以运行在任意有.NET框架实现的平台上，即代码不依赖于操作系统，也不依赖硬件环境。

  .NET平台上的跨语言是通过CLS这个概念来实现的，比如在C#项目中，可以像自身代码一样正常使用来自vb这个dll的扩展方法。但要注意，比如在vb中访问一个该语言不支持的类型（如指针）会报错的，会提示：字段的类型不受支持。因此必须得知道.NET平台支持的每一种语言和我编写代码所使用的语言的差异，从而在编写代码中避免这些。

  因此.NET专门为此参考每种语言并找出了语言间的共性，然后定义了一组规则，开发者都遵守这个规则来编码，那么代码就能被任意.NET平台支持的语言所通用。
  而与其说是规则，不如说它是一组语言互操作的标准规范，它就是公共语言规范 - Common Language Specification，简称CLS