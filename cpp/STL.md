# reference

STL源码剖析 + 侯捷STL课程

> effective STL
>
> https://zhuanlan.zhihu.com/p/458156007
>
> https://blog.csdn.net/fengbingchun/article/details/103223914
>
> https://blog.csdn.net/m0_46415159/article/details/108561735



# STL



![image-20221114152820130](E:\MarkDown\picture\image-20221114152820130.png)



面向对象编程OOP和泛型编程GP（基于模板实现）是STL容器设计中使用的两种设计模式

- OOP的目的是将**数据**和**方法**绑定在一起，例如对`std::list`容器进行排序要调用`std::list::sort`方法.
- Generic Programming(GP,泛型編程)，就是使用template (模板)为主要工具来编写程序。GP的目的是将**数据**和**方法**分离开来，例如对`std::vector`容器进行排序要调用`std::sort`方法：
  ![image-20221114192854674](E:\MarkDown\picture\image-20221114192854674.png)



> 因此，操作符重载和模板是STL实现的两大基础



# STL的六大部件

- 容器（container）
- 分配器（allocator）
- 迭代器（iterator）
- 适配器（adapter）
- 泛型算法（algorithm）
- 仿函式/仿函数（functor），作为参数传递给泛型算法

![image-20221114152930080](E:\MarkDown\picture\image-20221114152930080.png)

![image-20221114153058004](E:\MarkDown\picture\image-20221114153058004.png)

```cpp
#include <iostream>
#include <algorithm>//count_if
#include <functional>//less	bind
#include <vector>

using namespace std;

int main(void)
{
	//创建一个容器
	int buf[] = { 27,210,12,47,109,83 };
	//这里刻意写出第二模板参数，是为了解释分配器的作用
	vector< int, allocator<int> > v(buf, buf + 6);
	using namespace std::placeholders;//使用 占位符_1所在的 命名空间
	//输出vector中小于40的元素的个数
	cout << count_if(v.begin(), v.end(),  bind( less<int>(), _1, 40 ) ) << endl;
	return 0;
}
```



![image-20221114153226744](E:\MarkDown\picture\image-20221114153226744.png)

六大部件之间的关系：

- 容器用来存储元素，实现是一种class template
- 分配器用来给容器分配内存大小，class template
- 迭代器用来实现容器中元素的访问，是算法和容器之间的桥梁，class template
- 算法可以独立出来，通过迭代器作用到多种容器，function template
- 适配器用做对象的包装，使之适应不同的接口实现，class template
- 仿函式是一种用来实现函数功能的class template



## 一、容器

### 容器分类

> 分为顺序容器、关联式容器、无序容器
>
> 也可分为顺序容器和关联式容器两类，无序容器本质上属于关联式容器

![image-20221114154659964](E:\MarkDown\picture\image-20221114154659964.png)

<img src="E:\MarkDown\picture\image-20221114154432770.png" alt="image-20221114154432770"  />

​	容器不包括容器适配器（stack、queue、priority_queue），以及bitset容器（STL中的二进制容器，有空间优化，一个元素一般只占1bit。具体见primer17.2）、valarray容器（主要用于执行数学运算）

### 容器使用

### 顺序容器

> STL中定义的基本顺序容器有：
> 数组(array)： 顺序存储、支持下标访问、不可扩增（c++11)
> 向量(vector)： 顺序存储、支持下标访问、可在尾部扩增
> 双向队列(deque)： 非顺序存储，支持下标访问，可双向扩增
> 双向链表(list)： 非顺序存储，不支持下标访问，可在任意位置扩增
> 堆(heap)： 利用完全二叉树存储，不支持下标访问，可在尾部扩增
> 至于另外一些常见的如栈(stack)、单向队列(quque)、单向链表(forward_list)、优先队列(priority_queue)等等，都只是基本容器的适配器。

![image-20221114185036569](E:\MarkDown\picture\image-20221114185036569.png)

```CPP
// 创建一个辅助函数用于测试容器
const long ASIZE = 500000L;		// 数组大小

// 输入一个给定范围内的long
long get_a_target_long() {
    long target = 0;
   
    cout << "target (0~" << RAND_MAX << "): ";
    cin >> target;
    return target;
}

// 从console中读入一个string
string get_a_target_string() {
    long target = 0;
    char buf[10];

    cout << "target (0~" << RAND_MAX << "): ";
    cin >> target;
    snprintf(buf, 10, "%d", target);  // 将target按整形的格式格式化为字符串，字符串长度取10，然后存入buf中
    return string(buf);
}

// 比较 long
int compareLongs(const void *a, const void *b) {
    return (*(long*)a - *(long*)b);
}

// 比较 string
int compareStrings(const void *a, const void *b) {
    if (*(string *) a > *(string *) b)
        return 1;
    else if (*(string *) a < *(string *) b)
        return -1;
    else
        return 0;
}
```

#### 1.array

​	array是在C++11之后才加入STL之中的，因为C++本身就有一个原生的数组，array的用法也和它完全相同，但是`array可以使用STL的泛型算法`，而原生数组因为没有定义迭代器，所以不能融入STL之中

**使用**

array必须指定容器的大小

```cpp
#include <array>

//模板参数1是元素的类型，2是元素的个数，3是分配器，默认为std::allocator
array<int, 5> arr1 = {1,2,3,4,5};
int arr2[] = { 6,7,8,9,0 };  // 创建一个原生数组
//输出 1 2 3 4 5
for_each(arr1.begin(), arr1.end(), [&](const int& a) {cout << a << " "; });
cout << endl;
for (auto a : arr2) { cout << a << " "; }//输出6 7 8 9 0
```

arr1是一个容器，所以可以使用STL中的算法for_each()去遍历整个数组；而arr2是原生的数组，所以只能使用for循环去遍历

**成员类型**

![image-20230403104657764](E:\MarkDown\picture\image-20230403104657764.png)

![img](E:\MarkDown\picture\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0MjY5NjMy,size_16,color_FFFFFF,t_70.png)

**底层内存结构和实现方法**

> array的所有元素一定位于连续且相邻的内存内，因此总有 &a[i] == &a[0] + i
>
> 对于array，其成员函数**array.data()** == &array[0]，.data()返回首元素的地址，即指向数组对象中第一个元素的指针，这个成员函数仅array、vector和string提供

​	array就是一个固定长度的数组。将数组封装成容器`array`是为了使之与STL算法兼容，其内部实现只是简单封装了一下数组，甚至没有构造函数和析构函数。只不过外附了一个array的迭代器，使之可以融入STL之中。他的迭代器的类型是随机迭代器。



在TR1中，用原生指针做迭代器，通过iterator traits时走偏特化路线

![image-20221115103725216](E:\MarkDown\picture\image-20221115103725216.png)

```cpp
template<typename _Tp, std::size_t _Nm>
struct array {
    typedef _Tp value_type;
    typedef _Tp *pointer;
    typedef value_type *iterator;

    value_type _M_instance[_Nm ? _Nm : 1];	// Support for zero-sized arrays mandatory

    iterator begin() {
        return iterator(&_M_instance[0]);
    }

    iterator end() {
        return iterator(&_M_instance[_Nm]);
    }
};

```

在G4.9中，array的源代码和vector一样变得复杂，它的迭代器变成了类，详略



#### 2.==vector==

https://en.cppreference.com/w/cpp/container/vector

![在这里插入图片描述](E:\MarkDown\picture\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0MjY5NjMy,size_16,color_FFFFFF,t_70-16804902140452.png)

![img](E:\MarkDown\picture\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0MjY5NjMy,size_16,color_FFFFFF,t_70-16804902292984.png)

##### 2.1 扩容机制

​	vector每当遇到空间不同的情况，都会按照当前最大空间的两倍空间进行空间申请。如果申请不到两倍大的空间，生命就会自动结束。当向vector中插入元素时，如果元素有效个数size与空间容量capacity相等时，vector内部会触发扩容机制：**开辟新空间**、拷贝元素、释放旧空间。

**注意：每次扩容新空间不能太大，也不能太小，太大容易造成空间浪费，太小则会导致频繁扩容而影响程序效率。**

**linux下是按照2倍的方式扩容的，而vs下是按照1.5倍的方式扩容的。**

