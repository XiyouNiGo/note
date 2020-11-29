[TOC]

const 和 constexpr 变量之间的主要区别在于：const 变量的初始化可以延迟到运行时，而 constexpr 变量必须在编译时进行初始化。

# 第三章 字符串、向量和数组

## 初始化方式

C++提供了几种不同的初始化方式，大多数情况下这些初始化方式可以互相等价地使用，不过也并非一直如此。

例外：

1. 使用拷贝初始化时（即使用=时），只能使用一个初始值（除非创建临时对象）。

   ```c++
   //使用一个初始值
   string s1 = "hello";
   //创建临时变量
   string temp(10, 'c');
   string s2 = temp;
   ```

2. 如果提供的是一个类内初始值们只能使用拷贝初始化或花括号的形式进行初始化。

3. 如果提供的是初始元素值的列表，则只能把初始值都放在花括号中进行列表初始化，而非圆括号。

   ```c++
   vector<string> v1{"a", "b", "c"};	//正确
   vector<string> v2("a", "b", "c");	//错误
   ```

# 第四章 表达式

## 显示转换

```c++
cast-name<type>(expreesion);
```

### static_cast

任何具有明确定义的类型转换，只要不包含地底层const，都可以使用static_cast。

### const_cast

const_cast只能改变运算对象的底层const。

```C++
const char *pc;
char *p = const_cast<char*>(pc);	//正确，但通过p写值是未定义的行为
```

### reinterpret_cast

reinterpret_cast通常为运算对象的位模式提供较低层次上的重新解释，本质依赖于机器。

### 旧式强转

如果换成const_cast和static_cast也合法，则其行为与对应的命名转换一致。如果替换后不合法，则旧式强制类型转换执行与reinterpret_cast类似的功能。

# 第六章 函数

## inline

inline默认具有静态链接，类似于static，是为了防止重复定义和性能的优化。

**inline函数必须跟申明放在同一个文件中，因为它是需要就地展开的。**       
如果你不把它放在.h中的话，那么在你调用这个函数时，只要包含这个文件能就编译成目标文件，但是在链接时，却会发现找不到这个函数体，而无法展开，从而造成链接失败。（找不到函数符号）

所谓内联函数，就是编译器将函数定义（{...}之间的内容）在函数调用处展开，藉此来免去函数调用的开销。如果这个函数定义在头文件中，所有include该头文件的编译单元都可以正确找到函数定义。然而，如果内联函数fun()定义在某个编译单元A中，那么其他编译单元中调用fun()的地方将无法解析该符号，因为在编译单元A生成目标文件A.obj后，内联函数fun()已经被替换掉，A.obj中不再有fun这个符号，链接器自然无法解析。

## 可变形参

### initializer_list

如果实参数量未知但是全部实参的类型都相同，我们可以使用initializer_list类型的形参。

拷贝或赋值一个initializer_list对象不会拷贝列表中的元素；拷贝后，原始列表和副本共享元素（浅拷贝）。

initializer_list对象中的元素永远是常量值。



initializer_list由编译器自动构造。

```c++
constexpr initializer_list(const _Elem *_First_arg,
		const _Elem *_Last_arg) noexcept
		: _First(_First_arg), _Last(_Last_arg)
		{	// construct with pointers
		}
```

`initializer_list<int> li = {100, 200, 300, 400, 500}`是怎么初始化的？

原理为：

1. 在栈上面分配一个数组。
2. 取到数组的第一个和最后一个地址的下一个（`ebp-18h`, `ebp-2Ch`).
3. 然后调用构造函数`initializer_list(const _Elem *_First_arg, const _Elem *_Last_arg) noexcept`。

### 省略符形参

省略符形参只能出现在形参列表的最后一个位置。

```c++
void foo(parm_list, ...);
void foo(...);
```

省略符形参所对应的实参无须类型检查。在第一种形式中，形参声明后面的逗号是可选的。

大多数类类型的对象在传递给省略符形参时都无法正确拷贝。

# 第七章 类

## 成员函数

成员函数的声明必须在类的内部，定义既可以放在内部也可以放在外部。

定义在类内部的函数是隐式的inline函数。

对于定义和声明分离的成员函数，其中之一声明为inline函数，则为inline函数（其实只是向编译器建议）。

定义的函数类似于 某个内置运算符时，应该令该函数的行为尽量模仿这个运算符（例如：+=则返回引用类型）。

虽然无须在声明和定义的地方同时说明inline，不过最好只在类外部定义的地方说明inline，这样可以使类更容易理解。

## 非成员函数

一些辅助函数，如add、read和pint等，与类相关但实际上并不属于类本身。这些函数在概念上属于类但是定义不在类中，一般与类的声明（而非定义）在同一个头文件中。

## this

指向对象自身的指针（所以是一个常量指针），在类的成员函数中默认包含。

不能显式定义this指针，非静态成员函数编译器会自动帮我们加上，静态函数则不会。

### 返回*this

返回*this时，可以用值也可以用引用去接收它，引用效率较高，切记不能返回本地变量的引用。

如果是值，返回值将是*this的临时副本，而不是该对象。在后续修改中也只是修改临时副本的值。

```c++
//用引用接收*this（效率高）
A& A::func()
{
	return *this;
}
//用值（对象）接收*this（返回临时副本）
A A::func()
{
	return *this;
}
```

### 动态绑定

如果是通过this指针调用，且向上转型，调用虚函数，此时为动态绑定。（通过对象调用为静态绑定，汇编代码中：call xxx）

动态绑定类似于：(*(p->vptr)[n])(p)

## const成员函数（常量成员函数）

