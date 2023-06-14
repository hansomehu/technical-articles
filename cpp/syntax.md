

## Fundamentals



### Properties & Keywords



#### main函数

```cpp
#include <iostream>
using namespace std;
 
// main() 是程序开始执行的地方
 
int main()
{
   cout << "Hello World"; // 输出 Hello World
   return 0;
}
```



##### main的参数集合

main的参数 main(int argc, char* argv[], char* envp[])

**int argc**	argv中的参数数量计数

**char* argv[]**	命令行参数集合，argv[0]表示调用该main方法

**char* envp[]**	环境变量的参数集合



#### 头文件hpp



#### volatile

可以使用 **`volatile`** 限定符提供对异步过程（如中断处理程序）使用的内存位置的访问权。如果结构不具有可通过使用一个指令在当前体系结构上复制的长度，则此结构上可能完全丢失 **`volatile`**

**`volatile`**最终的效果也和所在机器的架构相关，ARM与非ARM架构在使用它的时候的最有方式有显著的区别

如果满足下列条件之一，则 **`volatile`** 关键字可能对字段不起作用：

- 可变字段的长度超过可使用一条指令在当前体系结构上复制的最大大小
- 最外层包含 **`struct`** 的长度 - 或如果它是可能嵌套的 **`struct`** 的成员 - 超过可使用一条指令在当前体系结构上复制的最大大小



#### 存储类

存储类定义 C++ 程序中变量/函数的范围（可见性）和生命周期。这些说明符放置在它们所修饰的类型之前



##### `extern`

让变量全局可见（在所有的翻译单元可见）

这里涉及到CPP的编译、链接体系，在CPP中每一个cpp文件是一个独立的翻译单元，根据hpp文件去寻找其依赖的资源。定义在其他cpp文件中的变量等可以理解为在不同文件中是相互隔离的。在大规模的开发中，很可能会产生变量重名的问题，在这种情况下会异常。尽管每个翻译单元在编译器是独立的，但是编译完成之后的链接期却是相互可见的，那么就可能导致在链接期出现重名异常

为了从一开始就杜绝这个问题，有两个方案，一个是将名称定义在namespace当中，另外一种就是使用external让其在编译器就是在各个翻译单元中可见



比如external int x，如果在链接期在全部的翻译单元中都没有找到x的定义的话，则会报错



##### `thread_local`

它不属于一种类型，只是一种对于变量等的修饰符

```cpp
thread_local int x;  // 命名空间下的全局变量
class X
{
    static thread_local std::string s; // 类的static成员变量
};
static thread_local std::string X::s;  // X::s 是需要定义的
 
void foo()
{
    thread_local std::vector<int> v;  // 本地变量
}
```



##### `static关键字`

1. 修饰的变量初始化值为0，并且在全局中具有value retain的效果，不会因为函数被二次调用而发生值重置的现象

2. 程序中的所有实例共享一个static变量，见下代码，只要static的值一发生改变则全局生效。不仅仅是在类中，在成员函数中该特性也依旧成立

3. 如果要强制一个全局名称具有内部链接，可以将它显式声明为 **`static`**。 此关键字将它的可见性限制在声明它的同一翻译单元内。 在此上下文中，**`static`** 表示与应用于局部变量时不同的内容。

   ```cpp
   // static2.cpp
   // compile with: /EHsc
   #include <iostream>
   
   using namespace std;
   class CMyClass {
   public:
      static int m_i;
   };
   
   int CMyClass::m_i = 0;
   CMyClass myObject1;
   CMyClass myObject2;
   
   int main() {
      cout << myObject1.m_i << endl;
      cout << myObject2.m_i << endl;
   
      myObject1.m_i = 1;
      cout << myObject1.m_i << endl;
      cout << myObject2.m_i << endl;
   
      myObject2.m_i = 2;
      cout << myObject1.m_i << endl;
      cout << myObject2.m_i << endl;
   
      CMyClass::m_i = 3;
      cout << myObject1.m_i << endl;
      cout << myObject2.m_i << endl;
   }
   ```



##### `auto`

自动类型推断与转换，它可以用来表示类型的占位符例如 `auto x = 3,14;` 这样auto在编译时会被推断为double



#### Template 模板

就是面向对象中的“泛型”概念，这方面的东西很多，如果要进一步深入C++的话这部分的东西是必须清楚了解的

<!--// todo 类模版、函数模版深入学习-->

##### `函数模板`

```c++
template <typename T>
T minimum(const T& lhs, const T& rhs)
{
    return lhs < rhs ? lhs : rhs;
}
```



##### `类模板`

```c++
template <typename T, typename U, typename V> class Foo{};

// 使用class代替typename是一样的效果
template <class T, class U, class V> class Foo{};

// “任意个数”类型泛型
template<typename... Arguments> class vtclass;

vtclass< > vtinstance1;
vtclass<int> vtinstance2;
vtclass<float, bool> vtinstance3;
```





