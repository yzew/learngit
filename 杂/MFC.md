- ## lesson1 程序运行原理

- **API ：如操作系统所提供给应用程序的可操作函数的集合，即称为提供给应用程序编程的接口**


---

- **MSG：将事件包装成的称为消息的结构体**

HWND hwnd：窗口的句柄  资源(如窗口、图标等)的标识，通过其来进行索引

UNIT message：消息内容，无符号整形，(如按下鼠标左键WM_LBUTTONDOWN等,通过宏来表示具体消息)

WPARAM wparam/LPARAM lparam：都是整形，指示关于消息的附加信息,如通过附加消息得知到底按下的哪个按键

DWORD time：dword为双十二位整数,传递消息的时间

POINT pt：消息投递时光标的位置,是点结构体(x,y)

- **WinMain函数 windows程序的入口函数**
//此函数由操作系统调用

HINSTANCE hinstance/hprevinstance：当前/前一实例的句柄

LPSTR lpCmdLine：所有lp开头的都为指针类型，命令行参数

INT nCmdShow：显示状态


---
### 创建一个完整窗口
- **wndclass窗口类**

1. UNIT style 类型，某位为1为具有某种特性，如CS_HREDRAW/CS_VREDRAW：水平重画/垂直重画，即拉伸窗口导致水平垂直坐标变化时窗口进行重画
```c++
WNDCLASS wndclass;
//同时具有两种特性
wndclass.style=CS_HREDRAW | CS_VREDRAW | CS_NOCLOSE;
//在原有几个特征中去掉某个特征,取反后 与
style & ~CS_NOCLOSE;
```
2. WNDPROC lpfnWndProc 窗口过程函数，指定不同类型窗口的回调函数
```C++
wndclass.lpfnWndProc=WinSunProc;
//将回调函数函数名作为指针
```
3.int cbClsExtra/cbWndExtra
程序中一般设为0

4.HINSTANCE hInstance
```
windclass.hInstance=hInstance;
```

5.HICON hIcon 图标的句柄
```
windclass.hIcon=LoadIcon(NULL,IDI_APPLICATION);
//IDI_APPLICATION表示应用程序的图标，ERROR为错误的图标，就是图案变了
```

6.HCURSOR hCursor 光标的句柄
```
windclass.hCursor=LoadCursor(NULL,IDC_ARROW);
//IDC_ARROW箭头形状光标
```

7.HBRUSH hbrBackground
```
windclass.hbrBackground=(HBRUSH)GetStockObject(DKGRAY_BRUSH);
//DKGRAY深黑色画刷，画刷就是背景
//(HBRUSH)数据类型强制转换
```
8.LPCTSTR lpszMenuName/lpszClassName
菜单/类的名字
```
windclass.lpszMenuName=NULL;
windclass.lpszClassName="WeiXin";//这个窗口类的名字，自己取个
```


- **registerclass 注册窗口类**
```
RegisterClass(&wndclass);
```
- **creatwindow 创建窗口**
```
HWND hwnd;//定义一个句柄，保存新建窗口的标识
hwnd=creatwindow(“WeiXin”,"随便写",WS_OVERLAPPEDWINDOW,
CW_USEDEFAULT,CW_USEDEFAULT,CW_USEDEFAULT,
CW_USEDEFAULT,NULL,NULL,hInstance,NULL);
//第一个WeiXin是前面定义的窗口类的名字
//第二个是窗口的名字，标题栏上显示的那个，自己取一个
//第三个，窗口类型，有很多，比如用的这个就是很多功能与起来的。若想单独去掉某个功能，如去掉最大化按钮，在后面加上 & ~WS_MAXIMIZEBOX，即取反后与一下
//第四五分别是x y坐标，这里是缺省，就是默认，这里坐标的意思是窗口出现的时候左上角的坐标，如果都设为0则窗口左上角在屏幕最左上
//第六七分别是宽度和高度
//第九是副窗口的句柄，一个窗口的就没有
//第十是菜单句柄
//第十一，当前应用程序实例的句柄
//第十二，wm create附加参数的指针，在创建多文档参数时要用到
```
- **showwindow 显示窗口**
```
showwindow(hwnd,SW_SHOWNORMAL);
//句柄和显示状态,SW_SHOWMAXIMIZED是最大化显示，这个是正常显示
```
- **UpdateWindow 刷新窗口**
```
UpdateWindow(hwnd);
```