把const放在成员函数的参数列表之后，此时的const表示this是一个指向常量的指针，因为this是一个指向常量的指针，所以常量成员函数不能改变调用它的对象的内容。

一个const成员函数如果以引用形式返回*this，返回类型将是常量引用，不能把它内嵌到一系列动作中。

```c++
myScreen.display(cout).set('*');	//错误
```

常量对象，以及常量对象的引用或指针都只能调用常量成员函数。（其实相当于const指针不能赋值给非const指针）

## 基于const的重载

通过区分成员函数是否是const的，我们可以对其进行重载，原因与我们之前根据指针参数是否指向const而重载函数的原因差不多。

因为非常量版本的函数对于常量对象是不可用的，所以我们只能在一个常量对象中调用const成员函数。另一方面，虽然可以在非常量对象上调用常量版本或非常量版本，但是显然此时非常量版本是一个更好的匹配。（对象是否是const决定了应该调用哪个版本）

## class with pointer

### 拷贝赋值函数

对于带pointer的类，拷贝赋值函数必须判断传入参数是否为本身。（避免野指针）

```c++
inline String& String::operator=(const String& str)
{
   if (this == &str)
      return *this;
   
   delete[] m_data;
   m_data = new char[strlen(str.m_data+1)];
   strcpy(m_data, str.m_data);
   return *this;
}
```

### 拷贝构造函数

**拷贝构造函数**是一种特殊的构造函数，它在创建对象时，是使用同一类中之前创建的对象来初始化新创建的对象。

拷贝构造函数通常用于：

1. 通过使用另一个同类型的对象来初始化新创建的对象。
2. 复制对象把它作为参数传递给函数。
3. 复制对象，并从函数返回这个对象。

如果在类中没有定义拷贝构造函数，编译器会自行定义一个。如果类带有指针变量，并有动态内存分配，则它必须有一个拷贝构造函数。

```c++
#include <iostream>
#include <cstring>

using namespace std;

class A
{
	friend void print(A a);
	friend A return_by_value();
public:
	A()
	{
		ret = new int;
		*ret = 0;
		cout << "无参构造函数" << endl;
	}
	A(int n)
	{
		ret = new int;
		*ret = n;
		cout << "带参构造函数" << endl;
	}
	//拷贝构造函数
	A(const A& a)
	{
		ret = new int;
		*ret = *a.ret;
		cout << "拷贝构造函数" << endl;
	}
	~A()
	{
		delete ret;
		cout << "析构函数" << endl;
	}
private:
	int *ret;
};

void print(A a)
{
	cout << *a.ret << endl;
}

A return_by_value()
{
	A temp(0);
	return temp;
}

int main(int argc, char **argv)
{
	A a(10);
	print(a);	//复制对象把它作为参数传递给函数
	A b(a);		//通过使用另一个同类型的对象来初始化新创建的对象(A b = a也一样)
	return_by_value();	//复制对象，并从函数返回这个对象
	return 0;
}
```



需要注意的是，拷贝构造函数不处理静态成员。

例如：定义一个静态成员作计数器，需要在拷贝构造函数中给该计数器++。



拷贝构造函数可以存在多个，编译器生成默认拷贝构造函数时只生成一个。

这个默认的参数可能为 A::A(const A&)或 A::A(A&),由编译器根据上下文决定选择哪一个，由编译器根据上下文进行选择。

```c++
class A
{    
public:
  A();    
  A(A&);
};    

const A ca;    
A a = ca;    // 错误
```



### array delete

对于array，delete必须使用[^delete\[\]  array]，并不是对象存在的空间出现问题，而是成员所指空间。

如果只使用[^delete array]，只对数组第一个元素调用析构函数。

### 某些类不能依赖于合成的版本

尽管编译器能替我们合成拷贝、赋值和销毁的操作（朴素的一个位一个位拷贝），但是对于某些类合成的版本无法正常工作，特别是当类需要分配类对象之外的资源时（例如一个成员指针指向一个堆区空间）。

如果类包含vector或者string成员，则其拷贝、赋值、销毁的合成版本能够正常工作。对vector成员的对象执行这些操作时，vector类会设法拷贝或赋值成员中的元素（标准库写好的）。

## =default

```c++
Sales_data() = default;
```

如果已经提供了一个构造函数，编译器不会自动生成默认的构造函数。如果需要默认构造函数，必须显示地把它声明出来。

定义这个构造函数的目的仅仅是因为我们需要其他形式的构造函数，它是一个默认构造函数。

在C++11新标准中，如果我们需要默认构造函数，可以在参数列表后面写上[^=default]，它既可以和声明一起出现在类的内部，也可以作为定义出现在类的外部（内部默认内联，外部默认不内联）。

## =delete

```c++
A(A& a) = delete;
```

作用：禁止使用编译器默认生成的函数（也可以定义成private）

delete 关键字可用于任何函数，不仅仅局限于类的成员函数。

## 构造函数初始值列表

```c++
Sales_data(const std::string& s):
	bookNo(s), units_sold(0), revenue(0) { };
```

通常情况下，构造函数使用类内初始值不失为一种好的选择。

不应该轻易覆盖掉类内的初始值，除非新赋的值与原值不同。

没有出现在构造函数初始值列表的成员将通过相应的类内初始值初始化，或者执行默认初始化。



## 访问控制（public、protected、private）

1. 类的一个特征就是**封装**，public和private作用就是实现这一目的。所以：

用户代码（类外）可以访问public成员而不能访问private成员；private成员只能由类成员（类内）和友元访问。