#### Object Oriented Programming in C++

##### `基类`



##### `继承`



##### `抽象类（接口）`



#### Data Type

int

char

bool

##### `float`

##### `double`

单精度在小数点后有7位有效数字，而双精度有15位

单精度是4byte，双精度是8byte，需要额外一个bit来存储小数点位置和额外一个bit来存储正负值





### Expressions





#### i++与++i

从左往右来看这个问题，i++表示在运用完i当前的值之后才会进行+1的操作

而++i表示先将i的值+1再使用i进行对应的操作

```cpp
int i = 0;
int j = 0;
printf(i++); // output 0
printf(++j); // output 1
```



#### Functions Def

在CPP中所独有的函数声明方式



##### **`constexpr`**

指示函数的返回值是常量值，可以在编译时进行计算

```cpp
constexpr float exp(float x, int n)
{
    return n == 0 ? 1 :
        n % 2 == 0 ? exp(x * x, n / 2) :
        exp(x * x, (n - 1) / 2) * x;
};
```



##### **`inline`**

指示编译器将对函数的每个调用替换为函数代码本身。 在某个函数快速执行并且在性能关键代码段中重复调用的情况下，内联可以帮助提高性能

```cpp
inline double Account::GetBalance()
{
    return balance;
}
```



##### **`noexcept`** 

指定函数是否可以引发异常。 在以下示例中，如果 `is_pod` 表达式的计算结果 **`true`**为 ，函数不会引发异常。它是C++中函数的**“后缀”**签名，十分类似于Java中的throws Exception，但是C++中的noexcept可以根据一个constexpr的计算逻辑来最终确定是否允许抛出异常  

```cpp
#include <type_traits>

template <typename T>
T copy_object(T& obj) noexcept(std::is_pod<T>) {...}
```



##### `外部（cpp文件级别）函数调用`

```cpp
// C++上机作业.cpp : 定义控制台应用程序的入口点。
//'0-9': 48-57
 
#include "stdafx.h"
using namespace std;
extern void gotoxy(short x, short y); // 这样声明本文件以外的函数
extern void sort_by_name();
extern int Strtoint();
 
int main()
{
	system("title 功能主函数");
	gotoxy(23, 2); cout << "功能列表";
	gotoxy(15, 3); cout << "1:字符串转换为数值类型";
	gotoxy(15, 4); cout << "2:对中文字符进行排序";
 
    return 0;
}
```



#### Exception Handling

##### `C++中内置异常一览图`

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230516162113678.png" alt="image-20230516162113678" style="zoom: 33%;" />

##### **`C++异常处理实践`**







### Pointers

> C++的指针



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230518113117051.png" alt="image-20230518113117051" style="zoom:50%;" />

在数组中，**`&arr_name`**是代表整体而**`*arr_name`**是代表首元素



在形参数中如果用的*指针，那么在调用传递实参的时候需要使用取址符来对其传入地址

```cpp
int a[5] = {0, 0, 0, 0, 0};

arr_demo2(*&a);
arr_demo2(a);

void arr_demo2(int *arr){
    cout << arr[0] << endl;
}
```

一维数组本质是一个指向首元素的地址，所以int *var在实际传入的时候可以穿入一个数组引用arr，也可以 *&arr，它们本质是相同的，都是取首位的地址

指针也就是在和数组一起使用的时候特殊一些，在C++中，一个定义的数组arr它所指代的是数组空间首位元素的地址，本质是以恶搞地址，所以和int *var是等价的    





void关键字与指针



野指针



##### cosnt关键字与指针

常量指针

指针常量

指针常量指针

上面三个概念都是根据指针的值（地址）和其指向的数据的可变性与否进行分类的，但是开发中会用到的基本只是“常量指针”用来指示一个特定类型数据的起始地址



二级指针



空指针问题



##### 指针与数组

在C++的编译器中，数组就当作地址来进行处理的，所以和指针本质上并无区别（大多数情况下）

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230521112556978.png" alt="image-20230521112556978" style="zoom: 25%;" />



数组的栈分配与堆分配



##### 行指针与二维数组

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230521143334097.png" alt="image-20230521143334097" style="zoom: 25%;" />



三维数组用行指针表示 int(*p) 【2】【3】 表示指向单元为两行三列的矩阵的三维数组



##### 函数指针（辨析其与指针函数的关系）

意义在于可以在通用函数中通过参数的方式传入个性函数，实现通用函数的具体化

在C++中函数名字所指代的就是函数的地址

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230521144745042.png" alt="image-20230521144745042" style="zoom: 25%;" />



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230521144933851.png" alt="image-20230521144933851" style="zoom: 25%;" />





#### 



## Deep Down C++

> 这一章主要是编程中用不到的偏C++底层/计算机硬件的知识





<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230520141610957.png" alt="image-20230520141610957" style="zoom: 25%;" />

主要关注栈和堆的异同：