[很详细的扩容机制解析(什么时候扩容、如何避免扩容)](https://blog.csdn.net/qq_44918090/article/details/120583540)

![image-20221114231509541](E:\MarkDown\picture\image-20221114231509541.png)



> 特性：可以自动扩充
>
> 扩展的方式是两倍扩展（即从4到5，4的容量不够了，就扩大到8，然后把5填上。可以从size和capacity看出这里的扩展是另外找一个两倍大小的地方，再搬过去

前面讲的做的一个测试，当输入1000000个元素时，vector.size()= 1000000，而vector.capacity()= 1048576，vector自动进行了扩充

```cpp
how many elements:1000000

test_vector()..........
milli-seconds : 211
vector.max_size()= 576460752303423487
vector.size()= 1000000
vector.front()= 30271
vector.back()= 7756
vector.data()= 0x4830040
vector.capacity()= 1048576

target (0~32767):23456
std::find(), milli-seconds : 1
found, 23456

sort(), milli-seconds : 2695
bsearch(), milli-seconds : 1
found, 23456
```

像array，是一设定后就固定大小的，所以没有max_size，而vector、list、forward_list，他们的max_size()都很大，可以无限扩充

**扩容的实现**

<img src="E:\MarkDown\picture\image-20230508165858212.png" alt="image-20230508165858212" style="zoom:50%;" />

> std::copy是个某些情况下优于memcpy的函数，具体介绍见c++备战.md中memcpy的介绍中

​	vector扩容时使用的是uninitialized_copy（[层层剖析见此](https://blog.csdn.net/qq_21197941/article/details/125231939)），会先判断一个元素是否为POD，如果是的话，通过std::copy实现；如果不是的话先通过**__uninitialized_copy** 实现内存初始化，再通过_Construct函数拷贝构造对象，调用construct函数，将输入区间[first,last) 的每个对象生成一个复制品，然后放置于未初始化输出区间 [result, result+(last-first))。

##### 2.2 ==**vector实现**==

```cpp
// gcc7.1.0
// vector继承于_Vector_base，而_Vector_base中内存分配又是结构体_Vector_impl实现的，这个结构体继承于_Tp_alloc_type，类型_Tp_alloc_type的完整类型是__gnu_cxx::__alloc_traits<_Alloc>::template rebind<_Tp>::other，而根据函数_M_allocate最终实现内存分配是通过这样一个方式实现的：__gnu_cxx::__alloc_traits<_Tp_alloc_type>::allocate(_Tp_alloc_type, __n);

template<typename _Tp, typename _Alloc>
    struct _Vector_base
    {
      typedef typename __gnu_cxx::__alloc_traits<_Alloc>::template
    rebind<_Tp>::other _Tp_alloc_type;
      typedef typename __gnu_cxx::__alloc_traits<_Tp_alloc_type>::pointer
        pointer;

      struct _Vector_impl
      : public _Tp_alloc_type
      {
      ...
      };
      ...
      public:
      _Vector_impl _M_impl;

      pointer
      _M_allocate(size_t __n)
      {
    typedef __gnu_cxx::__alloc_traits<_Tp_alloc_type> _Tr;
    return __n != 0 ? _Tr::allocate(_M_impl, __n) : pointer();
      }
      ...
     };
class vector : protected _Vector_base<_Tp, _Alloc>
{...};



// vector的简单实现代码
template<class T, class Alloc= alloc>
class vector {
public:
    typedef T value_type;
    typedef value_type* iterator;
    typedef value_type& reference;
    typedef size_t size_type;
protected:
    // vector的实现是连续的,因此可以直接使用指针类型做迭代器(即迭代器vector<T>::iterator的实际类型是原生指针T*)
    // 对于vector的迭代器，直接使用了原生指针。vector主要是通过三个指针来维护的，分别是起点，当前终点，以及当前最大空间。迭代器start指向第一个元素，迭代器finish指向最后一个元素的下一个元素，这两个迭代器对应begin()和end()的返回值，维持了前闭后开的特性
    iterator start;
    iterator finish;
    iterator end_of_storage;
public:
    iterator begin() { return start; }
    iterator end() { return finish; }
    size_type size() const { return size_type(end() - begin()); }
    size_type capacity() const { return size_type(end_of_storage - begin()); }
    bool empty() const { return begin() == end(); }
    // vector对使用者是连续的。因此重载了[]运算符
    reference operator[](size_type n) { return *(begin() + n); }
    reference front() { return *begin(); }
    reference back() { return *(end() - 1); }
};

```



**构造函数和析构函数**

这里以一个简单的vector实现为例

```cpp
public：
	//构造析构函数
	MyVecotr() :start(0), finish(0), end_of_storage(0) {};
	MyVecotr(size_type n, const T& value) { fill_initialize(n, value); }
	MyVecotr(int n, const T& value) { fill_initialize(n, value); }
	MyVecotr(long n, const T& value) { fill_initialize(n, value); }
	~MyVecotr() {
		destroy(start, finish);//调用析构函数
		deallocate();//释放内存
	}
```

在这里只列出了一种**构造函数**，也就是填充为N个value，其他构造函数的思想相同
在构造函数中会调用fill_initialize()进行实际的空间构造，这个函数定义如下：

```cpp
	void fill_initialize(size_type n, const T& value)
	{//初始化n个value
		start = allocate_and_fill(n, value);//申请空间并填入值
		finish = start + n;//调整迭代器
		end_of_storage = finish;
	}
```

它首先申请n个value的空间，之后调整三个迭代器的位置，使他们指向正确的位置，申请空间的函数是：

```cpp
iterator allocate_and_fill(size_type n, const T& value)
	{//申请空间并填入值
		//iterator res = std::allocator<T>().allocate(n);//调用标准分配器分配内存空间
		iterator res = (iterator)alloc.allocate(n * sizeof(T));//调用MyAllocator
		uninitialized_fill_n(res, n, value);//填充初值
		return res;
	}
```

这里我列出了两种申请空间的方式，一种是使用标准分配器std::allocator的；另一种是使用我自己定义的分配器alloc的。由于这时第一次申请，所以我们只申请足够的空间就行了，当需要扩增时，再去进行申请。

这里由于我自定义的分配器为单例模式，它需要为全部的容器服务，所以他所交付出的内存的指针是void*的类型，需要强制类型转换。但是无论是哪一种方式分配出的内存，都是一整块大的内存，所以我们不能直接为他填入初值，这里需要调用另外一个函数uninitialized_fill_n()：

```cpp
template<typename ForwardIterator, class Size , class T>
inline ForwardIterator uninitialized_fill_n(ForwardIterator first, Size n, const T& x)
{
	ForwardIterator cur = first;
	for (; n > 0; --n, ++cur)
		_construct(&*cur, x);
	return cur;
}

template<typename T1, typename T2>
inline void _construct(T1* p, const T2 value) {
	new(p) T2(value);
}
```

在uninitialized_fill_n()中，我们依次跳转指针，在每个空间上通过new()填入一个值

这里值得一提的是，在STL中为了提升性能，它使用了traits对这一步进行了优化，对于不同的value_type类型，偏特化出了很多不同版本的uninitialized_fill_n()和_construct()，我这里使用的是最泛化的版本。

对于析构函数，需要分两步进行，一是调用所有成员的析构函数，二是释放全部内存
调用析构函数的函数如下：

```cpp
template<class T>
inline void destroy(T frist, T last) {//对一个范围中的所有成员调用析构函数
	for (; frist != last; frist++)
	{
		destroy(&*frist);
	}
}

template<class T>
inline void destroy(T* ptr) {//对一个成员调用析构函数
	ptr->~T();
}
```

这里值得一提的是，并不是所有类型的成员都需要调用析构函数。在STL库中，这里会利用traits进行一次判断，只有当该成员需要调用析构函数时才会去调用，这样可以提高效率。具体做法是，在成员中定义自己的析构函数类型，然后通过traits去询问。我这里为了突出容器的行为，就只列出了最泛化的版本。
释放内存的函数：

```cpp
// 这里首先要进行判断，只有当vector中有元素时才会进行内存的回收。至于回收内存的动作，就交给分配器去做了。
void deallocate() {//释放vector空间
    if (start)
        //std::allocator<T>().deallocate(start, (end_of_storage - start));//调用标准分配器
        alloc.deallocate(start, (end_of_storage - start)*sizeof(T));//调用MyAllocator
}
```

##### 2.3 push_back实现

![在这里插入图片描述](E:\MarkDown\picture\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASU5saW5LQw==,size_20,color_FFFFFF,t_70,g_se,x_16.png)

`vector::push_back`方法先判断内存空间是否满。若内存空间不满则直接插入；若内存空间满则调用`insert_aux`函数先扩容两倍再插入元素

```cpp
template<typename T1, typename T2>
inline void _construct(T1* p, const T2 value) {
	new(p) T2(value);
}

void push_back(const T &x) {
    if (finish != end_of_storage) { // 尚有备用空间,则直接插入,并调整finish迭代器
        _construct(finish, x);		
        ++finish;					
    } else  // 已无备用空间则调用 insert_aux 先扩容再插入元素
        // 当push_back调用时，这个插入点就是尾后指针
        insert_aux(end(), x);
}

// 在移除元素时，不需要进行缩放内存，单纯的令最后一个元素调用析构函数（STL库中仍然会通过traits去判断是否真的需要去调用），并移动尾后指针就可以了。
void pop_back() {
    destroy(finish);
    finish--;
}
```

`insert_aux`被设计用于在容器任意位置插入元素，在容器内存空间不足会现将原有容器扩容

```cpp
template<class T, class Alloc>
void vector<T, Alloc>::insert_ux(iterator position, const T &x) {
    if (finish != end_of_storage) {     // 尚有备用空间,则将插入点position后的元素后移一位并插入元素
        construct(finish, *(finish - 1)); //在尾指针处初始化一块空间，内容为vector的最后一个元素
        ++finish;
        T x_copy = x;
        //将position到finish-2的内容向后移一格
        copy_backward(position, finish - 2, finish - 1);
        *position = x_copy;  //在插入处赋值
    } else {
        // 已无备用空间,则先扩容,再插入
        const size_type old_size = size();
        const size_type len = old_size != 0 ?: 2 * old_size:1;  // 扩容后长度为原长度的两倍

        iterator new_start = data_allocator::allocate(len);
        iterator new_finish = new_start;
        try {
            //将position之前的内容拷贝到新位置
            new_finish = uninitialized_copy(start, position, new_start);
            construct(new_finish, x);  //将position位置的内容创建
            ++new_finish;
            // 拷贝插入点后的元素
            new_finish = uninitialized_copy(position, finish, new_finish);
        }
        catch (...) {
            // 插入失败则回滚,释放内存并抛出错误
            destroy(new_start, new_finish) :
            data_allocator::deallocate(new_start, len);
            throw;
        }
        // 释放原容器所占内存
        destroy(begin(), end());
        deallocate();
        // 调整迭代器
        start = new_start;
        finish = new_finish;
        end_of_storage = new_start + len;
    }
};
```

##### 2.4==push_back/emplace_back==

> reference：https://zhuanlan.zhihu.com/p/213853588
>
> 注意，在c++11之后，这两个就相同了，push_back直接调用emplace_back
>
> ```cpp
> #if __cplusplus >= 201103L
> void push_back(value_type &&__x) {
>     emplace_back(std::move(__x));
> }
> #endif
> ```

​	emplace_back()是 C++11 新增加的，其功能和 push_back() 相同，都是在 vector 容器的尾部添加一个元素。emplace_back() 和 push_back() 的区别，就在于**底层实现的机制不同**。

相同点：
	实现的逻辑是相同的，即首先判断容器满没满，如果没满那么就构造新的元素，然后插入新的元素；如果满了，那么就重新申请空间，然后拷贝数据，接着插入新数据

**区别：**

* push_back() 向容器尾部添加元素时，首先会**创建**这个元素，然后再将这个元素**拷贝**或者**移动**到容器中（如果是拷贝的话，事后会自行销毁先前创建的这个元素）。即调用构造函数后再调用移动构造函数
  <img src="E:\MarkDown\picture\image-20230403114322455.png" alt="image-20230403114322455" style="zoom:55%;" />
* emplace_back() 在实现时，通过forward完美转发直接将x传递给对应类的构造函数，直接将构造函数所需的参数传递过去，然在容器尾部构建一个新的对象，省去了拷贝或移动元素的过程。即只调用构造函数。
  ![image-20230403114436921](E:\MarkDown\picture\image-20230403114436921.png)

```cpp
// emplace_back()
_Alloc_traits::construct(this->_M_impl, this->_M_impl._M_finish,
	std::forward<_Args>(__args)...); 
// push_back()
_Alloc_traits::construct(this->_M_impl, this->_M_impl._M_finish,
	__x);

// std::forward
// code from bits/move.h
template<typename _Tp>
constexpr _Tp &&forward(typename std::remove_reference<_Tp>::type &__t) noexcept {
    return static_cast<_Tp &&>(__t);
}
```



##### **vector的迭代器**

G2.9版本的vector迭代器

由于vector本身就是连续的，内存也是连续的，所以正常来讲vector的迭代器不必设置的非常复杂，只需要一个指针就够了。事实上，G2.9中确实是这么做的。
在G2.9版本中，vector的迭代器就是一个指针。如果将它放入iterator traits当中的话，由于这个迭代器是单独的指针而不是一个类，所以会走偏特化的路线来为算法提供所需的性质。
![image-20221115102201082](E:\MarkDown\picture\image-20221115102201082.png)

G4.9版本的vector迭代器

然而在G4.9中，vector的迭代器被设计的十分复杂，同时变成了一个类。所以G4.9之后的vector迭代器不会再走指针偏特化的iterator traits了。
 但是这个操作十分的复杂，而且并没有影响最终的结果，也就是说**G2.9和G4.9的迭代器并没有什么本质区别**。

![image-20221115103259600](E:\MarkDown\picture\image-20221115103259600.png)

#### 3.list

gcc2.9中`list`及相关类的代码如下所示:

list中只有一个指针类型的data，为四个字节，其指向的对象有三个数据

![image-20221114205249864](E:\MarkDown\picture\image-20221114205249864.png)

为实现前闭后开的特性，在环形链表末尾加入一个用以占位的空节点，并将迭代器`list::end()`指向该节点

![image-20221114205521010](E:\MarkDown\picture\image-20221114205521010.png)

迭代器 __list_iterator重载了指针的*,->,++,--等运算符，并定义了iterator_category、value_type、difference_type、pointer和pointer5个关联类型(associated types)，这些特征将被STL算法使用
![image-20221114210723155](E:\MarkDown\picture\image-20221114210723155.png)

```cpp
template<class T, class Ref, class Ptr>
struct __list_iterator {
    typedef __list_iterator<T, Ref, Ptr> self;
    typedef bidirectional_iterator_tag 	iterator_category; 	// 关联类型1
    typedef T 							value_type;			// 关联类型2
    typedef ptrdiff_t 					difference_type;	// 关联类型3
    typedef Ptr 						pointer;			// 关联类型4
    typedef Ref 						reference;			// 关联类型5

    typedef __list_node <T>*			link_type;
    link_type node;		// 指向的链表节点

    reference operator*() const { return (*node).data; }
    pointer operator->() const { return &(operator*()); }

    self& operator++() {
        node = (link_type) ((*node).next);
        return *this;
    }

    self operator++(int) {
        self tmp = *this;
        ++*this;
        return tmp;
    }
};
```

#### forward_list

单向链表，就是list少一个向前的指针

其优点在于，`存储相同个数的同类型元素，单链表耗用的内存空间更少，空间利用率更高`，并且对于实现某些操作单链表的执行效率也更高。因此只要是 list 容器和 forward_list 容器都能实现的操作，应优先选择 forward_list 容器。

![image-20221115104009993](E:\MarkDown\picture\image-20221115104009993.png)



**使用**

​	c++11新标准，相比于list，只能从前向后遍历，不支持反向遍历，因此只提供前向迭代器，不具有rbegin、rend之类的成员函数，迭代器不支持递减运算--

```cpp
// 基础功能实现

// forward_list容器中不提供size()函数，想要获取forward_list容器中存储元素的个数，可以使用头文件<iterator>中的distance()函数。
#include<iterator>
std::forward_list<int> a{1,2,3,4};
int count = std::distance(std::begin(a), std::end(a));
// 或
int count = std::distance(a.begin(), a.end());

// forward_list 容器迭代器的移动除了使用 ++ 运算符单步移动，还能使用 advance() 函数
std::forward_list<int> values{1,2,3,4};
auto it = values.begin();
std::advance(it, 2);
while (it!=values.end())
{
    std::cout << *it << " ";  // 这里仅输出3 4
    ++it;
}
```

**特殊的forward_list操作**

​	对于链表，由于在添加或删除元素的时候需要改变前一个元素的后继，因此我们需要访问这个元素的前驱，而单向链表无法通过简单的方法获取前驱，因此forward_list中添加或删除元素的操作是通过**改变给定元素之后的元素**来完成的
![image-20220728150655173](E:\MarkDown\picture\image-20220728150655173.png)

```cpp
// 在forward_list中添加或删除元素时，需要关注两个迭代器：一个指向我们要处理的元素，一个指向其前驱。通过对前驱的操作来处理当前元素
forward_list<int> a = {1,2,3,4,5};
auto prev = a.before_begin();
auto curr = a.begin();
while(curr != a.end()) {
	if(*curr % 2) {
		curr = a.erase_after(prev);
    }
    else {
		prev = curr;
    	++curr;  // 尽量用前置操作
    }
}
```

***



#### 4.==deque双端队列==

> 能说出双端队列的实现的原理及具体的代码实现方式等

**实现原理**

​	如果deque仍然以顺序存储的话，那么怎么在头部扩展呢？难道头部每次增加一个单位就需要将后边的元素全部向后拷贝一次吗？那这样的话开销也太大了。因此deque对外是连续的，但内部不是连续的，内部空间是`分段连续空间`，类似hashtable的形式，**通过一个数组保存指针类型的map结点。数组中每个map指针都指向了另一个数组，而这个数组才是用来存放数据的（称为buf）**。在map数组中，最开始并不会直接用第一个元素去创建buf数组，而是从map数组的中间开始创建第一个buf数组。这样，若最前端的buf中不能满足在前边加入元素的话，那就在这个buf数组的map节点的前一个map节点在开辟出一个buf数组。如下图所示

![image-20221114185253999](E:\MarkDown\picture\image-20221114185253999.png)

​	当整个map空间全部占用后，在需要拓展空间并不需要拷贝全部的数据成员，只需要将原map数组拷贝到新内存空间的中段，这样deque就可以保持向左和向右扩张的能力，这就大大节省了时间上的开销。
![image-20221115105649467](E:\MarkDown\picture\image-20221115105649467.png)



**迭代器的实现**

​	迭代器`deque::iterator`的核心字段是4个指针：`cur`指向当前元素、`first`和`last`分别指向当前buffer的开始和末尾、`node`指向控制中心，即每个迭代器由四个指针构成。`deque::map`的类型为二重指针`T**`，称为**控制中心**,其中每个元素指向一个buffer

![image-20221115105514089](E:\MarkDown\picture\image-20221115105514089.png)

![在这里插入图片描述](E:\MarkDown\picture\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASU5saW5LQw==,size_20,color_FFFFFF,t_70,g_se,x_16-16684810823192.png)

```cpp
// __deque_iterator的实现
template<class T, class Ref, class Ptr, size_t BufSiz>
struct __deque_iterator {
    // 定义5个关联类型
    typedef random_access_iterator_tag	iterator_category; 	// 关联类型1
    typedef T 							value_type;       	// 关联类型2
    typedef ptrdiff_t 					difference_type;	// 关联类型3
    typedef Ptr 						pointer;			// 关联类型4
    typedef Ref 						reference;			// 关联类型5

    typedef size_t size_type;
    typedef T **map_pointer;
    typedef __deque_iterator self;

    // 迭代器核心字段:4个指针
    T *cur;     		// 指向当前元素
    T *first;   		// 指向当前buffer的开始
    T *last;    		// 指向当前buffer的末尾
    map_pointer node;   // 指向控制中心
    // ...
};

// deque的实现
// BufSize为每个Buffer容纳的元素个数，可以指定
template<class T, class Alloc =alloc, size_t BufSiz = 0>
class deque {
public:
    typedef T value_type;
    typedef _deque_iterator<T, T &, T *, BufSiz> iterator;
protected:
    typedef pointer *map_pointer;   // T**
protected:
    // 一个deque，大小为40个字节（按一个指针4字节
    iterator start;			// 大小为四个指针
    iterator finish;		// 大小为四个指针
    map_pointer map;		// 控制中心,数组中每个元素指向一个buffer，大小为一个指针
    size_type map_size;		// 大小为四个字节
public:
    iterator begin() { return start; }
    iterator end() { return finish; }
    size_type size() const { return finish - start; }
    // ...
};

```



**deque<T>::insert()**

`deque::insert`方法
1.先是判断是否是头插或者尾插。是的话直接头尾插入元素即可。
2.如果不是头插或者尾插，那么计算这个节点到头结点和尾节点之间的距离。假如说离头部节点近，那么就让从头部节点到插入位置之间的节点全部向前挪动，然后插入节点；反之亦然。

```cpp
// 在position处插入一个元素，其值为x
iterator insert(iterator position, const value_type& x) {
    if (position.cur == start.cur) {        	// 若插入位置是容器首部,则直接push_front，在最前面创建一个buffer放到前面
        push_front(x);
        return start;
    } else if (position.cur == finish.cur) {	// 若插入位置是容器尾部,则直接push_back
        push_back(x);
        iterator tmp = finish;
        --tmp;
        return tmp;
    } else {
        return insert_aux(position, x);
    }
}

template<class T, class Alloc, size_t BufSize>
typename deque<T, Alloc, BufSize>::iterator deque<T, Alloc, BufSize>::insert_aux(iterator pos, const value_type &x) {
    difference_type index = pos - start;    // 插入点前的元素数
    value_type x_copy = x;
    if (index < size() / 2) {    	  		// 1. 如果插入点前的元素数较少,则将前半部分元素向前推
        push_front(front());        		// 1.1. 在容器首部加入一个Buffer
        // ...
        copy(front2, pos1, front1); 		// 1.2. 将前半部分元素左移，这样才能腾个空出来
    } else {                        		// 2. 如果插入点后的元素数较少,则将后半部分元素向后推
        push_back(back());          		// 2.1. 在容器末尾创建与最末元素同值的元素
        copy_backward(pos, back2, back1); 	// 2.2. 将后半部分元素右移
    }
    *pos = x_copy;							// 3. 在插入位置上放入元素
    return pos;
}
```



**迭代器`deque::iterator`如何模拟连续空间**

​	这里重载了迭代器的移动方式，每当迭代器走到整个buf的终点时，就让他跳转到下一个buf(或上一个buf)，这样在使用迭代器的++或--时，看起来整个结构就像是连续的一样。当进行随机跳转的时候，也需要判定跳转之后是仍然在当前的buf中还是到了其他的buf。

```cpp
void set_node(map_pointer new_node) {  // 设置新的map节点
    node = new_node;
    first = *new_node;
    last = first + buffer_size;
}


self& operator++() {//重载前置++
    ++cur;
    if (cur == last)//若到达了当前buf的结尾
    {
        set_node(node + 1);//修改当前迭代器所在的node节点
        cur = first;//cur跳转至下一个node的第一个元素
    }
    return *this;
}

self operator++(int) {//重载后置++
    self temp = *this;
    ++*this;  // 调用了前置--
    return temp;
}

self& operator--() {//重载前置--
    if (cur == first)//如果是当前buf的第一个元素
    {
        set_node(node - 1);//修改当前迭代器所在的node节点
        cur = last;//cur跳转至上一个node的最后一个元素
    }
    --cur;
    return *this;
}

self operator--(int) {//重载后置--
    self temp = *this;
    --*this;
    return temp;
}

// 如果有一次前进多个的情况，那么相较于上一种情况会更加复杂，需要考虑缓冲区之间的切换
self& operator+=(difference_type n) {  // 重载 +=
    difference_type offset = n + (cur - first);// 计算偏移后应该处于当前buf的第几个位置
    if (offset >= 0 && offset < buffer_size)//若仍在当前buf中
        cur += n;
    else//若不在同一个buf中 
    {
        //计算需要偏移几个buf节点(向前或向后)
        difference_type node_offset =
            offset > 0 ? difference_type(offset / buffer_size) : -difference_type((-offset - 1) / buffer_size) - 1;
        set_node(node + node_offset);//调整到正确的node
        cur = first + (offset - node_offset * buffer_size);//设置cur
    }
    return *this;
}
self operator+(difference_type n) const {//重载+
    self temp = *this;
    temp += n;
    return temp;
}
self& operator-=(difference_type n) { return *this += -n; }
self operator-(difference_type n) const {
    self temp = *this;
    temp -= n;
    return temp;
}
//重载下标访问
reference operator[](difference_type n) const { return *(*this + n); }

```



在操作符重载中，需要注意两个迭代器之间距离的计算

![在这里插入图片描述](E:\MarkDown\picture\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASU5saW5LQw==,size_20,color_FFFFFF,t_70,g_se,x_16-16684833765314.png)



deque的前进++和后退--操作需要额外判断是否超过当前buffer设定大小

![在这里插入图片描述](E:\MarkDown\picture\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASU5saW5LQw==,size_20,color_FFFFFF,t_70,g_se,x_16-16684834221176.png)



如果有一次前进多个的情况，那么相较于上一种情况会更加复杂，需要考虑缓冲区之间的切换

![image-20221115113724617](E:\MarkDown\picture\image-20221115113724617.png)

#### 容器适配器

> 每个适配器都定义两个构造函数：默认构造函数和接受一个容器的构造函数
>
> * 默认构造函数创建一个空对象
> * 接受一个容器的构造函数拷贝一个容器来初始化适配器
>   `stack<int> stk(deq);`

<img src="E:\MarkDown\picture\image-20220729172303502.png" alt="image-20220729172303502" style="zoom:40%;" />

**stack和queue**

> ​	stack栈和queue队列的内部实现都是直接用的deque。功能只是deque的子集。并且不提供iterator，因为会破坏容器先进后出、先进先出的性质。因此也没有find等算法

**1.queue和stack的实现**
queue和stack中含有一个deque，然后调用已经完成的deque来完成我们需要的操作

![image-20221115114915476](E:\MarkDown\picture\image-20221115114915476.png)

![image-20221115114924197](E:\MarkDown\picture\image-20221115114924197.png)

**2.queue和stack的异同**

相同：

1.不允许遍历，不提供迭代器
2.可以使用deque或者list作为底层结构。不过一般使用deque，因为deque更加快。

```cpp
queue<int, list<int>> q1;
for (long i = 0; i < 10; ++i) {
    q1.push(rand());
}

stack<int, list<int>> s1;
for (long i = 0; i < 10; ++i) {
    s1.push(rand());
}

stack<int, vector<int>> s2;
for (long i = 0; i < 10; ++i) {
    s2.push(rand());
}

```

不同：

queue不可以使用vector作为底层结构，而stack可以。

**使用**

![image-20230212095239161](E:\MarkDown\picture\image-20230212095239161.png)

**queue**

> 默认基于deque实现，也可以用list或vector实现

```cpp
#include<queue>
// queue先进先出
q.pop();  // 弹出首元素
q.front();
q.back();
q.push();  // 在末尾插入
q.emplace(args);
```

**stack**

![image-20220729190910810](E:\MarkDown\picture\image-20220729190910810.png)



**priority_queue**

优先队列

> 允许在O(lgn)时间复杂度下插入数据，在O(1)时间复杂度下取得容器内最大最小值
>
> 优先队列虽然叫队列，但是其本质，或者说其**底层实现却是堆**（一种特殊的二叉树）。这里的priority_queue基于vector(默认)或deque实现，指的是存储数值的容器为vector或deque，但获得元素的方式是通过堆弹出元素实现的，更底层为完全二叉树
>
> 默认实现是降序，即大根堆，队头到队尾单调递减



```cpp
#include<queue>
priority_queue<int> q;
q.top();  // 返回最高优先级元素，但不删除
q.pop();  // 弹出q的最高优先级的元素；或者说弹出堆顶
q.push(item);  // 插入
priority_queue<pair<int, int>> p;
p.emplace(nums[i], i);

priority_queue<Type, Container, Functional>
//greater和less是std实现的两个仿函数（就是使一个类的使用看上去像一个函数。其实现就是类中实现一个operator()，这个类就有了类似函数的行为，就是一个仿函数类了）。对于自定义的数据类型，需要重载<或者写仿函数
//升序队列，小顶堆。vector<int>指明了Container
priority_queue <int,vector<int>,greater<int> > q;

//降序队列。堆是一颗顺序存储的完全二叉树，大顶堆（大根堆）即每个结点的关键字都不小于其孩子结点的关键字。将大顶堆的顶点元素不停pop出来，pop的值就是一个降序数组
priority_queue<int> q;// 默认使用<确定相对优先级，即降序。对于自定义类型，若重载了<，则默认的形式也写成这样
priority_queue <int,vector<int>,less<int> >q;

priority_queue<pair<int, int>> q;
priority_queue<pair<int, int>, vector<pair<int, int>>, mycomparison> q;  // 与上个等价
q.emplace(nums[i], i);
cout << q.top().second;
```





### 关联式容器

![image-20221114185652348](E:\MarkDown\picture\image-20221114185652348.png)



![image-20221114185709207](E:\MarkDown\picture\image-20221114185709207.png)

对于unordered_multiset：

c.bucket_count() 输出篮子的个数。篮子一定比元素多，因为很多篮子都是空的，并且也是设计上的考量如果元素个数大于篮子，则对篮子进行两倍的扩充，并将元素再次打乱放到篮子里c.load_factor()载重因子，下一讲再讲。最大载重因子为1



#### 红黑树RB-Tree

> reference:https://blog.csdn.net/qq_34269632/article/details/116592934
>
> 红黑树的具体介绍在数据结构与算法.md中

​	因为红黑树是一颗BST平衡二叉搜索树，这样迭代器不应该修改那些已经排序插入的节点值。但是由于在C++中红黑树是作为set和map的底层，而map支持修改value，所以在C++中，红黑树没有阻止我们去修改节点值。
​	红黑树对外界提供了两种插入形式，`insert_unique()`和`insert_equal()`，前者代表key在这颗红黑树中是唯一的，否则插入失败；而后者表示节点的key可重复
​	对于`rb_tree`，定义一个概念：节点的`value`包括其`key`和`data`,这里的`data`表示一般说法中的`value`

<img src="E:\MarkDown\picture\image-20221116103719225.png" alt="image-20221116103719225" style="zoom:67%;" />

**红黑树的主类**

​	在主类中只通过header一根指针来对树进行关联，所有的操作都将借由这跟指针展开。
​	`rb_tree`的`header`指向一个多余的空节点,用以维持其前闭后开的特性。header的parent指向根节点，left指向最小的节点，right指向最大的节点，如下图所示：

<img src="E:\MarkDown\picture\image-20221116103908939.png" alt="image-20221116103908939" style="zoom:67%;" />

要点：红黑树主类rb_tree中的key对应key，而value对应pair类型的key-value的结合体

```cpp
// 就放了一部分在这

template<class Key,				// 指定key类型
         class Value,			// 指定Value类型。这里的value不是key-value的value，而是key和value的某种形式的结合体，如pair
         class KeyOfValue,		// 仿函数类,指定从Value中获取Key的方式
         class Compare,			// 仿函数类,指定Key的排序方式
         class Alloc = alloc>
class rb_tree {
protected:
    typedef _rb_tree_node <Value> rb_tree_node;  // 红黑树真正的结点，继承于_rb_tree_node_base
    typedef _rb_tree_node_base* base_ptr;  // 红黑树基层结点
public:
    typedef key key_type;
    typedef value value_type;
    typedef value* pointer;
    typedef value& reference;
    typedef rb_tree_node *link_type;
    typedef rb_tree_node& reference_type;
    typedef size_t size_type;
    typedef ptrdiff_t difference_type;
    typedef _rb_tree_iterator<value> iterator;  // 迭代器
protected:
    size_type node_count;	// rb_tree的大小(节点数量)
    link_type header;		// 头节点
    Compare key_compare; 	// Key的排序方式
};
```

**主类的结点和迭代器**

![image-20230404122551884](E:\MarkDown\picture\image-20230404122551884.png)

**_rb_tree_node_base**为基层结点，包含父节点和左右子树

```cpp
//全局的定义，用来储存红黑树的颜色属性
typedef bool color_type;//红黑树颜色的变量类型
const color_type red = false;//红色为0
const color_type black = true;//黑色为1

//红黑树节点的 基层节点
struct _rb_tree_node_base
{
	typedef _rb_tree_node_base*  base_ptr;
	
	_rb_tree_node_base()//构造函数
		:color(false), parent(nullptr), left(nullptr), right(nullptr) 
	{};

	color_type color;//节点的颜色
	base_ptr parent;//父节点
	base_ptr left;//左子树
	base_ptr right;//右子树

	//寻找最小值，由于二叉搜索树的最小值永远在左边，所以一直向左走就行
	static base_ptr minimum(base_ptr x) {
		while(x->left != nullptr)
			x = x->left;
		return x;
	}
	//寻找最大值，由于二叉搜索树的最大值永远在右边，所以一直向右走就行
	static base_ptr maximum(base_ptr x) {
		while(x->right != nullptr)
			x = x->right;
		return x;
	}
};
```

 **_rb_tree_node**为红黑树真正的结点，在基层节点基础上增加了一个数值域

​	**对于set来说，这个value既是数值也是键值，对于map来说，这应该是个由数值和键值组成的pair。**

```cpp
//红黑树的节点
template<class value>
struct _rb_tree_node : public _rb_tree_node_base
{//继承自基层节点
	typedef _rb_tree_node<value>* link_type;
	_rb_tree_node(value x = 0) : value_field(x) {};
	value value_field;//数值域
};
```

**_rb_tree_iterator_base** 红黑树迭代器的基类

​	这里使用一个指向红黑树基层节点的指针作为迭代器和容器之间的联系，并且实现了两个函数：increment()和decrement()，这两个函数的作用是将当前的迭代器移动到下一个更大(或上一个更小)的节点，这是为了让迭代器重载++和--的操作符使用。这个两个函数利用了红黑树主类中的一种定义，会在下边提及

```cpp
//红黑树迭代器的 基层节点
struct _rb_tree_iterator_base
{
	typedef _rb_tree_node_base::base_ptr base_ptr;
	typedef ptrdiff_t difference_type;

	//指向红黑树的一个节点
	base_ptr node;

	void increment()
	{//移动到下一个更大的节点
		if (node->right != nullptr) {//若当前节点存在右子树
			node = node->right;//从节点的右子树开始寻找
			while (node->left != nullptr)//找到右子树的最左节点
				node = node->left;
		}
		else//若没有右子节点
		{
			base_ptr y = node->parent;//找出父节点
			while (node == y->right)//若当前node是它父节点的右子节点
			{//一直上移，直到当前node是它父节点的左子节点
				node = y;
				y = y->parent;
			}
			if (node->right != y)//若此时node的右子节点等于他的父节点，则父节点为解答
				node = y;		//否则node为解答
		}
	}
	void decrement()
	{//移动到上一个更小的节点
		if (node->color == red && node->parent->parent == node)//如果是根节点或end（）时
			node = node->right;
		else if (node->left != 0)//若当前节点存在左子树
		{
			base_ptr y = node->left;//从左子树开始
			while (y->right != nullptr)//找到他的最右子树
				y = y->right;
			node = y;
		}
		else//既不是根节点，也没有左子树
		{
			base_ptr y = node->parent;//从父节点开始
			while (node == y->left)//直到当前节点node不是它父节点的左子节点
			{
				node = y;
				y = y->parent;
			}
			node = y;//此时的父节点就是解答
		}
	}
};
```

**_rb_tree_iterator** 迭代器主类

```cpp
//红黑树的迭代器
template<class value>
struct _rb_tree_iterator : public _rb_tree_iterator_base
{//继承自基层迭代器
	//定义型别
	typedef value value_type;
	typedef value& reference;
	typedef value* pointer;
	typedef _rb_tree_iterator<value> iterator;
	typedef _rb_tree_iterator<value> self;
	typedef _rb_tree_node<value>* link_type;

	//构造与析构
	_rb_tree_iterator() { node = nullptr; };
	_rb_tree_iterator(link_type x) { node = x; }
	_rb_tree_iterator(const iterator& x) { node = x.node; }

	//函数与重载
	reference operator*() const { return link_type(node)->value_field; }
	pointer operator->() const { return &(operator()); }
	self& operator++() { increment(); return *this; }
	self operator++(int) { self temp = *this; increment(); return temp; }
	self& operator--() { decrement(); return *this; }
	self operator--(int) { self temp = *this; decrement(); return temp; }
	bool operator!=(const self& x) const { return x.node != node; }
	bool operator==(const self& x) const { return x.node == node; }
};
```

**红黑树键值的查找**

```cpp
//查找给定的key值所在的位置
//返回指向的迭代器，若没找到目标则返回end()
template<class key, class value, class keyofvalue, class compare>
typename MyRbTree<key, value, keyofvalue, compare>::iterator//返回值
MyRbTree<key, value, keyofvalue, compare>::find(const key_type &v) {
	link_type x = header;
	link_type y = root();
	while(x != nullptr)//一直向下寻找，直到x走到尽头
		if(!key_compare(key(x),v))//若当前节点x的key值不小于（大于等于）给定的v
			y = x, x = left(x);//则令y等于当前节点，x移向左子树
		else
			x = right(x);//否则x移向右子树

	iterator j = iterator(y);
	return (j == end() || key_compare(k, key(j.node))) ? end() : j;
}
```



#### set/multiset

> https://en.cppreference.com/w/cpp/container/set

* 容器set和multiset以rb_tree为底层容器，因此其中元素是自动排序的，排序的依据是key。红黑树中的Value对应在map中是pair<key,value>，**对应在set和multiset中value和key一致**
* set和multiset提供迭代器iterator用以顺序遍历容器，但无法使用iterator改变元素值，因为set和multiset使用的是内部rb_tree的const_iterator。
* set元素的key必須独一无二，因此其insert()调用的是内部rb_tree的insert_unique()方法；multiset元素的key可以重复，因此其insert()调用的是内部rb_tree的insert_equal()方法
* 虽然C++有全局泛化的::find()函数，但是它的效率远远不如set中定义的set::find()，我们**应当尽量优先使用容器中定义的函数**
* **注意set.insert()函数，返回的是pair<iterator, bool>，set.insert(x).second这个bool值表示是否插入成功**

#### map multimap

* 容器map和multimap以rb_tree为底层容器，因此其中元素是有序的，排序的依据是key
* map和multimap提供迭代器iterator用以顺序遍历容器。无法使用iterator改变元素的key，但可以用它来改变元素的data，因为map和multimap内部自动将key的类型设为const
* map元素的key必須独一无二，因此其insert()调用的是内部rb_tree的insert_unique()方法；multimap元素的key可以重复，因此其insert()调用的是内部rb_tree的insert_equal()方法

![在这里插入图片描述](E:\MarkDown\picture\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASU5saW5LQw==,size_20,color_FFFFFF,t_70,g_se,x_16-16685673651872.png)

**map和multimap独有的[]运算符设计**

我们通过[]访问map/multimap，如果这个key不存在与map/multimap中，那么他会自动在map/multimap中创建并添加一个这个key对应的value的默认值，然后将其返回

`map`容器重载的`[]`运算符返回对应`data`的引用

```cpp
mapped_type& operator[](key_type&& __k)
{
    // 先查找key
    // 若key存在,则直接返回其data的引用
	iterator __i = lower_bound(__k);
    // 若key不存在,则先插入key,再返回对应的引用
    if (__i == end() || key_comp()(__k, (*__i).first))
        __i = _M_t._M_emplace_hint_unique(__i, std::piecewise_construct, std::forward_as_tuple(std::move(__k)), std::tuple<>());
    return (*__i).second;
}
```



### 无序容器

本质上是关联容器。底层为hashtable

#### hashtable

**基础概念**

​	哈希表是为了实现高效的**存储**以及高效**查找**而实现的。具体操作就是将我们需要存放的数据进行哈希运算之后得到哈希值，然后将哈希值取模，插入哈希表中对应的篮子（busket）中去

​	哈希表的长度是一个**质数**（在大于1的自然数中，除了1和它本身外不再有其他因数）；hashcode对应在哈希表数组中的位置是通过取余，即x mod M，M为哈希表长度，x为要保存的数经过哈希函数计算得到的hashcode

> 为什么要是质数？
>
> HASH函数需要把原始数据均匀地分布到HASH数组里，比如大部分是偶数，这时候如果HASH数组容量是偶数，容易使原始数据HASH后不会均匀分布：
>
> 2 4 6 8 10 12这6个数，如果对 6 取余 得到 2 4 0 2 4 0 只会得到3种HASH值，冲突会很多。如果对 7取余 得到 2 4 6 1 3 5 得到6种HASH值，没有冲突。
>
> 同样地，如果数据都是3的倍数，而HASH数组容量是3的倍数，HASH后也容易有冲突，用一个质数则会减少冲突的概率，更分散。

* **Separate Chaining**：当出现哈希碰撞时，将相同哈希值的节点组成一个链表挂在这个值对应的哈希值的后面。
* **Rehashing**：当哈希表中的总元素数量 >= 哈希表长度时，将哈希表的长度扩展到它两倍原本大小的最近的质数（不是vector的两倍扩容，而是寻找离它两倍大小值最近的一个质数，作为新的大小），然后将元素重新插入。



**插入操作举例**

​	hashtable 使用时，一开始是53个元素节点，因为咱上面28个质数中，最小的是53。也就是说 buckets vector 保留的是 53 个bucket，每个 bucket 是一个指针，初始值为 null。如果我们循序加入 6 个元素：59、63、108、2、53、55，于是 hashtable 变成如下图上方的样子，59 除 53 余数为 6，所以放到第六个bucket中，同理，63放第十个bucket中，而108,2,55都放在第二个 bucket 中。
​	如果，我们在插入48个元素（0~47），是总数达到54个元素，则超过了 buckets vector 的大小，符合表格重建的条件，hashtable扩充到第二个质数 97。然后排列如下图下部分所示，hashtable 进行了重整，bucket 2 和 bucket 11 的节点个数都是 2，其余的灰色bucket，节点个数都是1。白色 bucket 表示节点个数为0。

注意：**hashtable 中，键值相同的元素，一定落在用一个 bucket list 中**。

![在这里插入图片描述](E:\MarkDown\picture\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASU5saW5LQw==,size_20,color_FFFFFF,t_70,g_se,x_16-16685681830774.png)

`hashtable`最开始只有53个桶,当元素个数大于桶的个数时,桶的数目扩大为最接近当前桶数两倍的质数,实际上,桶数目的增长顺序被写死在代码里:

```cpp
static const unsigned long __stl_prime_list[__stl_num_primes] = {
        53, 97, 193, 389, 769, 1543, 3079, 6151, 12289, 24593,
        49157, 98317, 196613, 393241, 786433, 1572869, 3145739,
        6291469, 12582917, 25165843, 50331653, 100663319,
        201326611, 402653189, 805306457, 1610612741,
        3221225473ul, 4294967291ul};
```

**源码分析**

![image-20221116111446948](E:\MarkDown\picture\image-20221116111446948.png)



![在这里插入图片描述](E:\MarkDown\picture\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASU5saW5LQw==,size_20,color_FFFFFF,t_70,g_se,x_16-16685685018636.png)

​	类似deque，hashtable中是用**vector**当主控，存放的是node*类型的指针，这个指针指向一个_hashtable_node类型构成的链表，存放着val和下个结点的指针

**插入操作**

​	hashtable 的插入 跟 RB-tree 的插入类似，有两种插入方法 insert_unique 和 insert_equal ，意思也是一样的，insert_unique 不允许有重复值，而 insert_equal 允许有重复值。

**insert_unique**

插入时，新节点直接插入到链表的头节点，代码如下：

```cpp
pair<iterator, bool> insert_unique(const value_type& obj)
{
	resize(num_elements + 1);  // resize是另一个函数，其中包括是否重建Hashtable的判断等
	return insert_unique_noresize(obj);
}

pair<iterator, bool> insert_unique_noresize(const value_type& obj)
{
	const size_type n = bkt_num(obj);	//决定 obj 应位于 buckets 的那一个链表中
	node* first = buckets[n];

	//遍历当前链表，如果发现有相同的键值，就不插入，立刻返回
	for( node* cur = first; cur; cur = cur->next)
	{
		if( equals(get_key(cur->val), get_key(obj)) )
			return pair<iterator, bool>(iterator(cur, this), false);
	}
	
	//离开以上循环（或根本未进入循环）时，first指向 bucket 所指链表的头节点
	node* tmp = new_node(obj);		//产生新节点
	tmp->next = first;
	buckets[n] = tmp;				//令新节点为链表的第一个节点
	++num_elements;					//节点个数累加1
	return pair<iterator, bool>( iterator(tmp,this), true);
}
```

**insert_equal**

允许重复插入，需要注意的是插入时，**重复节点插入到相同节点的后面**，**新节点还是插入到链表的头节点**，代码如下：

```cpp
iterator insert_equal(const value_type& obj) {
	resize( num_elements + 1 ); //判断是否 需要重建表格，如需要就扩充
	return insert_equal_noresize(obj);
}

iterator insert_equalnoresize(const value_type& obj) {
	const size_type n = bkt_num(obj);	//决定 obj 应位于 buckets 的那一个链表中
	node* first = buckets[n];

    //遍历当前链表，如果发现有相同的键值，就马上插入，立刻返回
    for( node* cur = first; cur; cur = cur->next) {
    	if( equals(get_key(cur->val), get_key(obj)) ) {
    		node* tmp = new_node(obj);
    		tmp->next = cur->next;		//新节点插入当前节点位置之后
    		cur->next = tmp;
    		++num_elements;
    		return iterator(tmp, this);
    	}
    }
    	
    //运行到这里，表示没有发现重复的键值
    node* tmp = new_node(obj);		//产生新节点
    tmp->next = first;
    buckets[n] = tmp;				//令新节点为链表的第一个节点
    ++num_elements;					//节点个数累加1
    return iterator(tmp, this);
}
```



**哈希函数**

c++中为我们封装好了一些已有的哈希函数。

![在这里插入图片描述](E:\MarkDown\picture\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASU5saW5LQw==,size_20,color_FFFFFF,t_70,g_se,x_16-166856852104410.png)

![image-20221116112516961](E:\MarkDown\picture\image-20221116112516961.png)

**一个万用的哈希运算**

![在这里插入图片描述](E:\MarkDown\picture\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASU5saW5LQw==,size_20,color_FFFFFF,t_70,g_se,x_16-16691750190105.png)



#### **==hashtable 对比 R-B Tree==**

* map始终保证遍历的时候是按key的大小顺序的，默认有序，这是一个主要的功能上的差异。unordered_map无序。
* 时间复杂度上，红黑树的查找和删除性能都是O(logN)，是相对于稳定的，最差情况下都是高效的。
  而哈希表的查找和删除性能理论上都是O(1)，这有个前提就是哈希表不发生数据碰撞。在发生碰撞的最坏的情况下，哈希表的插入和删除时间复杂度最坏能达到O(n)。最坏情况就是所有的哈希值全部都在同一个链表上
* map可以做范围查找，而 unordered_map 不可以。
* 对于**内存占用**，红黑树占用的内存更小（仅需要为其存在的节点分配内存，每一个节点需要保存其父节点位置、孩子节点位置及红/黑性质，因此每一个节点占用空间大），而哈希事先就应该分配足够的内存存储散列表，是牺牲内存换取更快的查找速度。
  对于**内存效率**，unordered_map的散列空间会存在部分未被使用的位置，所以其内存效率不是100%的。而map的红黑树的内存效率接近于100%。
* Hash Table 由于建立了哈希表，在最开始建立的时候比较耗时间
* **扩容导致迭代器失效**。
  map的iterator除非指向元素被删除，否则永远不会失效。
  unordered_map的iterator在对unordered_map修改时有时会失效。因为在操作 unordered_map 容器过程（尤其是向容器中添加新键值对）中，一旦当前容器的负载因子超过最大负载因子（默认值为 1.0），该容器就会适当增加桶的数量（通常是翻一倍），并自动执行 rehash() 成员方法，重新调整各个键值对的存储位置（此过程又称“重哈希”），此过程很可能导致之前创建的迭代器失效。

适用场景：
	在**有顺序要求**的场合，肯定是要用RB-tree的；
	如果我们只操作一次，为了**保证最坏情况下的运行时间**，最好也适用RB-tree；
	而如果是需要经常操作，RB-tree肯定是没有hash_table快的。
因此，除了有**顺序要求和有单次操作时间要求的场景下用RB-tree**，其他场景都使用hash_table。

## 二、分配器(allocator)

> * 在内存分配中，分配器的主要目的是为了降低malloc产生的内存开销，具体见内存分配的笔记
> * 在STL中，分配器的主要目的是用来为容器存储的数据提供内存空间

​	容器中的数据都是存在动态分配的内存中的，栈中的容器只含有一些用来维护这些内存空间的指针。因为容器的这种特性，所以容器需要频繁的分配内存和回收内存。如果直接调用new或者malloc，其产生的空间和时间上的开销也是相当大的，而**allocator将内存分配和对象构造分离开来**，更适合容器使用

​	分配器的效率非常重要。因为容器必然会使用到分配器来负责内存的分配，它的性能至关重要。

![image-20221114190638104](E:\MarkDown\picture\image-20221114190638104.png)





## 三、迭代器

> ​	**泛型算法**之所以说是泛型算法，因为它并不是单独的为某一种容器去设计的，而是**可以供多种不同的容器去使用的算法**。为了满足这个需求，就需要对于算法来说，屏蔽掉容器的实现方法，但是却可以按照同一种顺序去访问大多数的容器，这就是**迭代器**的作用
>
> 迭代器是每一个容器都需要定义的一个class，它最起码具有以下特征：
>
> * 具有一个指向容器空间的指针成员
> * 重载了\*和->操作符，使这个class可以像指针一样去访问自己的指针成员
> * 重载了++操作符，通过++可以遍历整个容器
> * 定义了自己的五个标准型别



![image-20221114224131550](E:\MarkDown\picture\image-20221114224131550.png)

![image-20221114224238777](E:\MarkDown\picture\image-20221114224238777.png)

​	对于reference，从能否改变指向元素的内容这一角度看，迭代器分为不能改变内容的constant和可以改变内容的mutable。若一个指针p的value_type是T，如果他是mutable的，那么 * p 的类型就不应该是T而应该是T&，因为只有这样才可以保证他是一个可被改变的左值；如果p是constant的，那么* p的类型也应该是const T&

​	对于difference_type，在对迭代器或指针进行操作时，我们很有可能令两个指向同一个空间的指针相减，从而得到两个指针之间的距离，而difference_type型别的作用就是记录用什么样的元素去保存这个数值。若difference_type的类型是int，那么当两个指针之间的距离超过了int可能承载的数值时，就会出错。大多数情况下，我们都会使用头文件 xutility 中所定义的类型ptrdiff_t去储存两个原生指针之间的距离，所以在迭代器中，一般情况下也使用这个类型作为difference_type.

​	对于iterator_category：

### iterator_category（迭代器类型

> 使用类而非枚举来表示迭代器类型，是出于以下两个考虑：
>
> - 使用类的继承可以表示不同迭代器类型的从属关系
> - STL算法可以根据传入的迭代器类型调用不同版本的重载函
>
> ​	此外，迭代器的移动类型对于算法效率的影响是很深远的，例如：当我们需要在排序好的查找容器中的一个元素所在位置时，对于随即迭代器，我们可以采用二分查找，但是对于双向或者前序迭代器，我们只能遍历整个容器去查找。这两者最有情况下的时间复杂度分别是O(logN) 和 O(N)，效率差别还是挺大的。

​	共5种，用类表示，分别为**输入迭代器、输出迭代器、前向迭代器、双向迭代器、随机访问迭代器**

```cpp
//输入迭代器，不能移动
struct input_iterator_tag {};

//输出迭代器，不能移动
struct output_iterator_tag{};

//前序迭代器，单向移动
struct forward_iterator_tag : public input_iterator_tag{};

//双向迭代器，双向移动
struct bidirectional_iterator_tag : public forward_iterator_tag{};

//随机迭代器，双向移动，并且可以进行跳跃访问
struct random_access_iterator_tag : public bidirectional_iterator_tag{};
```

五种迭代器类型间的继承关系：

![image-20221121160546579](E:\MarkDown\picture\image-20221121160546579.png)



### **不同容器的迭代器类型**

![image-20221121162435098](E:\MarkDown\picture\image-20221121162435098.png)



​	容器array、vector、deque对使用者来说是连续空间,是可以跳跃的,其迭代器是随机访问迭代器 random_access_iterator 类型

​	容器list是双向链表,容器set、map、multiset、multimap本身是有序的,他们的迭代器都可以双向移动,因此是双向迭代器，bidirectional_iterator类型.

​	容器forward_list是单向链表，容器unordered_set、unordered_map、unordered_multiset、unordered_map哈希表中的每个桶都是单向链表。因此其迭代器只能单向移动，因此是forward_iterator类型

​	容器适配器stack、queue、priority_queue不支持迭代器

| iterator_category          | 简述                                                 | 容器                                                         |
| -------------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| forward_iterator_tag       | 仅单向前进，不支持operator--，可以反复读或写一个位置 | forward_list，unordered_set，unordered_map，unordered_multiset，unordered_multimap |
| bidirectional_iterator_tag | 双向，允许前进和后退                                 | list，set，map，multiset，multimap                           |
| random_access_iterator_tag | 允许访问随机下标                                     | array，vector，string，deque                                 |

​	另外有两种比较特殊，他们各自仅包含了一种迭代器。**输入迭代器是每个迭代位置只能被读一次的只读迭代器。输出迭代器是每个迭代位置只能被写一次的只写迭代器。**输入和输出迭代器被塑造为读和写输入和输出流(例如，文件)。因此并不奇怪输入和输出迭代器最通常的表现分别是istream_ iterator和ostream_iterator

| iterator_category   | 包含的迭代器     |
| ------------------- | ---------------- |
| input_iterator_tag  | istream_iterator |
| output_iterator_tag | ostream_iterator |





### （迭代器的）萃取器(iterator_traits)

> traits技术：[《C++ templates 2ed》](https://link.zhihu.com/?target=https%3A//book.douban.com/subject/30226708/)第19章，技术更新也更详细

​	顾名思义，萃取器是一定要从某件事物中提取出某些东西的。在STL中，Traits被广泛的用于泛型算法的优化上，他通过萃取出object的某些属性，对拥有特殊属性的object进行函数模板的偏特化，进而提高算法的效率。traits萃取器的目的是为了获得iterator所指向元素的type，这个名字取得就很形象，把iterator丢进去，把元素type萃取出来。

例如对于函数copy()，当我们需要复制一对迭代器之间的内容时，可能会出现多种情况：

* 若这对迭代器指向内嵌数据类型时，我们可以使用memmove( )去实现，效率相当高。
* 若这对迭代器指向某一个class时，我们就可能需要先为其分配空间，再去调用构造函数。

萃取器在这里面起到的作用就是用来提取出迭代器的类型，确定需要使用哪一种方法。

简单的说，traits就是用来提取出object的某一种属性的一个object



<img src="E:\MarkDown\picture\image-20221114230346875.png" alt="image-20221114230346875" style="zoom:80%;" />

1、为了使STL算法同时兼容迭代器和一般指针，就在迭代器(指针)和算法之间加一个中间层**萃取器**(traits)来区分，实际上就是利用了C++中**模板的偏特化**来进行一个区分，告诉算法调用的是指针还是迭代器。
	要知道，我们只能为自定义的class写typedef。那么，对于内嵌的object(比如原生的指针)呢？我们不可能为编辑器自带的object写typedef(除非修改源代码)，但是编辑器自带的object需要使用STL的泛型算法时就需要提供自己的typedef，这时就凸显了萃取器的作用。**STL中的萃取器通过模板的偏特化，为内嵌的object定义了属于他们自己的偏特化版本，只用来回复他们对应的问题；而对于自定义的class，就使用泛化的traits去询问class本身，从而取得对应的属性。**

![image-20230402230328180](E:\MarkDown\picture\image-20230402230328180.png)

2、对于迭代器，算法在用迭代器的时候，如sort(v.begin(), v.end())时是不知道里面元素的数据类型的，此时通过在萃取器中定义了五个类型，作为算法到迭代器的一个过渡，由迭代器提供这五个类型到iterator_traits即迭代器的萃取器中，然后萃取器再告知算法

![image-20221114224131550](E:\MarkDown\picture\image-20221114224131550.png)

![image-20221114224238777](E:\MarkDown\picture\image-20221114224238777.png)



```cpp
// 针对一般的迭代器类型,直接取迭代器内定义的关联类型
template<class I>
struct iterator_traits {
    typedef typename I::iterator_category 	iterator_category;
    typedef typename I::value_type 			value_type;
    typedef typename I::difference_type 	difference_type;
    typedef typename I::pointer 			pointer;
    typedef typename I::reference 			reference;
};

// 针对指针类型进行特化,指定关联类型的值
template<class T>
struct iterator_traits<T *> {
    typedef random_access_iterator_tag 		iterator_category;
    typedef T 								value_type;
    typedef ptrdiff_t 						difference_type;
    typedef T*								pointer;
    typedef T&								reference;
};

// 针对指针常量类型进行特化,指定关联类型的值
template<class T>
struct iterator_traits<const T *> {
    typedef random_access_iterator_tag 		iterator_category;
    typedef T 								value_type;		// value_tye被用于创建变量,为灵活起见,取 T 而非 const T 作为 value_type
    typedef ptrdiff_t 						difference_type;
    typedef const T*						pointer;
    typedef const T&						reference;
};


// 想要在算法内获取关联类型的值,只需像下面这样写，就可以获得迭代器iter的value_type的属性了。
template<typename T>
void algorithm(...) {
    typename iterator_traits<I>::value_type v1;
}

```

c++11中的type_traits

C++11在头文件`type_traits`中引入了一系列辅助类,这些辅助类能根据传入的模板参数自动进行获取该类的基本信息,实现类型萃取,并不需要我们为自己创建的类手动编写类型萃取信息

![请添加图片描述](E:\MarkDown\picture\20210328134926624.png)

```cpp
cout << "is_ void\t" << is_void<T>::value << endl;
cout << "is_ integral\t" << is_integral<T>::value << endl;
cout << "is_ floating point\t" << is_floating_point<T>::value << endl;
// ...
std::cout << std::is_same<const int, std::add_const<int>::type>::value << endl; //结果为true
```



## 四、算法Algorithm

> 算法实现建议看：[https://en.cppreference.com/w/cpp/header/algorithm](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/cpp/header/algorithm)

STL算法的一般形式如下:

```CPP
template<typename Iterator>
Algorithm(Iterator itr1, Iterator itr2) {
    // ...
}

template<typename Iterator, typename Cmp>
Algorithm(Iterator itr1, Iterator itr2, Cmp comp) {
    // ...
}
```

STL算法是看不到容器的，算法所需要的信息都是从迭代器取得的，因此迭代器内要存在与容器相关的信息，其中最重要的就是迭代器的5个关联类型

### 迭代器对算法的影响

#### 迭代器对算法的影响的四个实例

##### distance函数

![image-20221121163129566](E:\MarkDown\picture\image-20221121163129566.png)

##### advance(和distance近似

![image-20221121164114093](E:\MarkDown\picture\image-20221121164114093.png)

**copy**

copy用了很多次泛化和特化，除了iterator traits以外还用了type traits。
copy对其template参数所要求的条件非常宽松。其输入区间只需由inputIterators构成即可，输出区间只需要由OutputIterator构成即可。这意味着可以使用copy算法，将任何容器的任何一段区间的内容，复制到任何容器的任何一段区间上
![image-20221121164836641](E:\MarkDown\picture\image-20221121164836641.png)

##### destory(和copy近似

![image-20221121165033652](E:\MarkDown\picture\image-20221121165033652.png)



**迭代器的特殊情况**

![image-20221121171359481](E:\MarkDown\picture\image-20221121171359481.png)





## 算法实现



![image-20221121191529116](E:\MarkDown\picture\image-20221121191529116.png)

### accumulate

算法`accumulate`的默认运算是`+`，但是重载版本允许自定义运算，支持所有容器，源码如下:

![image-20221121192513060](E:\MarkDown\picture\image-20221121192513060.png)

```cpp
// 源码
template<class InputIterator, class T>
T accumulate(InputIterator first, InputIterator last, T init) {
    for (; first != last; ++first)
        init = init + *first;
    return init;
}

template<class InputIterator, class T, class BinaryOperation>
T accumulate(InputIterator first, InputIterator last, T init, BinaryOperation binary_op {
    for (; first != last; ++first)
        init = binary_op(init, *first);
    return init;
}

// 使用 
#include <iostream>     // std::cout
#include <functional>   // std::minus
#include <numeric>      // std::accumulate

// 自定义函数
int myfunc(int x, int y) { return x + 2 * y; }

// 自定义仿函数
struct myclass {
    int operator()(int x, int y) { return x + 3 * y; }
} myobj;

int main() {
	int init = 100;
    int nums[] = {10, 20, 30};

    cout << accumulate(nums, nums + 3, init);  				// 使用默认运算`+`,输出160
    cout << accumulate(nums, nums + 3, init, minus<int>()); // 使用仿函数指定运算`-`, 输出40
    cout << accumulate(nums, nums + 3, init, myfunc);    	// 使用自定义函数指定运算, 输出220
    cout << accumulate(nums, nums + 3, init, myobj);    	// 使用四定义仿函数,输出280
    return 0;
}        
```



### for_each

算法`for_each`支持所有容器,源码如下:

```cpp
template<class InputIterator, class Function>
Function for_each(InputIterator first, InputIterator last, Function f) {
    for (; first != last; ++first)
        f(*first);
    return f;
}
```

C++11中引入了新的**range-based for**语句,形式如下:
![image-20221121192825934](E:\MarkDown\picture\image-20221121192825934.png)

```cpp
// 自定义函数
void myfunc(int i) { cout << ' ' << i; }

// 自定义仿函数
struct myclass {
    void operator()(int i) { cout << ' ' << i; }
} myobj;

int main() {
    vector<int> myvec = {10, 20, 30};

    for_each(myvec.begin(), myvec.end(), myfunc);	// 使用自定义函数,输出10 20 30
    for_each(myvec.begin(), myvec.end(), myobj);	// 使用自定义仿函数,输出10 20 30

    // C++11引入的range-based for语法
    for (auto &elem : myvec)
        elem += 5;

    for (auto elem : myvec)
        cout << ' ' << elem;    	// 输出15 25 35
	return 0;
}
```



### replace, replace_if, replace_copy

* 算法replace将范围内所有等于old_value的元素都用new_value取代

* 算法replace_if将范围内所有满足pred()为true的元素都用new_value取代

* 算法replace_copy将范围内所有等于old_value的元素都以new_value放入新区间,不等于old_value的元素直接放入新区间

它们支持所有容器,源码如下:

```cpp
template<class ForwardIterator, class T>
void replace(ForwardIterator first, ForwardIterator last, const T &old_value, const T &new_value) {
    // 将范围内所有等于old_value的元素都用new_value取代
    for (; first != last; ++first)
        if (*first == old_value)
            *first = new_value;
}

template<class ForwardIterator, class Predicate, class T>
void replace_if(ForwardIterator first, ForwardIterator last, Predicate pred, const T &new_value) {
	// 将范围内所有满足pred()为true的元素都用new_value取代
    for (; first != last; ++first)
        if (pred(*first))
            *first = new_value;
}

template<class InputIterator, class OutputIterator, class T>
OutputIterator replace_copy(InputIterator first, InputIterator last, OutputIterator result, const T &old_value, const T &new_value) {
	// 将范围内所有等于old_value的元素都以new_value放入新区间,不等于old_value的元素直接放入新区间.
    for (; first != last; ++first, ++result)
        *result = (*first == old_value ? new_value : *first);
    return result;
}
```



### count, count_if

- 算法`count`计算范围内等于`value`的元素个数
- 算法`count_if`计算范围内所有满足`pred()`为`true`的元素个数

它们支持所有容器,但关联型容器(`set`、`map`、`multiset`、`multimap`、`unordered_set`、`unordered_map`、`unordered_multiset`和`unordered_map`)含有更高效的`count`方法,不应使用STL中的`count`函数.源码如下:

```cpp
// 注意，这里的typename为修饰，让编译器知道::前的iterator_traits<InputIterator>是类型，difference_type是类型成员
template<class InputIterator, class T>
typename iterator_traits<InputIterator>::difference_type 
count(InputIterator first, InputIterator last, const T &value) {
    // 计算范围内等于value的元素个数
    typename iterator_traits<InputIterator>::difference_type n = 0;
    for (; first != last; ++first) /
        if (*first == value)
            ++n;
    return n;
}

template<class InputIterator, class Predicate>
typename iterator_traits<InputIterator>::difference_type
count_if(InputIterator first, InputIterator last, Predicate pred) {
    // 计算范围内所有满足pred()为true的元素个数
    typename iterator_traits<InputIterator>::difference_type n = 0;
    for (; first != last; ++first)
        if (pred(*first))
            ++n;
    return n;
}
```



### find, find_if

- 算法`find`查找范围内第一个等于`value`的元素.
- 算法`find_if`查找范围内第一个满足`pred()`为`true`的元素.

它们支持所有容器,但关联型容器(`set`、`map`、`multiset`、`multimap`、`unordered_set`、`unordered_map`、`unordered_multiset`和`unordered_map`)含有更高效的`find`方法,不应使用STL中的`find`函数。源码如下：

```cpp
template<class InputIterator, class T>
InputIterator find(InputIterator first, InputIterator last, const T &value) {
    while (first != last && *first != value)
        ++first;
    return first;
}

template<class InputIterator, class Predicate>
InputIterator find_if(InputIterator first, InputIterator last, Predicate pred) {
    while (first != last && !pred(*first))
        ++first;
    return first;
}
```



### sort

算法sort暗示参数为random_access_iterator_tag类型迭代器，因此该算法只支持容器array、vector和deque

容器list和forward_list含有sort方法

容器set、map、multiset、multimap本身是有序的，容器unordered_set、unordered_map、unordered_multiset和unordered_map本身是无序的，不需要排序

```cpp
template<typename RandomAccessIterator>
inline void sort(RandomAccessIterator first, RandomAccessIterator last)
{
    // ...
}

// 使用
// 自定义函数
bool myfunc(int i, int j) { return (i < j); }

// 自定义仿函数(重载了()运算符的类
struct myclass {
    bool operator()(int i, int j) { return (i < j); }
} myobj;

int main() {
    
    int myints[] = {32, 71, 12, 45, 26, 80, 53, 33};
    vector<int> myvec(myints, myints + 8);          // myvec内元素: 32 71 12 45 26 80 53 33

    sort(myvec.begin(), myvec.begin() + 4);         // 使用默认`<`运算定义顺序,myvec内元素: (12 32 45 71)26 80 53 33
    sort(myvec.begin() + 4, myvec.end(), myfunc); 	// 使用自定义函数定义顺序,myvec内元素: 12 32 45 71(26 33 53 80)
    sort(myvec.begin(), myvec.end(), myobj);     	// 使用自定义仿函数定义顺序,myvec内元素: (12 26 32 33 45 53 71 80)
    sort(myvec.rbegin(), myvec.rend());				// 使用反向迭代器逆向排序,myvec内元素: 80 71 53 45 33 32 26 12
    return 0;
}
```

其中的`rbegin`和`rend`是迭代器适配器，生成一个逆向增长的迭代器

![image-20221121195331415](E:\MarkDown\picture\image-20221121195331415.png)

![image-20221121195641041](E:\MarkDown\picture\image-20221121195641041.png)

### binary_search

算法binary_search从排好序的区间内查找元素value，支持所有可排序的容器

算法binary_search内部调用了算法lower_bound，使用二分查找方式查询元素

算法lower_bound和upper_bound分别返回对应元素的第一个和最后一个可插入位置![image-20221121195708777](E:\MarkDown\picture\image-20221121195708777.png)

```cpp
template<class ForwardIterator, class T>
bool binary_search(ForwardIterator first, ForwardIterator last, const T &val) {
    first = std::lower_bound(first, last, val);		// 内部调用lower_bound
    // !(val < *first)这个判断的意思是排序是从小到大，val应大于*first
    return (first != last && !(val < *first));
}

template<class ForwardIterator, class T>
ForwardIterator lower_bound(ForwardIterator first, ForwardIterator last, const T &val) {
    ForwardIterator it;
    typename iterator_traits<ForwardIterator>::difference_type count, step;
    count = distance(first, last);
    while (count > 0) {
        it = first;
        step = count / 2;
        advance(it, step);
        if (*it < val) { // or: if (comp(*it,val)) for version (2)
            first = ++it;
            count -= step + 1;
        } else
            count = step;
        return first;
    }
}
```

## 五、仿函数functor

仿函数是一类重载了()运算符的类，其对象可当作函数来使用，常被用做STL算法的参数

STL的所有仿函数都必须继承自基类unary_function或binary_function，这两个基类定义了一系列关联类型，这些关联类型可被STL适配器使用。为了扩展性，我们自己写的仿函数也应当继承自这两个基类之一

```cpp
template<class Arg, class Result>
struct unary_function {
    typedef Arg argument_type;			// 关联类型1
    typedef Result result_type;			// 关联类型2
};

template<class Argl, class Arg2, class Result>
struct binary_function {
    typedef Argl first_argument_type;	// 关联类型1
    typedef Arg2 second_argument_type;	// 关联类型2
    typedef Result result_type;			// 关联类型3
};
```

前文程序中用到的几个仿函数的源码如下

```cpp
// 算术运算类仿函数
template<class T>
struct plus : public binary_function<T, T, T> {
    T operator()(const T &x, const T &y) const { return x + y; }
};

template<class T>
struct minus : public binary_function<T, T, T> {
    T operator()(const T &x, const T &y) const { return x - y; }
};

// 逻辑运算类仿函数
template<class T>
struct logical_and : public binary_function<T, T, bool> {
    bool operator()(const T &x, const T &y) const { return x && y; }
};

// 相对关系类仿函数
template<class T>
struct equal_to : public binary_function<T, T, bool> {
    bool operator()(const T &x, const T &y) const { return x == y; }
};

template<class T>
struct less : public binary_function<T, T, bool> {
    bool operator()(const T &x, const T &y) const { return x < y; }
}
```

![image-20221122100348276](E:\MarkDown\picture\image-20221122100348276.png)

## 六、适配器

适配器在STL组件的灵活组合运用功能上，扮演着轴承、转换器的角色
 STL所提供的各种适配器中：
 1）改变仿函数接口者，称为函数适配器；
 2）改变容器接口者，称为容器适配器；
 3）改变迭代器接口者，称为迭代器适配器。
对于函数适配器，适配器他也需要获得对应的仿函数一些信息。
![image-20221122102152430](E:\MarkDown\picture\image-20221122102152430.png)

### 容器适配器

STL提供3个容器适配器：queue和stack、priority_queue

注意，forward_list是容器

![image-20221122102254448](E:\MarkDown\picture\image-20221122102254448.png)

### 函数适配器

#### bind2nd

对于模板，我们知道：
 1.对于类模板，它必须指明类中元素的类型，而不能由类自己推导
 2.对于函数模板，它有能力自己推导传入的参数类型。

![image-20221122103434093](E:\MarkDown\picture\image-20221122103434093.png)

**函数适配器bind2nd使用**

```cpp
int ia[7] = { 27, 210, 12, 47, 109, 83, 40 };
vector<int,allocator<int>> vi(ia,ia+7);

// 通过算法count_if来使用适配器bind2nd，它的作用是绑定less<int>()仿函数比较的的第2个元素为40，即用less比较小于40的元素；
cout << count_if(vi.begin(), vi.end(), bind2nd(less<int>(), 40));
cout << endl; 
```

上述count_if的功能是计算容器vi中数值小于40的个数，count_if具体实现方式如下：

![image-20221122104400710](E:\MarkDown\picture\image-20221122104400710.png)

**bind2nd的实现方式**

```cpp
// 辅助函数，让user得以方便使用binder2nd<Op>
// 编译器会自动推导Op的type
template<class Operation, class T>
inline binder2nd<Operation> bind2nd(const Operation &op, const T &x) {
    typedef typename Operation::second_argument_type arg2_type;
    // 传给bind2nd函数的第二参数必须能够转为Operation的第二参数类型,否则报错
    return binder2nd<Operation>(op, arg2_type(x));
}
```



    a、bind2nd函数适配器它首先是一个模板函数，模板参数有两个分别是运算操作的类型(例子中是less<int>)和绑定的第二个值
    b、因less是继承binary_function类，故有second_argument_type的回答，即回答第二参数类型。而less传入的类型是int，所以第二参数类型是int,也就是这边arg2_type;
    c、而bind2nd函数本身没有什么操作，是通过binder2nd()来进行间接运算的。注意出入的第二参数arg2_type(x)是用来判断绑定值类型是否满足要求。
**binder2nd的实现方式**

![image-20221122104547815](E:\MarkDown\picture\image-20221122104547815.png)

    a、binder2nd是函数bind2nd适配的主体，里面包含两个数据op和value，分别是仿函数对象和less比较第二参数的类型(也通过回答的方式声明)；
    
    b、当bind2nd返回binder2nd函数对象时，binder2nd先进行构造函数，将传入的less<int>()对象和x(40)值，初始化赋值给op和value；
    
    c、当count_if算法处理pred(*first)时，又会调用binder2nd的仿函数operator()，此时传入less比较的第一参数，返回less比较函数。注意此时第二参数换成了value值（40)，正是此机制才使bind2nd函数才具有绑定第二参数的作用。
    
    d、这是G2.9的绑定函数，在头文件functional中。现在有一种新型的适配器bind实现多种绑定的功能。
    
    在binder2nd源码中,调用了Operation类的first_argument、second_argument_type和result_type,这些字段都是从STL仿函数基类binary_function继承得到的.因此我们自己写的仿函数也要继承自基类binary_function,才能使用适配器binder2nd进行增强.
    
    binder2nd适配器增强得到的仍然是一个仿函数,因此也要继承基类unary_function,以便被其它适配器增强.
#### `unary_negate`及其辅助函数`not1`

仿函数适配器`unary_negate`将仿函数的结果取反，生成新的仿函数

```cpp
// 仿函数适配器unary_negate也是仿函数类,因此继承自仿函数基类unary_function
template<class Predicate>
class unary_negate : public unary_function<typename Predicate::argument_type, bool> {
protected:
    // 内部成员,记录被取反的仿函数
    Predicate pred;		
public:
    // 构造函数使用explicit修饰,避免隐式类型转换 
    explicit unary_negate(const Predicate &x) : pred(x) {}		
	
    // 重载()运算符,将函数结果取反
    bool operator()(const typename Predicate::argument_type &x) const {
        return !pred(x);
    }
};

// 辅助函数，用not1调用unary_negate
template<class Predicate>
inline unary_negate<Predicate> not1(const Predicate &pred) {
    return unary_negate<Predicate>(pred);
}

cout << count_if(vi.begin()，vi.end ()，
                 not1(bind2nd(less<int>(), 40)));
```

![在这里插入图片描述](E:\MarkDown\picture\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASU5saW5LQw==,size_20,color_FFFFFF,t_70,g_se,x_16.png)

#### bind和占位符（C++ 11）

函数`bind`要和命名空间`std::placeholders`中的占位符`_1`、`_2`、`_3`等占位符配合使用

```cpp
// 源码如下
// 其中_1, _2, _3是未指定的数字对象，用于function的bind中。 _1用于代替回调函数中的第一个参数， _2用于代替回调函数中的第二个参数，以此类推
namespace placeholders {
  extern /* unspecified */ _1;
  extern /* unspecified */ _2;
  extern /* unspecified */ _3;
  // ...
}
```



`bind`函数可以绑定:

1. 函数和函数对象.
2. 成员函数(绑定成员函数时占位符`_1`必须是该类对象的地址).
3. 成员变量(绑定成员变量时占位符`_1`必须是该类对象的地址).

```cpp
#include <iostream>     // std::cout
#include <functional>   // std::bind

double my_divide(double x, double y) { return x / y; }

struct MyPair {
    double a, b;
    double multiply() { return a * b; }
};

int main() {
    // 必须带这条语句
    using namespace std::placeholders;    // 引入占位符_1, _2, _3,...

    // 将10和2绑定到函数的第一参数和第二参数上
    auto fn_five = std::bind(my_divide, 10, 2);               // returns 10/2
    std::cout << fn_five() << '\n';                          // 5

    // 将2绑定到函数的第e参数上
    auto fn_half = std::bind(my_divide, _1, 2);               // returns x/2
    std::cout << fn_half(10) << '\n';                        // 5

    // 将函数的第一参数和第二参数绑定到第二参数和第一参数上
    auto fn_invert = std::bind(my_divide, _2, _1);            // returns y/x
    std::cout << fn_invert(10, 2) << '\n';                    // 0.2

    // 将int绑定到函数的返回值上
    auto fn_rounding = std::bind<int>(my_divide, _1, _2);     // returns int(x/y)
    std::cout << fn_rounding(10, 3) << '\n';                  // 3

    
    ////////////////////////////////////////// 类
    MyPair ten_two{10, 2};

    // 将对象ten_two绑定到函数的第一参数上
    // z
    auto bound_member_fn = std::bind(&MyPair::multiply, _1); // returns x.multiply()
    std::cout << bound_member_fn(ten_two) << '\n';           // 20
	
    // 将对象ten_two绑定到函数的成员变量上
    auto bound_member_data = std::bind(&MyPair::a, ten_two); // returns ten_two.a
    std::cout << bound_member_data() << '\n';                // 10

    return 0;
}
```

### 迭代器适配器

#### 逆向迭代器`reverse_iterator`

容器的`rbegin()`和`rend()`方法返回逆向迭代器`reverse_iterator`，逆向迭代器的方向与原始迭代器相反。或者说可以通过一个双向顺序容器调用rbegin()和rend()来获取相应的逆向迭代器，前提是提供begin和end的双向容器

```cpp
class container{
public:
	reverse_iterator rbegin() { 
        return reverse_iterator(end()); 
    }
	
    reverse_iterator rend() {
        return reverse_iterator(begin()); 
    }
    
    // ...
}
```

![在这里插入图片描述](E:\MarkDown\picture\20210323125912962.png)

源码：
![image-20221123101609867](E:\MarkDown\picture\image-20221123101609867.png)



#### 用于插入的迭代器`insert_iterator`及其辅助函数`inserter`

迭代器适配器`insert_iterator`生成用于原地插入运算的迭代器,使用`insert_iterator`迭代器插入元素时，就将原有位置的元素向后推

```cpp
list<int> foo = {1, 2, 3, 4, 5};
list<int> bar = {10, 20, 30, 40, 50};

list<int>::iterator it = foo.begin();
advance(it, 3);  // 令迭代器向后移动三个元素

copy(bar.begin(), bar.end(), inserter(foo, it));
```

![请添加图片描述](E:\MarkDown\picture\20210323173940271.png)

`insert_iterator`通过重载运算符`=`、`*`和`++`实现上述功能。本质是调用容器的push_front、push_back、insert成员函数

```cpp
// 适配器类insert_iterator
template<class Container>
class insert_iterator {
protected:
    // 内部成员,记录底层容器和迭代器
    Container *container; 
    typename Container::iterator iter;
public:
    // 定义5个关联类型
    typedef output_iterator_tag iterator_category; 	
    
    insert_iterator(Container &x, typename Container::iterator i)
            : container(&x), iter(i) {}

    // 重载赋值运算符=
    insert_iterator<Container> &
	operator=(const typename Container::value_type &value) {
        iter = container->insert(iter, value); 		// 调用底层容器的insert
        ++iter; 									// 令insert_iterator永远随其target同步移动
        return *this;
    }
    
	// 重载运算符*和++: 不做任何动作
    insert_iterator<Container> &operator*() { return *this; }
    insert_iterator<Container> &operator++() { return *this; }
    insert_iterator<Container> &operator++(int) { return *this; }
};

// 辅助函数inserter，帮助用户使用insert_iterator
template<class Container, class Iterator>
inline insert_iterator<Container> inserter(Container &x, Iterator i) {
    typedef typename Container::iterator iter;
    return insert_iterator<Container>(x, iter(i));
}
```

#### 输出流迭代器`ostream_iterator`

输出流迭代器`ostream_iterator`常用于封装`std::cout`。**下面程序将容器中元素输出到**`std::cout`**中**

```cpp
int main() {
    std::vector<int> myvector = {10, 20, 30, 40, 50, 60, 70, 80, 90};
    std::ostream_iterator<int> out_it(std::cout, ",");
    std::copy(myvector.begin(), myvector.end(), out_it);
    return 0;
}
```

![在这里插入图片描述](E:\MarkDown\picture\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASU5saW5LQw==,size_20,color_FFFFFF,t_70,g_se,x_16.png)

#### 输入流迭代器`istream_iterator`

输入流迭代器`istream_iterator`用于封装`std::cin`，下面程序从`std::in`中读取数据：

```cpp
std::istream_iterator<double> eos;  // 标志迭代器,通过与该迭代其比较以判断输入流是否终止
std::istream_iterator<double> iit(std::cin);  // 封装std::cin的输入流迭代器

double value;
if (iit != eos) 
    value = *iit;		// 从输入流读取数据到变量value中,相当于: std::cin >> cvalue
```

`istream_iterator`重载了运算符`=`,`*`和`++`,其源码如下:

![在这里插入图片描述](E:\MarkDown\picture\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASU5saW5LQw==,size_20,color_FFFFFF,t_70,g_se,x_16-16691741177613.png)

下面程序使用输入流迭代器`istream_iterator`从`std::cin`中读取数据到容器中:

<img src="E:\MarkDown\picture\image-20221123113147255.png" alt="image-20221123113147255" style="zoom:50%;" />