2. 类的另一个特征就是**继承**，protected的作用就是实现这一目的。所以：

protected成员可以被派生类对象访问，不能被用户代码（类外）访问。

### 关于继承

1. public继承：基类public成员，protected成员，private成员的访问属性在派生类中分别变成：public, protected, private

2. protected继承：基类public成员，protected成员，private成员的访问属性在派生类中分别变成：protected, protected, private

3. private继承：基类public成员，protected成员，private成员的访问属性在派生类中分别变成：private, private, private

## class和struct

使用class和struct定义类唯一的区别就是默认的访问权限。

class：private					struct：public

实例化对象时可以把类名跟在关键字class和struct后面。

## 友元

类可以允许其他类或者函数访问它的非公有成员，方法是令其他类或者函数成为它的友元。

```c++
friend Sales_data add(const Sales_data&, const Sales_data&);
```

一般来说，最好在类定义开始或结束前的位置集中定义友元。

友元的声明**仅仅指定了访问权限**，而非一个通常意义上的函数声明。如果我们希望类的用户能够调用某个友元函数，那么我们就必须在友元声明之外再专门对函数进行一次声明。（一些编译器允许在尚无友元函数的初始声明下就使用它）。

为了使友元对类的用户可见，我们通常把友元的声明与类本身放置在同一个头文件中（类的外部）。此外，友元函数能定义在类的内部（隐式内联）。

即使在类内部定义友元，我们也必须在类的外部提供相应声明从而使得函数可见。

### 类之间的友元关系

类还可以把其他的类定义成友元，也可以把其他类的成员函数定义友元。

友元类的成员函数可以访问此类包括private成员在内的所有成员。

友元关系不存在传递性（每个类负责控制自己的友元类或友元函数）。

### 函数重载与友元

如果一个类想把一组重载函数声明成它的友元，它需要对这组函数中的每一个分别声明。

## 封装的益处

1. 确保用户代码不会无意间破坏封装对象的状态。
2. 被封装的类的具体实现细节可以随时改变，而无须调整用户级别的代码。只要类的接口不变，用户的代码就无须改变。

## mutable（可变数据成员）

通过在变量的声明前加入mutable关键字，我们能够在const成员函数中修改类的某个数据成员。

```c++
class A
{
public:
	A(int ret) : ret(ret) {}
	A() = default;

	void setRet() const
	{
		ret = 1;
	}

private:
	mutable int ret;
};
```

## 建议：对公有代码使用私有功能函数

```c++
class Screen
{
public:
	//根据对象是否是const重载了display函数
	Screen& display(std::ostream& os)
	{
		do_display(os);
		return *this;
	}
	const Screen& display(std::ostream& os) const
	{
		do_display(os);
		return *this;
	}
private:
	string content;
	void do_display(std::ostream &os) const
	{
		os << content;
	}
};
```

为什么要费力定义一个单独的do_display函数？

原因：

1. 避免在多处使用相同的代码。
2. 随着类规模的发展，display函数有可能变得复杂。
3. 我们很可能在开发过程中给display函数添加某些调试信息，而这些信息将在代码的最终产品中去掉。
4. 这个额外的函数调用不会增加任何的开销（类内部定义函数被隐式的声明成内联函数）。

在实践中，良好的C++代码常常包含大量类似于do_display的小函数，通过调用这些函数，可以完成一组其他函数的“实际工作”。

## 不完全类型

像函数的声明和定义分离开一样，我们也可以仅仅声明类而暂时不定义它，这种声明有时被称为前向声明。

```c++
class Screen;
```

对于类型Screen，在它定义之前是一个不完全类型，也就是说，此时我们已知Screen是一个类类型，但是不清楚它到达包含哪些成员。

不完全类型只能在非常有限的情景下使用：可以定义指向这种类型的指针或引用，也可以声明（但是不能定义）以不完全类型作为参数或者返回类型的函数。因此类允许包含指向它自身类型的引用或指针（例如链表结点定义）。

对于一个类来说，我们创建它的对象之前该类必须被定义过，而不能仅仅被声明。

## 单例模式

### 基础要点

- 全局只有一个实例：static 特性，同时禁止用户自己声明并定义实例（把构造函数设为 private）
- 线程安全
- 禁止赋值和拷贝
- 用户通过接口获取实例：使用 static 类成员函数

### 实现

#### 有缺陷的懒汉式

懒汉式(Lazy-Initialization)的方法是直到使用时才实例化对象，也就说直到调用get_instance() 方法的时候才 new 一个单例的对象。好处是如果被调用就不会占用内存。

```C++
#include <iostream>
// version1:
// with problems below:
// 1. thread is not safe
// 2. memory leak

class Singleton{
private:
    Singleton(){
        std::cout<<"constructor called!"<<std::endl;
    }
    Singleton(Singleton&)=delete;
    Singleton& operator=(const Singleton&)=delete;
    static Singleton* m_instance_ptr;
public:
    ~Singleton(){
        std::cout<<"destructor called!"<<std::endl;
    }
    static Singleton* get_instance(){
        if(m_instance_ptr==nullptr){
              m_instance_ptr = new Singleton;
        }
        return m_instance_ptr;
    }
    void use() const { std::cout << "in use" << std::endl; }
};

Singleton* Singleton::m_instance_ptr = nullptr;

int main(){
    Singleton* instance = Singleton::get_instance();
    Singleton* instance_2 = Singleton::get_instance();
    return 0;
}
```

可以看到，获取了两次类的实例，却只有一次类的构造函数被调用，表明只生成了唯一实例，这是个最基础版本的单例实现，他有哪些问题呢？