---
### 消息循环
- **GetMessage 获取消息**


类型为Bool，从消息队列中取出消息时返回值为真
```
GetMessage(&msg,NULL,0,0);
//结构体变量的地址；句柄，NULL是获取任何窗口的消息；第一个和最后一个消息，设为0是返回所有可以利用的消息，设其他值是范围的过滤
```
- **TranslateMessage 转换消息**
```
TranslateMessage(&msg);
//将按键的消息对转化为WM_Char消息
```
- **DispatchMessage**
```
DispatchMessage(&msg);
//将函数回调给操作系统，让操作系统的窗口过程函数处理
```
- **WinSunProc 窗口过程函数**

或叫WindowProc
```
LRESULT CALLBACK WinSunProc
{
    HWND hwnd,
    UINT uMsg,
    WPARAM wParam,
    LPARAM lParam
}//窗口句柄；消息标识；第一第二个消息的附加参数
```
---
### 消息类型
uMsg有五种以上可能
switch(uMsg)

---
#### WM_CHAR
```
case WM_CHAR://有按键输入
    char szChar[20];
    sprintf(szChar,"char is %d",wParam);//将ASC码格式化到字符数组中
    MessageBox(hwnd,szChar,"weixin",MB_OK);//这里按下按键就会弹窗显示asc码
    break;
```

- **messagebox**


```c++
MessageBox(hwnd,szChar,"weixin",MB_OK);
//句柄、内容、标题、类型(OK OKCANCEL YESNO等)，写作MB_OK
```
---
#### WM_LBUTTONDOWN
```
case WM_LBUTTONDOWN://鼠标左键按下消息
    MessageBox(hwnd,"mouse click","weixin",MB_OK);
    HDC hDC;//获取DC的句柄.
    //DC是设备描述表，用于应用程序和物理设备交互
    hDC=GetDC(hwnd);
    TextOut(hDC,0,50,"计算机培训",strlen("计算机培训"));
    ReleaseDC(hwnd.hDC);
    break;
```

- **TextOut 输出文本**

句柄;开始输出位置的X坐标;Y坐标;文本内容;字符数(可通过strlen获取字符串长度)
```c++
TextOut(hDC,0,50,"啊啊啊",strlen("啊啊啊"));
```
- **GetDC 得到DC  ReleaseDC 释放DC**

得到一个DC用完后，要释放DC

不能在WM_PAINT重绘中使用
```c++
GetDC(hwnd);
ReleaseDC(hwnd,hDC);
```
---
#### WM_PAINT 重绘
```
case WM_PAINT:
    hdc=BeginPaint(hwnd,&ps);
    TextOut(hdc,0,0,"啊啊啊",strlen("啊啊啊"));
    EndPaint(hwnd,&ps);
    break;
```

- **BeginPaint EndPaint**

DC获取和释放，只能在WM_PAINT重绘中使用

句柄;LPPAINTSTRUCT结构体指针，带LP是指针



---
#### WM_CLOSE
```
case WM_CLOSE:
    if(IDYES==MessageBox(hwnd,"你是否要退出程序？","weixin",MB_YESNO))
    {
    //IDYES是MessageBox返回值之一，若点击的是YES，则满足if，退出
    //常量放在前面，如果==写成=时可以及时纠错
        DestroyWindow(hwnd);
    }
    break;
```
- **DestroyWindow 销毁指令窗口**

但程序没有退出
```c++
DestroyWindow(hwnd);
//返回WM_DESTROY然后再下一步中销毁程序
```
---
#### WM_DESTROY
```
case WM_DESTROY:
    PostQuitMessage(0);
    break;
```
- **PostQuitMessage 退出程序**
```c++
PostQuitMessage(0);
```
---

- **缺省(默认)窗口过程**
```c++
default:
    return DefWindowProc(hwnd,uMsg,wParam,lParam);
//其他不感兴趣的消息，就设为默认。必须要写
```


---
## lesson2 C++基本语法

### 1.输入输出流
cin>> 输入
```
int i;
cin>>i;
```
cout<<输出

