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

# 第六章 函数

## inline

inline默认具有静态链接，类似于static，是为了防止重复定义和性能的优化。

**inline函数必须跟申明放在同一个文件中，因为它是需要就地展开的。**       
如果你不把它放在.h中的话，那么在你调用这个函数时，只要包含这个文件能就编译成目标文件，但是在链接时，却会发现找不到这个函数体，而无法展开，从而造成链接失败。（找不到函数符号）

所谓内联函数，就是编译器将函数定义（{...}之间的内容）在函数调用处展开，藉此来免去函数调用的开销。如果这个函数定义在头文件中，所有include该头文件的编译单元都可以正确找到函数定义。然而，如果内联函数fun()定义在某个编译单元A中，那么其他编译单元中调用fun()的地方将无法解析该符号，因为在编译单元A生成目标文件A.obj后，内联函数fun()已经被替换掉，A.obj中不再有fun这个符号，链接器自然无法解析。

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

# 第十五章 面向对象程序设计