1. **线程安全的问题**,当多线程获取单例时有可能引发竞态条件：第一个线程在if中判断 `m_instance_ptr`是空的，于是开始实例化单例;同时第2个线程也尝试获取单例，这个时候判断`m_instance_ptr`还是空的，于是也开始实例化单例;这样就会实例化出两个对象,这就是线程安全问题的由来; **解决办法**:加锁。
2. **内存泄漏**. 注意到类中只负责new出对象，却没有负责delete对象，因此只有构造函数被调用，析构函数却没有被调用;因此会导致内存泄漏。**解决办法**： 使用共享指针。（当 shared_ptr 析构的时候，new 出来的对象也会被 delete掉。以此避免内存泄漏）

#### magic static

```c++
#include <iostream>

class Singleton
{
public:
    ~Singleton(){
        std::cout<<"destructor called!"<<std::endl;
    }
    Singleton(const Singleton&)=delete;
    Singleton& operator=(const Singleton&)=delete;
    static Singleton& get_instance(){
        static Singleton instance;
        return instance;
    }
private:
    Singleton(){
        std::cout<<"constructor called!"<<std::endl;
    }
};

int main(int argc, char *argv[])
{
    Singleton& instance_1 = Singleton::get_instance();
    Singleton& instance_2 = Singleton::get_instance();
    return 0;
}
```