cerr<<标准错误输出
```
#include <iostream>
//C++标准输入输出头文件
/*
struct Point
{*/

class Point
{
public://结构体和这里的类都可以
//结构体，默认是公有，可以在外访问，而类，默认是私有的，private，无法访问，因此要public
	int x;
	int y;
/*	void init()
	{
		x = 0;
		y = 0;
	}*/
	//设个初始值。也可以防止忘记赋值后出现的随机数太大。
	//这里用C++提供的构造函数来完成
	Point()
	{
		x = 0;
		y = 0;
	}
	//函数的重载，省去后面一个个设参数
	Point(int a, int b)
	{
		x = a;
		y = b;
	}
	//析构函数。不允许带参数
	//当一个对象生命周期结束时，其所占有的内存空间就要被回收，此工作由析构函数完成
	~Point()
	{
	}
	void output()//C++中结构体里是可以包含函数的
	{
		//调用C++标准库时，写上std
		std::cout << x << std::endl << y << std::endl;
		//endl是换行
	}
};//注意这个分号
int main()
{
   // std::cout << "Hello World!\n";
	//Point pt;
	Point pt(3,3);
	//pt.init();
	//pt.x = 5;
	//pt.y = 5;
	pt.output();
	return 0;
}
```

### 2.类的继承
```
#include <iostream>

class Animal
{
public:
	Animal(int height, int weight)
	{
		std::cout << "animal construct" << std::endl;
	}
	~Animal()
	{
		std::cout << "animal deconstruct" << std::endl;
	}
	void eat()
	{
		std::cout << "animal eat" << std::endl;
	}
protected:
	void sleep()
	{
		std::cout << "animal sleep" << std::endl;
	}
};

//继承
class Fish :public Animal
{
public:
	void test()
	{
		sleep();//这是可以的，父类里的protected可以在子类里访问
	}
	Fish() :Animal(400, 300)
	//在子类中向父类带参数的构造函数传递参数
	{
		std::cout << "fish construct" << std::endl;
	}
	~Fish()
	{
		std::cout << "fish deconstruct" << std::endl;
	}
	void breathe()
	{
	    Animal::breathe();
	    //这里子类的breathe覆盖了父类的，只显示子类的。若想也显示父类的，用作用域标识符::，表示这个函数属于哪个类
	    std::cout<<"fish bubble"<<std::endl;
	}
	
};

int main()
{
	Animal an;
	an.eat();
	Fish fh;
	fh.sleep();//这是错误的，父类里的protected只能在子类里访问
	fh.breathe();
}
```
#### Animal 基类，即原始类，父类

#### Fish 派生类，即继承类，子类

class Fish :public Animal

父类的protected可以且只能在子类里访问，不能在外部访问；private在子类里也不能访问



| 基类的访问特性           | 类的继承特性 | 子类的访问特性                 |
| ------------------------ | ------------ | ------------------------------ |
| Public Protected Private | Public       | Public Protected  No access    |
| Public Protected Private | Protected    | Protected Protected  No access |
| Public Protected Private | Private      | Private Private  No access     |

### 3.引用
```
int a=6;
int &b=a;
b=5;
```

### 4.
```
class Animal
{
public:
    void eat();
};//声明放头文件,如Animal.h


//实现放源文件，如Animal.cpp
#include "Animal.h"
#include <iostream.h>
Animal::Animal()
{}//构造函数，初始化成员
void Animal::eat()
{}
```
## lesson3 MFC框架程序剖析
### 代码结构
![在这里插入图片描述](E:\MarkDown\picture\20201214101048458.png)在MFC程序有且仅有一个从应用程序类(CWinApp)派生的类，而且仅有一个该派生类的实例化对象；我们发现该程序中确实存在一个theApp的全局变量，该全局变量就代表了这个应用程序本身；win32和MFC应用程序实例表示区别如下：
![在这里插入图片描述](E:\MarkDown\picture\20201214102849654.png)知识点：
        1.Afx前缀的函数代表应用程序框架（Application Framework）函数，属于全局函数，它们可以在程序的任何地方被调用。
        2.以域作用符“::”开始的表示的函数，表明该函数是一个全局函数。

### MFC运行流程