这种方法又叫做 Meyers' Singleton[Meyer's的单例](https://stackoverflow.com/questions/449436/singleton-instance-declared-as-static-variable-of-getinstance-method-is-it-thre/449823#449823)， 是著名的写出《Effective C++》系列书籍的作者 Meyers 提出的。所用到的特性是在C++11标准中的[Magic Static](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2660.htm)特性：

> If control enters the declaration concurrently while the variable is  being initialized, the concurrent execution shall wait for completion of the initialization.
>  如果当变量在初始化的时候，并发同时进入声明语句，并发线程将会阻塞等待初始化结束。

这样保证了并发线程在获取静态局部变量的时候一定是初始化过的，所以具有线程安全性。

[C++静态变量的生存期](https://stackoverflow.com/questions/246564/what-is-the-lifetime-of-a-static-variable-in-a-c-function) 是从声明到程序结束，这也是一种懒汉式。

**这是最推荐的一种单例实现方式：**

1. 通过局部静态变量的特性保证了线程安全 (C++11, GCC > 4.3, VS2015支持该特性);
2. 不需要使用共享指针，代码简洁；
3. 注意在使用的时候需要声明单例的引用 `Single&` 才能获取对象。

## 类的作用域

一个类就是一个作用域。在类的外部，成员的名字被隐藏起来了。

一旦遇到类名，定义的剩余部分就在类的作用域之内了。

当成员函数定义在类的外部时，返回类型必须指定它是哪个类的成员。（返回类型使用的名字都位于类的作用域之外）

```c++
Window_mgr::SceenIndex	//返回类型
Window_mgr::addScreen(const Screen& s)
{
	sceens.push_back(s);
	return sceens.size() - 1;
}
```

## 名字查找（name lookup）

1. 首先，在名字所在的块中寻找其声明语句，只考虑在名字的使用之前出现的声明。
2. 如果没找到，继续查找外层作用域。
3. 如果最终没有找到匹配的声明，则程序报错。

### 类的定义分两步处理

1. 首先，编译成员的声明。
2. 直到类全部可见后才编译函数体。

编译器处理完类中的全部声明后才会处理成员函数的定义。（简化类代码的组织方式）

### 类成员声明的名字查找

上面两阶段的处理方式都只适用**成员函数中使用的名字**（即定义中的）。声明中使用的名字，包括返回类型或者参数列表中使用的名字 ，都必须在使用前确保可见。

```c++
typedef double Money;
string bal;

class Account
{
public:
	Money balance()
	{
		return bal;
	}
private:
	Money bal;
}
```

该函数的return语句返回名为bal的成员。

一般来说，不建议使用其他成员的名字作为某个成员函数的参数，不要隐藏外层作用与中可能被用到的名字。

```c++
void Screen::dummy_fcn(pos height)
{
	cursor = width * ::height;	//显示的通过作用域运算符来进行请求
}
```



### 类型名要特殊处理

一般来说，内层作用域可以重新定义外层作用域中名字，即使该名字已经在内层作用域中使用过。然而在类中，如果成员使用了外层作用域中的某个名字，而该名字代表一种类型，则类不能在之后重新定义该名字。

类型名的定义通常出现在类的开始处，这样就能确保所有使用该类型的成员都出现在类名的定义后。

## 构造函数

### 构造函数初始值列表

```c++
ConstRef::ConstRef(int i1):i(i1), ci(i1), ri(i) {}
```

如果成员是const、引用，或者属于某种未提供默认构造函数的类类型，我们必须通过构造函数初始值为这些成员提供初始值。

建议养成使用构造函数初始值的习惯。

成员初始化顺序与他们在类定义中的出现顺序一致，初始值列表中初始值的前后位置关系不会影响实际的初始化顺序。

最好令构造函数初始值的顺序与成员声明的顺序保持一致。如果可能的话，尽量避免使用某些成员初始化其他成员。

### 委托构造函数

一个委托构造函数使用所属类的其他构造函数执行它自己的初始化过程。在委托构造函数内，成员初始值列表只有一个唯一的入口，就是类名本身。

```c++
Sales_data(std::string s): Sales_data(s, 0, 0) {}
```

### 转换构造函数

通过一个实参调用的构造函数定义了一条从构造函数的参数类型向类类型隐式转换的规则。

编译器只会自动地执行一步类型转换。

### explicit构造函数

在要求隐式转换的程序上下文中，我们可以通过把构造函数声明为explicit加以阻止。explicit构造函数只能用于直接初始化。

```c++
Sales_data item1(null_book);	//right
Sales_data item2 = null_book;	//wrong
```

关键字explicit只对一个实参的构造函数有效。需要多个实参的构造函数不能用于执行隐式转换，所以无须将这些构造函数指定为explicit的。

只能在类内声明构造函数时使用explicit关键字，在类外部定义时不应该重复。

尽管编译器不会将explicit构造函数用于隐式转换的过程，但是我们可以使用static_case<>显式地进行强制转换。

## 类的静态成员

与类本身直接相关，而不是与类的各个对象保持关联。

类的静态成员存在于任何对象之外，对象中不包含任何与静态数据成员有关的数据。类似的，静态成员函数也不与任何对象绑定在一起，它们不包含this指针。作为结果，静态成员函数不能声明为const的，而且我们也不能在static函数体内使用this指针。

我们可以使用作用域运算符直接访问静态成员，也可以使用类的对象、引用、指针来访问静态成员。

```c++
r = Account::rate();	//使用作用域运算符访问静态成员
r = ac1.rate();			//通过对象或引用
r = ac2->rate();		//通过指针
```

当在类的外部定义静态成员时，不能重复static关键字，该关键字只出现在类内部的声明语句。

### 静态成员的类内初始化

一般来说，我们不能在类的内部初始化静态成员。相反的，必须在类的外部定义和初始化每个静态成员。

然而，我们可以为静态成员提供const整数类型的类内初始值，不过要求静态成员必须是字面值常量类型的constexpr。

```C++
class Account
{
private:
	static constexpr int period = 30;	//为constexpr字面值常量类型静态成员提供const整数类型的初始值
	double daily_tbl[period];
};
```

如果某个静态成员的应用场景仅限于编译器可以替换它的值的情况（例如上面定义daily_tbl的维度），则一个初始化的const或constexpr static不需要分别定义。相反，如果我们将它用于值不能替换的场景中（例如取地址），则该成员必须有一条定义语句。

即使一个常量静态数据成员在类内部被初始化，通常情况下也应该在类的外部定义一下该成员。

### 静态成员和普通成员的区别

静态数据成员的类型可以就是它所属的类类型。而非静态成员只能声明成它所属的类的指针或引用。

```c++
class Bar
{
public:
	//***
private:
	static Bar mem1;	//正确，静态成员可以是不完全类型
	Bar *mem2;			//正确，指针成员可以是不完全类型
	Bar mem3;			//错误，数据成员必须是完全类型
};
```

另一个区别是我们可以使用静态成员作为默认实参。非静态数据成员不能作为默认实参，因为它的值本身属于对象的一部分。

# 第八章 IO库

## IO类型间的关系

概念上，设备类型和字符大小都不会影响我们要执行的IO操作。

标准库使我们能忽略这些不同类型的流之间的差异，这是通过继承机制实现的。

类型ifstream和istringstream都继承自istream。因此，我们可以像使用istream对象一样来使用他们。

## IO对象无拷贝或赋值

由于不能拷贝IO对象，因此我们也不能将形参或返回类型设置为流类型。进行IO操作的函数通常以引用方式传递和返回流。

读写一个IO对象会改变其状态，因此传递和返回的引用不能是const的。

## 管理输出缓冲

默认情况下，对cerr是设置unitbuf的，因此写到cerr的内容都是立即刷新的。

一个输出流可能被关联到另一个流。当读写被关联的流时，关联到的流的缓冲区会被刷新，例如，默认情况下，cin和cerr都关联到cout。因此，读cin或写cerr都会导致cout的缓冲区被刷新。

交互式系统通常应该关联输入流和输出流，这意味着所有输出，包括用户提示信息，都会在读操作**之前**被打印出来。

每个流同时最多关联到一个流，但多个流可以同时关联到一个ostream。

## unitbuf操纵符

如果想在每次输出操作后都刷新缓冲区，我们可以使用unitbuf操纵符。而nounitbuf操纵符则重置流。

```c++
cout << unitbuf;
cout << nounitbuf;
```

## 程序异常终止

如果程序崩溃秘书处缓冲区不会被刷新（正常return 0会自动刷新）。

当一个程序崩溃后，它所输出的数据很可能停留在缓冲区中等待打印。

## open

实际上，对一个已经打开的文件流调用open会失败，并会导致failbit被置位。随后试图使用文件流的操作都会失败。为了将文件流关联到另外一个文件，必须首先关闭已经关联的文件。

## 自动析构和构造

当一个fstream对象被销毁时，close会被自动调用。

## 文件模式

默认情况下，即使我们没有指定trunc，以out模式打开的文件也会被截断。

保留被ofstream打开的文件中已有数据唯一方法是显式指定app或in模式。

# 第九章 顺序容器

所有容器类都共享公共的接口，不同容器按不同方式对其进行拓展。

## 顺序容器概述

顺序容器为程序提供了控制元素存储和访问顺序的能力。这种顺序不依赖于元素的值，而是与元素加入容器时的位置相对应。

vector：可变大小数组。支持快速随机访问。在尾部之外的位置插入或删除元素可能很慢。

string：与vector相似的容器，但专门用于保存字符。

string和vector将元素保存在连续的内存空间中。

如果必须在中间位置插入元素，考虑在输入阶段使用list，一旦输入完成，将list中的内容拷贝到一个vector中。

容器均定义为模板类。

我们可以定义一个容器，其元素的类型是另一个容器。

虽然我们可以在容器中保存几乎任何类型，但某些容器操作对元素类型有其自己的特殊要求。我们可以为不支持特定操作需求的类型定义容器，但这种情况下就只能使用那些没有特殊要求的容器操作了。（例如某些类没有默认构造函数，我们在构造这种容器时不能只传递给它一个元素数目参数）

```c++
vector<noDefault> v1(10, init);	//正确，提供了元素初始化器
vector<noDefault> v2(10);		//错误，必须提供元素初始化器
```

所有容器都支持=和!=运算符，而无序关联容器不支持关系运算符。

标准库容器的所有迭代器都定义了递增运算符。

## 迭代器

### i++和++i

后置`++`要多生成一个局部对象 tmp，因此执行速度比前置的慢。同理，迭代器是一个对象，[STL](http://c.biancheng.net/stl/) 在重载迭代器的`++`运算符时，后置形式也比前置形式慢。在次数很多的循环中，`++i`和`i++`可能就会造成运行时间上可观的差别了。

```c++
CDemo CDemo::operator++ ()
{  
    //前置++    
    ++n;    
    return *this;
}

CDemo CDemo::operator ++(int k)
{  
    //后置++    
    CDemo tmp(*this);  
    //记录修改前的对象    
    n++;    
    //返回修改前的对象
    return tmp;
}
```

### 迭代器的功能分类

常用的迭代器按功能强弱分为输入、输出、正向、双向、随机访问五种。

对于两个随机访问迭代器 p1、p2，表达式`p2-p1`也是有定义的，其返回值是 p2 所指向元素和 p1 所指向元素的序号之差。

随机访问迭代器具有双向迭代器的全部功能。若 p 是一个随机访问迭代器，i 是一个整型变量或常量，则 p 还支持以下操作：

- p+=i：使得 p 往后移动 i 个元素。
- p-=i：使得 p 往前移动 i 个元素。
- p+i：返回 p 后面第 i 个元素的迭代器。
- p-i：返回 p 前面第 i 个元素的迭代器。
- p[i]：返回 p 后面第 i 个元素的引用。

双向迭代器不支持用“<”进行比较，也不支持+=，但是支持++。（因为不支持[]，[]又相当于*(+））

end可以和begin指向相同的位置（表示容器为空），但不能指向begin之前的位置。

只有顺序容器不包括（array）的构造函数才能接受参数大小。

### 迭代器的辅助函数

STL 中有用于操作迭代器的三个函数模板，它们是：

- advance(p, n)：使迭代器 p 向前或向后移动 n 个元素。
- dis[tan](http://c.biancheng.net/ref/tan.html)ce(p, q)：计算两个迭代器之间的距离，即迭代器 p 经过多少次 + + 操作后和迭代器 q 相等。
- iter_swap(p, q)：用于交换两个迭代器 p、q 指向的值。

要使用上述模板，需要包含头文件 algorithm。

### 迭代器失效

### vector迭代器失效

1. 当执行erase方法时，指向删除节点的迭代器全部失效，指向删除节点之后的全部迭代器也失效 。
2. 当进行push_back（）方法时，end操作返回的迭代器肯定失效。 
3. 当插入(push_back)一个元素后，capacity返回值与没有插入元素之前相比有改变，则需要重新加载整个容器，此时first和end操作返回的迭代器都会失效。
4. 当插入(push_back)一个元素后，如果空间未重新分配，指向插入位置之前的元素的迭代器仍然有效，但指向插入位置之后元素的迭代器全部失效。

### deque迭代器失效

1. 对于deque,插入到除首尾位置之外的任何位置都会导致迭代器、指针和引用都会失效，但是如果在首尾位置添加元素，迭代器会失效，但是指针和引用不会失效 。
2. 如果在首尾之外的任何位置删除元素，那么指向被删除元素外其他元素的迭代器全部失效 。
3. 在其首部或尾部删除元素则只会使指向被删除元素的迭代器失效。

### 关联式容器迭代器失效

对于关联容器(如map,  set,multimap,multiset)，删除当前的iterator，仅仅会使当前的iterator失效，只要在erase时，递增当前iterator即可。这是因为map之类的容器，使用了红黑树来实现，插入、删除一个结点不会对其他结点造成影响。erase迭代器只是被删元素的迭代器失效，但是返回值为void，所以要采用erase(iter++)的方式删除迭代器。 

map是关联容器，以红黑树或者平衡二叉树组织数据，虽然删除了一个元素，整棵树也会调整，以符合红黑树或者二叉树的规范，但是单个节点在内存中的地址没有变化，变化的是各节点之间的指向关系。

## 容器初始化

为了创建一个容器为另一个容器的拷贝，两个容器的类型及其元素类型必须匹配。不过，当传递迭代器参数来拷贝一个范围时，就不要求容器类型是相同的了，新容器和原容器中的元素类型也可以不同，只要能将要拷贝的元素转换为要初始化的容器的元素类型即可。

```c++
vector<const char*> articles = {"a", "an", "the"};
forward_list<string> words(articles.begin(), articles.end();	//正确，可以将const char*类型转为string
```

```c++
list<string> authors = {"A", "B", "C"};	//列表初始化
```

## array

当定义一个array时，除了指定元素类型，还要指定容器大小。

```c++
array<int, 42> arr;
```

虽然我们不能对内置数组类型进行拷贝或对象赋值操作，但array并无此限制（要求大小和元素类型相等）。

```c++
array<int, 10> digits = {0, 1, 2};
array<int, 10> copy1 = digits;
array<int, 5> copy2 = copy1;	//错误，大小不相等
```

## 赋值和swap

```c++
c1 = {a, b, c};
```

赋值运算后，左边容器将与右边容器相等。如果两个容器原来大小不同，赋值运算后两者大小都与右边的原大小相同。

```c++
swap(c1, c2);	//交换c1和c2中的元素。swap通常比从c2向c1拷贝元素快的多
c1.swap(c2);
```

## deque和vector

deque 是分段连续内存空间，有中央控制，维持整体连续的假象。

Vector是单向开口的连续线性空间，deque则是一种双向开口的连续线性空间。deque对象在队列的两端放置元素和删除元素是高效的，而向量vector只是在插入序列的末尾时操作才是高效的。deque和vector的最大差异，一在于deque允许于常数时间内对头端进行元素的插入或移除操作，二在于deque没有所谓的capacity观念，因为它是动态地以分段连续空间组合而成，随时可以增加一段新的空间并链接起来。换句话说，像vector那样“因旧空间不足而重新配置一块更大空间，然后复制元素，再释放旧空间”这样的事情在deque中是不会发生的。也因此，deque没有必要提供所谓的空间预留（reserved）功能。 

虽然deque也提供Random Access  Iterator，但它的迭代器并不是普通指针，其复杂度和vector不可同日而语，这当然涉及到各个运算层面。因此，除非必要，我们应尽可能选择使用vector而非deque。对deque进行的排序操作，为了最高效率，可将deque先完整复制到一个vector身上，将vector排序后（利用STL的sort算法），再复制回deque。 

deque是一种优化了的对序列两端元素进行添加和删除操作的基本序列容器。通常由一些独立的区块组成，第一区块朝某方向扩展，最后一个区块朝另一方向扩展。它允许较为快速地随机访问但它不像vector一样把所有对象保存在一个连续的内存块，而是多个连续的内存块。并且在一个映射结构中保存对这些块以及顺序的跟踪。

### deque的一些特点：

1. 支持随机访问，即支持[ ]以及at()，但是性能没有vector好。

2. 可以在内部进行插入和删除操作，但性能不及list。

3. deque两端都能够快速插入和删除元素，而vector只能在尾端进行。

4. deque的元素存取和迭代器操作会稍微慢一些，因为deque的内部结构会多一个间接过程。

5. deque迭代器是特殊的智能指针，而不是一般指针，它需要在不同的区块之间跳转。

6. deque可以包含更多的元素，其max_size可能更大，因为不止使用一块内存。

7. deque不支持对容量和内存分配时机的控制。

8. 在除了首尾两端的其他地方插入和删除元素，都将会导致指向deque元素的任何pointers、references、iterators失效。不过，deque的内存重分配优于vector，因为其内部结构显示不需要复制所有元素。

9. deque的内存区块不再被使用时，会被释放，deque的内存大小是可缩减的。不过，是不是这么做以及怎么做由实际操作版本定义。

10. deque不提供容量操作：capacity()和reverse()，但是vector可以。

    

    ![img](https://img-blog.csdnimg.cn/20190217155302649.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjQ2MjIwMg==,size_16,color_FFFFFF,t_70)

# 第十章 泛型算法

## for_each()

for_each事实上是个function template。

```c++
template<typename InputIterator, typename Function>
Function for_each(InputIterator beg, InputIterator end, Function f) {
  while(beg != end) 
    f(*beg++);
}
//使用
for_each(vec1.cbegin(), vec1.cend(), 
        [](int elem) 
        {
            cout << elem << "  "; 
    });
```

## lambda表达式



# 第十二章 动态内存

## 不要混合使用普通指针和智能指针

**shared_ptr可以协调对象的析构，但这仅限于其自身的拷贝之间。**

推荐使用make_shared而不是new来避免无意中将同一块内存绑定到多个独立创建的shared_ptr上，也可以避免临时变量（下面举例）。

```c++
void process(share_ptr<int> ptr) {}

//使用此函数的正确方法是给它一个share_ptr

share_ptr<int> p(new int(42));	//引用计数为1
process(p);	//拷贝p会递增它的引用计数
int i = *p;	//正确，引用计数为1

int *x(new int(1024));	//危险：x是一个普通指针
process(x);	//错误：接受指针的转换构造函数是explict的
process(shared_ptr<int>(x));	//合法，但内存会被释放（传入一个临时变量）
int i = *x;	//未定义的，x是一个空悬指针（此时临时变量析构一次，函数中变量析构一次，引用计数为0）
```

同理，也不要使用get初始化另一个智能指针或为智能指针赋值，使用get返回的指针的代码，不能delete此指针。

## reset成员经常与unique一起使用

```
if (!(p.unique()))
	p.reset(new string(*p));	//不是唯一用户，分配新的拷贝
*p += newVal;	//现在我们知道自己是唯一的用户，可以改变对象的值
```



## 使用自定义的释放操作

```c++
connection connect(desctination*);	//打开连接
void disconnect(connection);		//给定关闭连接

void end_connection(connection *p) { disconnect(*p); }

void f(destination &d)
{
	connection c = connect(&d);	//获得一个链接
	//如果我们在f退出前忘记调用disconnect，就无法关闭c了
	//使用shared_ptr来保证connection被关闭，已被证明是一种有效的方法
	shared_ptr<connection> p(&c, end_connection);
	//当f退出时（即使是由于异常退出），connection会被正确关闭
	//例如在此处抛出异常，且异常未被捕获
}
```

## unique_ptr

由于一个unique_ptr拥有它指向的对象，因此unique_ptr不支持普通的拷贝或赋值操作。

下面是基本操作：

```c++
u = nullptr;	//释放u指向的对象，将u置为空
u.release()		//u放弃对指针的控制权，返回指针，并将u置为空（并不释放u指向的对象）
    			//调用release会切断unique_ptr和原来管理对象的联系
u.reset() 		//释放u指向的对象，如果提供了内置指针q，令u指向这个对象，否则置空
```

可以通过调用release或reset将指针的所有权从一个（非const）unique_ptr转移给另一个unique_ptr。

```c++
unique_ptr<string> p2(p1.release());
```

不能拷贝unique_ptr的规则有一个例外：我们可以拷贝或赋值一个将要被销毁的unique_str。最常见的例子是从函数返回一个unique_ptr。

```c++
unique_ptr<int> clone(int p)
{
	return unique_ptr<int>(new int(p));
}
```

unique_ptr管理删除器的方式和shared_ptr1，重载一个unique_ptr中的删除器回影响到shared_ptr的类型以及如何构造该类型的对象。在创建这种类型的对象时，必须提供一个指定类型的可调用对象。

```c++
void f(destination& d)
{
	connection c = connect(&d);
	unique_ptr<connection, decltype(end_connection)*>
		p(&c, end_connection);	//decltype(end_connection)返回一个函数类型，所以必须添加一个*来指出我们正在使用一个指针
	//即使是由于异常退出，connection会被正常关闭
}
```

## weak_ptr

weak_ptr是一种不控制所指向对象生存期的只能指针，它指向一个由shared_ptr管理的对象，具有“弱”共享的特点。

将weak_ptr绑定到shared_ptr不会改变shared_ptr的引用计数，最后一个指向对象的shared_ptr被销毁，对象也会被释放。

由于对象可能不存在，我们不能使用weak_ptr直接访问对象，而必须调用lock。

```c++
w.expired;	//是否失效
w.lock();	//如果expire为true，返回空shared_ptr；否则返回指向w的对象的shared_ptr
```

在lock()成功时会延长shared_ptr对象的生命周期,因为它递增了一个引用计数。

weak_ptr被用于解决shared_ptr循环引用（两个类中有指向对方的shared_ptr）。

## 动态数组

为了让new分配一个对象数组，在类型名之后跟一对方括号，在其中指明要分配的对象的数目。方括号中的大小必须是整型，但不必是常量。

```c++
int *pia = new int[get_size()];	//pia指向第一个int
```

当用new分配一个数组时，我们并未得到一个数组类型的对象，而是得到一个数组元素的指针，因此不能对动态数组调用begin或end，也不能用范围for语句来处理。

动态分配一个空数组是合法的，但得到的指针不能解引用。（毕竟不指向任何元素）

为了释放动态数组，我们指针前加上一个空方括号。

```c++
delete p;		//p必须指向一个动态分配的对象或为空
delete[] pa;	//pa必须指向一个动态分配的数组或为空
```

数组中的元素按逆序销毁。如果忘记方括号，行为是未定义的（我的编译器只销毁第一个元素）。

### 智能指针和动态数组

为了用一个unique_ptr管理动态数组，我们必须在对象类型后面跟一对空方括号。

当一个unique_ptr指向一个数组时，我们可以下标运算符来访问数组中的元素。

```c++
unique_ptr<int[]> up(new int[]);	//销毁时自动调用delete[]而不是delete
```

shared_ptr不直接支持管理动态数组。如果希望使用shared_ptr管理一个动态数组，必须定义自己的删除器。

shared_ptr未定义下标运算符，并且不支持指针的算数运算。为了访问数组中的元素，必须使用get获取一个内置指针，然后用它来访问元素。

```c++
shared_ptr<int> sp(new int[10], [](int *p) { delete [] p;});	//lambda表达式
for (std::size_t i = 0; i != 10; ++i)
		*(sp.get() + i) = i;
```

## new的缺陷

new有一些灵活性上的局限，其中一方面表现在它将内存分配和**对象构造**组合在了一起。我们分配单个对象时，通常希望内存分配和**对象初始化**组合在一起。

一般情况下，将内存分配和对象构造组合在一起可能会导致不必要的浪费。（创建永远也用不到的对象、确实要使用的对象在默认初始化和赋值时被赋值了两次）

并且，没有默认构造函数的类就不能动态分配数组了。

## allocator类

标准库allocator类定义在头文件memory.h中，它帮助我们将内存分配和对象构造分离开来。它分配的内存是原始的、未构造的。

定义时必须指明这个allocator类可以分配的对象类型，它会根据给定的对象类型来确定恰当的内存大小和对齐位置。

在调用deallocate之前，用户必须对每个在这块内容中创建的对象调用destory。

为了使用allocate返回的内存，我们必须用construct构造对象。

```c++
allocator<string> alloc;	//可以分配string的allocator对象
	auto const p = alloc.allocate(n);	//分配n个未初始化的string
	auto q = p;
	//construct成员函数接受一个指针和0或多个额外参数，额外参数用来初始化构造的对象
	alloc.construct(q++);
	alloc.construct(q++, 10, 'c');
	cout << *q << endl;	//错误，q指向未构造的内存
	while (q != p)
		alloc.destroy(--q);
	alloc.deallocate(p, n);
```

两个伴随算法：

```c++
vector<string> v;
auto p = alloc.allocate(v.size() * 2);
auto q = uninitialized_copy(v.begin(), v.end(), p);	//拷贝，该函数返回尾后指针
uninitialized_fill_n(q, v.size(), "hhh");	//填充
```

# 第十五章 面向对象程序设计