现在直接给出MFC程序执行顺序，但着重分析其运行机制和功能分析，其流程是“theApp全局对象定义->TestApp构造函数->WinMain函数”。在执行theApp对象的构造函数之前先执行CWinApp基类的构造函数，从而把我们自己创建的类和MFC类相关联起来了。


流程详解：


1.全局变量定义；程序入口函数WinMain加载时，系统先为全局对象分配内存空间，从而利用theApp完成应用程序的启动。

2.创建对象时会调用对象的构造函数；theApp是子类CTestApp是实例对象，子类继承于CWinApp，因此会先调用基类的构造函数，再调用子类的构造

函数，从而完成应用程序的初始化工作，例如基类中保存theApp的this指针。

3.进入WinMain函数；在AfxWinMain函数中可以获取子类的this指针，利用此指针调用InitApplication、InitInstance、Run等函数，从而完成窗口类的注册，创建，消息循环、显示，更新。

4.进入消息循环，响应各种消息，直到退出；MFC程序实际上是采用消息映射机制，来完成各种消息的处理，收到WM_QUIT消息时，退出消息循环。

![在这里插入图片描述](E:\MarkDown\picture\20201214103615331.png)
## lesson4 MFC消息映射机制的剖析
**添加消息： Ctrl+Shift+X 添加自定义消息**
在View类中增加消息
删除消息时也在这个界面删除
![在这里插入图片描述](E:\MarkDown\picture\2020121416133391.png)
```
//增加一个按下左键弹窗
void CMFCApplication4View::OnLButtonDown(UINT nFlags, CPoint point)
{
	// TODO: 在此添加消息处理程序代码和/或调用默认值
	MessageBox(_T("a"));
	CView::OnLButtonDown(nFlags, point);
}
```


画线
捕获两个消息，鼠标左键按下和抬起
#### 用全局函数完成

```
//在构造函数中定义一个原点，即鼠标按下的点
// CMFCApplication4View 构造/析构

CMFCApplication4View::CMFCApplication4View() noexcept
{
	// TODO: 在此处添加构造代码
	m_ptOrigin = 0;

}


//在view类中增加按键按下和抬起两个消息
void CMFCApplication4View::OnLButtonDown(UINT nFlags, CPoint point)
{
	// TODO: 在此添加消息处理程序代码和/或调用默认值
	m_ptOrigin = point;
	CView::OnLButtonDown(nFlags, point);
}


void CMFCApplication4View::OnLButtonUp(UINT nFlags, CPoint point)
{
	// TODO: 在此添加消息处理程序代码和/或调用默认值
	HDC hdc;
	hdc = ::GetDC(m_hWnd);//MFC中用全局的函数，需要加俩冒号
	//将点移动到一开始的m_ptOrigin位置
	MoveToEx(hdc, m_ptOrigin.x, m_ptOrigin.y,NULL);
	//画线函数
	LineTo(hdc, point.x, point.y);
	//最后要释放DC
	::ReleaseDC(m_hWnd, hdc);
	CView::OnLButtonUp(nFlags, point);
}
```

窗口相关操作封装到Wnd中，和作图相关的封装到CDC
#### 用CDC类里的函数实现
```
//仅此函数不同
void CMFCApplication4View::OnLButtonUp(UINT nFlags, CPoint point)
{
	// TODO: 在此添加消息处理程序代码和/或调用默认值

	CDC *pDC = GetDC();
	pDC->MoveTo(m_ptOrigin);
	pDC->LineTo(point);
	ReleaseDC(pDC);
	CView::OnLButtonUp(nFlags, point);
}

```

#### 用CClientDC实现
在构造的时候调用GetDC，在析构的时候调用ReleaseDC，自动释放句柄
```
void CMFCApplication4View::OnLButtonUp(UINT nFlags, CPoint point)
{
	// TODO: 在此添加消息处理程序代码和/或调用默认值

	CClientDC dc(this);//每一个类，后台都隐含着this指针指向本身
	//要是换成CWindowDC dc(GetParent());可以访问整个屏幕区域，即也可以在菜单上画线
    //换成CWindowDC dc(GetDesktopWindow());则可在整个屏幕上画线
	dc.MoveTo(m_ptOrigin);
	dc.LineTo(point);
	CView::OnLButtonUp(nFlags, point);
}
```
看到了47.09秒

## lesson11 图形的保存和重绘
单文档