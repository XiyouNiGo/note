[TOC]

# 第二章 变量和基本类型

## 引用

定义引用时，程序把引用和它的初始值绑定在一起。因为无法令引用重新绑定到另一个对象，因此引用必须初始化。

引用并非对象，它只是为一个已经存在的对象所起的另一个名字。因为引用不是对象，所以不能定义引用的引用。

引用类型的初始值必须是一个对象。

### 指针和引用

指针本身就是一个对象，允许对指针赋值和拷贝，而且在指针的生命周期内它可以先后指向几个不同的指针。

指针无须在定义时赋初值。

多数编译器用指针实现引用。

### 指向指针的引用

```c++
int *p;
int *&r = p;
```

最简单的办法是从右向左阅读r的定义。

离变量名最近的声明符对变量的类型有最直接的影响。

## 空指针

在新标准下，最好使用nullptr，同时避免使用NULL。（主要在函数重载那里体现出来）

nullptr是一种特殊类型的字面值，它可以被转换成任意其他的指针类型。

预处理变量NULL：在头文件cstdlib中定义，它的值就是0。

预处理变量不属于命名空间std，它由预处理器负责管理，使用前无须加上::std。

把int类型变量直接赋值给指针是错误的操作，即使ubt变量额度值恰好等于0。

```c++
int zero = 0;
int *p1 = 0;	//正确
int *p2 = zero;	//错误
```

不能直接操作void*指针所指对象，因为我们并不知道这个对象到底是什么类型，也就无法确定能在这个对象上做哪些操作。

## 复合类型

一条声明语句由一个基本数据类型和一个声明符列表组成（只能有一个基本类型）。

强调复合类型：将声明符和变量连在一起。

```c++
int *p1, p2;	//p1:int* 		p2:int
```

如果某个类型名指代的是复合类型或常量，用到声明语句中可能有意想不到的后果。

```c++
using pstring = char*;
const char *q = 0;		//基本数据类型为char，*为声明符，常量指针
const pstring q = 0;	//基本数据类型为char*，指针常量
```



## const

const对象一旦创建后其值就不能改变，所以const对象必须初始化。（声明的时候可以不用，例如：extern const int i）

默认状态下，const对象仅在文件内有效，当多个文件中出现了同名的const变量，等同于在不同文件中分别定义了变量。（如果没有const会报错重复定义）

如果想在多个文件中共享const对象，必须在变量的定义之前添加extern。

### 常量引用

引用的类型必须与引用对象的类型一致，但是有两个例外：

1. 初始化常量引用时允许用任意表达式作为初始值，只要该表达式的结果能够转换成引用的类型即可。
2. 允许为一个常量引用绑定非常量的对象、字面值、甚至是一般表达式。

常量引用可能引用一个非const的对象。

### 顶层const

顶层const：指针本身是一个常量（也可以表示任意对象是一个常量）

底层const：指针所指对象是一个常量

用于声明引用的const都是底层const（引用本身无法改变，类似于顶层const）

## constexpr

C++11新标准规定，允许将变量声明为constexpr来验证变量（字面值类型）是否是一个**常量表达式**（值不会改变且编译时期就能得到计算结果的表达式）。

字面值类型：简单，值显而易见，容易得到（算数类型、引用、指针）。

一个constexpr指针的初始值必须是nullptr、0或存储于某个固定地址的对象（定义于函数体之外的对象）。

const 和 constexpr 变量之间的主要区别在于：const 变量的初始化可以延迟到运行时，而 constexpr 变量必须在编译时进行初始化。

在constexpr声明中如果定义了一个指针，限定符constexpr只对指针有效（毕竟是对字面值类型而言...）。

```c++
const int *p = nullptr;			//常量指针
constexpr int *q = nullptr;		//指针常量
```

## auto

auto让编译器通过初始值来推算变量的类型，auto定义的变量必须有初始值。

编译器以引用类型哪作为auto的类型。

auto一般忽略顶层const(除了引用），保留底层const。顶层const需明确指出。

设置一个类型为auto的引用，初始值中的顶层const属性仍然保留。

```c++
const int ci = 0, &cr = ci;
auto b = ci;	//忽略顶层const
auto e = &ci;	//底层const（对const对象取地址是一种底层const）
auto &m = ci, *p = &ci;	//m：对常量的引用，p：常量指针
```

## decltype

分析表达式并得到它的类型（包括顶层const和引用），却不计算实际表达式的值。

引用作为其所指对象的同义词，只有用在decltype处是个例外。

如果表达式的内容是解引用操作，decltype将得到引用类型。

给变量加上一层括号，decltype得到引用。（变量是一种可以作为赋值语句左值的特殊表达式，会被当成引用）

```c++
decltype(*p);		//解引用将得到引用，p为int&
decltype((i)) q;	//双重括号的结果永远是一个引用
decltype(a = b) c;	//表达式都返回引用？
```

当用decltype来获得一个函数指针类型时，必须加上一个*。

当我们使用函数名字时，在需要的情况下会转化成一个指针。

```c++
multiset<Sales_data, decltype(compareIsbn)*>
	bookstore(compareIsbn);	//&compareIsbn也可以
```

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

**reinterpret_cast**的能力仅次于类C转换。虽然它**约束了整型、浮点和枚举类型的相互转换**，但是还是支持指针和整型的转换。它也**存在转换后运行时出错的隐患**。reinterpret_cast通常为运算对象的位模式提供较低层次上的重新解释，本质依赖于机器。

**static_cast弥补了reinterpret_case对整型、浮点和枚举类型的相互转换的功能。**除了这些转换外，它**要求操作的参数是指针**。且已经出现C++特性限制，**要求指针转换时的类存在继承关系（void\*除外）**。它也**存在转换后运行时出错的隐患**。隐式转换是static_cast。

**dynamic_cast**已经是纯的C++特性转换，**使用到了RTTI技术**。于是它**要求操作的指针类型具有多态特性**。它**解决了指针转换后使用出现运行时出错的问题**（出错返回nullptr，可能抛出std::bad_cast**）使用该方法要付出运行时计算的代价**。**如果能明确转换是安全的，建议使用static_cast方法（不使用reinterpret_cast是因为它还没体现出C++的特性）。**

dynamic_cast只可以用于指针之间的转换，它还可以将任何类型指针转为无类型指针，甚至可以在两个无关系的类指针之间转换。

**const_cast用于增加和去掉一些类型描述标志**，如const等。

### 旧式强转

如果换成const_cast和static_cast也合法，则其行为与对应的命名转换一致。如果替换后不合法，则旧式强制类型转换执行与reinterpret_cast类似的功能。

# 第六章 函数

## 默认实参

默认实参负责填补函数调用缺少的**尾部**实参（靠右位置）。

尽量让不怎么使用默认值的形参出现在前面，而让那些经常使用默认值的形参出现在后面。

如果一组重载函数（可能带有默认参数）都允许相同实參个数的调用，将会引起调用的二义性。

默认值可以是全局变量、全局常量，甚至是一个函数。默认值不可以是局部变量，因为**默认参数的函数调用是在编译时确定的**，而局部变量的位置与值在编译时均无法确定。

**默认参数在函数声明中提供**，当又有声明又有定义时，定义中不允许默认参数。如果函数只有定义，则默认参数才可出现在函数定义中。在给定的作用域中一个形参只能被赋予一次默认实参。

## inline

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

## 尾置返回类型

```c++
auto func(int i) -> int(*)[10] 
```

对于更复杂的定义，使用decltype更好。

## 函数重载

- `main`函数不能重载。
- **重载和const形参**：
  - 一个有顶层const的形参和没有它的函数无法区分。 `Record lookup(Phone* const)`和 `Record lookup(Phone*)`无法区分。
  - 相反，是否有某个底层const形参可以区分。 `Record lookup(Account*)`和 `Record lookup(const Account*)`可以区分。
- **重载和作用域**：在不同的作用域中无法重载函数名。

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

如果是通过this指针调用，且向上转型（基类指针或基类引用），调用虚函数，此时为动态绑定。（通过对象调用为静态绑定，汇编代码中：call xxx）

动态绑定类似于：(*(p->vptr)[n])(p)

## const成员函数（常量成员函数）

把const放在成员函数的参数列表之后，此时的const表示this是一个指向常量的指针，因为this是一个指向常量的指针，所以常量成员函数不能改变调用它的对象的内容。

一个const成员函数如果以引用形式返回*this，返回类型将是常量引用，不能把它内嵌到一系列动作中。

```c++
myScreen.display(cout).set('*');	//错误
```

常量对象，以及常量对象的引用或指针都只能调用常量成员函数。（其实相当于const指针不能赋值给非const指针）

## 基于const的重载

通过区分成员函数是否是const**（只能是底层const）**，我们可以对其进行重载，原因与我们之前根据指针参数是否指向const而重载函数的原因差不多。

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

如果是内置类型或只有默认析构函数的类型，delete一个new []的对象不会出问题。

delete对new，delete []对new []。

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
2. 被封装的类的具体实现细节可以随时改变，而无须调整用户级别的代码。只要类的接口不变，用户的代码就无须改变。+

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

不完全类型只能在非常有限的情景下使用(static例外）：可以定义指向这种类型的指针或引用，也可以声明（但是不能定义）以不完全类型作为参数或者返回类型的函数。因此类允许包含指向它自身类型的引用或指针（例如链表结点定义）。

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
3. **注意在使用的时候需要声明单例的引用 `Single&` 才能获取对象**。

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

1. 首先，在名字所在的块中寻找其声明语句，**只考虑在名字的使用之前出现的声明。**
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

一般来说，内层作用域可以重新定义外层作用域中名字，即使该名字已经在外层作用域中使用过。然而在类中，如果成员使用了外层作用域中的某个名字，而该名字代表一种类型，则类不能在之后重新定义该名字。



类型名的定义通常出现在类的开始处，这样就能确保所有使用该类型的成员都出现在类名的定义后。

## 构造函数

### 构造函数初始值列表

```c++
ConstRef::ConstRef(int i1):i(i1), ci(i1), ri(i) {}
```

如果成员是const、引用，或者属于某种未提供默认构造函数的类类型，我们必须通过构造函数初始值为这些成员提供初始值。

建议养成使用构造函数初始值的习惯。

**成员初始化顺序与他们在类定义中的出现顺序一致，初始值列表中初始值的前后位置关系不会影响实际的初始化顺序。**

最好令构造函数初始值的顺序与成员声明的顺序保持一致。如果可能的话，尽量避免使用某些成员初始化其他成员。

### 委托构造函数

一个委托构造函数使用所属类的其他构造函数执行它自己的初始化过程。在委托构造函数内，成员**初始值列表**只有一个唯一的入口，就是类名本身。

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

## 析构函数为什么是virtual的

1. 一般来说，如果一个类要被另外一个类继承，而且用其指针指向其子类对象时，如题目中的A* d = new  B();(假定A是基类，B是从A继承而来的派生类)，那么其(A类)析构函数必须是虚的，否则在delete  d时，B类的析构函数将不会被调用，因而会**产生内存泄漏和异常**； 

2. 在构造一个类的对象时，先构造其基类子对象，即调用其基类的构造函数，然后调用本类的构造函数；销毁对象时，先调用本类的析构函数，然后再调用其基类的构造函数； 

## 类的静态成员

与类本身直接相关，而不是与类的各个对象保持关联。

类的静态成员存在于任何对象之外，对象中不包含任何与静态数据成员有关的数据。类似的，静态成员函数也不与任何对象绑定在一起，它们不包含this指针。作为结果，**静态成员函数不能声明为const的，而且我们也不能在static函数体内使用this指针**。

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

对于关联容器(如map,  set,multimap,multiset)，删除当前的iterator，仅仅会使当前的iterator失效，只要在erase时，递增当前iterator即可。这是因为map之类的容器，使用了红黑树来实现，插入、删除一个结点不会对其他结点造成影响。erase迭代器只是被删元素的迭代器失效，但是返回值为void，所以要采用**erase(iter++)**的方式删除迭代器。 

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

## 概述

迭代器令算法不依赖容器。

泛型算法不会执行容器操作，只执行迭代器的操作，算法永远不会改变底层容器的大小。

算法不检查写操作。

## begin、end函数

获得指向arr首、尾后元素的指针

```c++
int arr[] = {1, 2, 3};
find(begin(arr), end(arr), 2);
```

## accumulate

```c++
int sum = accumulate(vec.cbegin(), vec.cend(), 0);
```

## equal

```c++
equal(r1.cbegin(), r1.cend(), r2.cbegin())
```

只接受一个迭代器来表示第二个容器的算法，都假定第二个序列至少与第一个序列一样长。

## fill

```c++
fill(vec.begin(), vec.end(), 0);
fill_n(vec.begin(), vec.size(), 0);
```

## copy

```c++
auto ret = copy(begin(a1), end(a1), begin(a2));
```

## replace

```c++
//将所有值为0的元素改为42
replace(l.begin(), l.end(), 0, 42);
```

## unique

```c++
//重排输入范围，使得每个元素只出现一次
//返回不重复区域之后一个位置的迭代器
auto end_unique = unique(words.begin, words.end());
```

## transform

```c++
transform(vi.begin(), vi.begin(), vi.begin(), [](int i) {
	return i < 0 ? -i : i;
});
```

## stable_sort

```c++
stable_sort(words.begin, words.end(), isShorter);
```

stable_sort维持相等元素额度原有顺序。

## lambda表达式

谓词：可调用对象（函数、函数指针、重载函数调用运算符的类、lambda表达式）

```c++
auto f1 = [](int i)->int {return i;};
//参数列表和返回类型可以忽略
auto f2 = [] {return i;};
```

捕获列表：lambda所在函数中定义的局部变量的列表。

捕获列表只用于局部非static变量，其他变量lambda可以直接使用。

如果未指定返回类型，自动推导只含单一return语句的lamdda，否则返回类型为void。

lamdda不能有默认参数。

当定义lambda时，编译器生成一个与lambda对应的新的类类型，该类型包含捕获列表中的数据成员。

```c++
//=:值捕获、&引用捕获（此时由捕获的数据成员编译器自动推导）
auto wc = find_if(words.begin(), words.end(), [=](const string &s) {
	return s.size() >= sz;
});
//混合使用时，第一个元素必须是=或者&，显示捕获和隐式捕获必须使用不同方式
//[&, args]
//[=, &args]
```

引用捕获的变量是否可以修改依赖于此引用指向的是否是const类型。

### 可变lambda

默认情况下，对于值被拷贝的变量，lamdda不会改变其值，如果希望改变，加上mutable。

```c++
auto f = [v1]() mutable {return ++v1;};
```

对于只在一两个地方使用的简单操作，lambda表达式是最有用的。如果在多处使用，通常应该定义一个函数。

lambda解决了向泛型算法传递一元谓词的问题。

## bind

可以将bind函数看作一个通用的函数适配器。

bind接受一个可调用的对象，生成一个新的可调用对象来“适应”原对象的参数列表。

bind也用于向泛型算法传递一元谓词的问题。

```c++
//占位符_n对应返回可调用对象的参数，定义在std::placeholders
//用bind重排参数顺序
//bind第一个参数为可调用对象，后面依次为该对象参数
auto g = bind(f, a, b, _2, c, _1);
g(_1, _2);
//相当于
f(a, b, _2, c, _1);
auto wc = find_if(words.begin(), words.end(), bind(check_size, _1, sz));
```

bind传递参数只能通过值传递，无法传递引用。

### ref、cref

ref、cref函数返回一个对象，包含给定引用，该对象可以拷贝。

## inserter

front_inserter和back_inserter调用push_back和push_front。

inserter类似于：

```c++
*it = val;
//相当于
it = c.insert(it, val);
++it;
//insert不同于上述
```

## iostream迭代器

```c++
istream_iterator<int> int_it(cin);
//用空迭代器充当eof，一旦关联的流遇到文件尾或IO错误
//和空迭代器相等
istream_iterator<int> int_eof;

//输出流迭代器可以指定额外字符串
ostream_iterator<int> out_int(os, "\n");
*out_int = val;
```

骚操作：

```c++
vector<int> vec(in_iter, eof);
copy(vec.begin(), vec.end(), out_int);
```

流迭代器要读取的类型必须重载<<或>>。

## 特定容器算法

对于list和forward_list，应该优先使用成员函数版本的算法（可以通过指针改变链接而不是交换它们的值）。

链表特有版本算法会改变底层容器，算法不会。

# 第十一章 关联容器

## 有序容器

multimap、multiset允许多个元素具有相同的关键字。

对于有序容器，关键字类型必须定义元素比较方法，默认使用<运算符，我们也可以用自己定义的操作来代替。

不能改变map和set的关键字（map::value_type为pair<const key_type, mapped type>，set::value_type为const key_type），可以调用erase删除，erase(key_type)返回实际删除的元素数量。

当使用一个迭代器遍历有序容器时，迭代器按关键字升序遍历元素。

创建pair最简单的方法是使用花括号初始化列表。

insert和emplace返回一个pair，包含指向新插入元素的迭代器和bool。

如果关键字不在map中，使用下标运算符会创建一个元素并插入到map中。

如果multimap和multiset中有多个元素具有给定的关键字，则这些元素在容器中会相邻存储。

如果元素不在multimap中，lower_bound和upper_bound会返回相等的迭代器（也可以使用equal_range)。

## 无序容器

对于无序容器，使用哈希函数和==运算符。

无序容器的输出通常会与有序容器的版本不同。

我们不能直接定义关键字类型为自定义类类型的无序容器，需要提供自己的hash模板版本。

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

在shared_ptr的生存期中，我们可以随时改变删除器类型，而unique_ptr需要在模板参数中指定。

shared_ptr可能的实现（不直接保存可调用对象）：

```c++
//del为一个指针
del ? del(p) : delete p;
```



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

## operator new和new operator

new operator是c++内建的，无法改变其行为；而operator new 是可以根据自己的内存分配策略去重载的。

当你写string *ps = new string(“Hands up!”)时，你所使用的new是所谓的new  operator，它其实干了两件事：一、分配足够的内存（实际大小是大于所创建的对象大小）二、调用对象构造函数，new  operator永远干这两件事。

编译器看到类类型的new或者delete表达式的时候，首先查看该类是否是有operator new或者operator delete成员，如果类定义了自己的new和delete函数，则使用这些函数为对象分配和释放内存，否则调用标准库版本。

当然你可以显示的调用:: operator new和:: operator delete强制使用全局的库函数。

# 第十三章 拷贝控制

## 三五法则

拷贝构造函数、拷贝赋值函数、析构函数、移动构造函数、移动赋值函数。

如果一个类有数据成员不能默认构造、拷贝、复制或销毁，则对应的成员函数将被定义删除的。

析构函数不能是删除的成员（无法delete）。

对于有引用成员的类，合成拷贝赋值运算符被定义为删除的。

## 拷贝构造函数

合成的拷贝构造函数会将其参数的成员逐个拷贝到正在创建的对象中。

对于类类型的成员，使用其拷贝构造函数来拷贝。

当**初始化**标准库容器或调用insert和push成员，容器会对其元素进行拷贝初始化。

使用emplace成员创建的元素都进行直接初始化。

拷贝构造函数的参数必须是引用类型（否则无限循环）。

在拷贝初始化过程中，编译器可以直接创建对象。

```c++
string null_book = "99999";
```

## 赋值运算符

如果将一个对象赋予它自身，赋值运算符必须能正确工作。

大多数赋值运算符组合了析构函数和拷贝构造函数的工作。

## swap

如果存在类型特定的swap版本，其匹配程度优先于std::swap。

### 拷贝并交换

自动处理了自赋值情况，且是异常安全的（发生异常，this不会被改变）。

单一的赋值运算符实现了拷贝赋值运算符和移动赋值运算两种功能（因为构造形参时，左值被拷贝，右值被移动）。

```c++
HasPtr &HasPtr::operator=(HasPtr rhs) {
    swap(*this, rhs);	//rhs是局部变量
    return *this;		//rhs被销毁，从而delete了rhs中的指针
}
```

## move

直接调用std::move而不是move，避免潜在的名字冲突。

调用move就意味着承诺，除了对移后源对象赋值或者销毁，不再使用它。

当我们调用move时，必须绝对确认移后源对象没有其他用户。

## 右值引用

右值引用指向将要被销毁的对象，可以自由地接管所引用的对象的资源。

不能将一个右值引用直接绑定到一个左值上。

## 移动构造函数

除了完成资源移动，移动构造函数还必须保证销毁移后源对象是无害的（nullptr、clear()）。

### noexcept

不抛出异常的移动构造函数和移动赋值运算符必须标记为noexcept。

标准库容器能对异常发生时其自身的行为提供保障。

除非移动构造函数不会抛出异常，否则在重新分配内存的过程中，它就必须使用拷贝构造函数而不是移动构造函数。

因为如果移动拷贝时发生异常，容器本身会出错。拷贝构造是异常安全的，发生异常直接释放新分配的空间即可。

## 合成的移动操作

只有当一个类没有定义任何自己版本的拷贝控制成员，且所有数据成员都能移动构造或移动赋值时，编译器才会为它合成移动构造函数或移动赋值运算符。

## 移动迭代器

移动迭代器解引用运算符生成一个右值引用。

```c++
auto last = uninitialized_copy(make_move_iterator(begin(), make_move_iterator(end()), first);
```

## 引用限定符&、&&

新标准仍然允许向右值赋值。

类似const限定符，引用限定符只能用于非static成员函数，且必须同时出现在函数的声明和定义中。

引用限定符也可以区分重载版本。

如果一个成员函数有引用限定符，则具有相同参数列表的所有版本都必须有引用限定符。

#  第十四章 重载运算与类型转换

## 运算符重载

当一个重载的运算符是成员函数时，this绑定到左侧运算对象。成员运算符函数的参数数量比运算对象的数量少一个。

输出运算符应该主要负责打印对象的内容而非控制格式，输出运算符不应该打印换行符。

输入运算符必须处理输入可能失败的情况。

输入运算符可能需要检查目标是否符合规范的格式，不符合时应该设置流的条件状态以标示出失败信息（通常为falibit）。

用int形参区分前置和后置运算符。

### 箭头运算符

重载的箭头运算符必须返回类的指针或自定义了箭头运算符的某个类的对象。

对于point->mem：

如果point是指针，应用内置的箭头运算符。如果是定义了operator->的类的对象，使用该运算符并重复该步骤（所以说会层层调用直到point为指针）。

## 标准库函数对象

```c++
sort(svec.begin(), svec.end(), greater<string>());
```

## function

```c++
//f是一个用来存储可调用对象的空function
//调用形式应该和T相同
function<T> f;
//例如
function<int(int, int)> f1 = add;
```

不能直接将重载函数的名字存入function类型的对象中。一条途径是使用存储函数指针。另一条是使用lambda。

```c++
int (*fp)(int, int) = add;
//binpos是一个map
binpos.insert({"+", fp});

binpos.insert({"+", [](int a, int b) {
    return add(a. b);
}});
```

## 类型转换运算符

转换构造函数和类型转换运算符共同定义了类类型转换。

类型转换运算符不能声明返回类型，形参列表也必须为空，通常为const。

```c++
operator int() const { return val; }
```

编译器一次只能执行一个用户定义的类型转换，但隐式的用户类型转换可以置于一个标准类型转换之前或之后。

### 显示的类型转换运算符

声明为explict，使用时通过强转。

该规则存在例外，如果表达式被用作**条件**，编译器会将显式的类型转换自动应用于它。

### 函数匹配与重载运算符

表达式中的运算符候选函数集包括成员函数和非成员函数中。

a sym b可能是：

a.operator sym(b);

operator sym(a, b);

# 第十五章 面向对象程序设计

编译器会隐式地执行派生类到基类的转换（智能指针也支持）（派生类部分将被忽略），不存在从基类向派生类的隐式类型转换。

在派生类对象中含有与其基类对应的组成部分。

派生类构造函数首先初始化基类的部分，按照声明的顺序依次初始化派生类成员。

派生类应该遵循基类的接口，调用基类的构造函数类初始化从基类继承的成员（即使可以直接赋值）。

友元关系不能被继承。

一条基类成员函数的using声明语句可以把所有重载实例添加到派生类作用域中。

## 虚函数

基类的虚函数在派生类中隐含地也是一个虚函数。

可以通过作用域运算符访问被覆盖的成员。

虚析构函数将阻止合成移动操作（即使通过=default）。

### 纯虚函数

```c++
virtual void f() = 0;
```

### 抽象基类

含有纯虚函数的类是抽象基类，不能创建抽象基类的对象。

## override、final

强制编译器检查某个函数是否重写基类虚函数，如果没有则报错。

override只是C++保留字，不是关键字，这意味着只有在正确的使用位置，oerride才启“关键字”的作用，其他地方可以作为标志符（如：int override；是合法的）。版本

final也是保留字，用于防止继承的发生。

```c++
void f() override { /* */ }
class NoDerived final { /* */ };
```

## 名字检查先于类型检查

对于p->mem()：

​	首先确定p的**静态类型**。

​	在静态类型中对应的类中查找mem。

​	一旦找到mem，就进行常规的类型检查。

​	合法则将根据是否是虚函数而产生不同的代码。

## 隐藏、覆盖、重写

重写 = 覆盖（基类virtual，且参数列表一致）（也能通过::来访问被覆盖的版本）

隐藏（否则）

覆盖是动态绑定，隐藏是静态绑定。

派生类成员将隐藏基类同名成员，即使形参列表不一样。

## 构造函数与拷贝控制

析构函数只销毁派生类自己分配的资源，基类部分自动销毁。

基类缺少移动操作会组织派生类拥有自己的合成移动操作。

当派生类定义了拷贝或移动操作，该操作负责拷贝或移动包括基类部分在内的整个对象。

```c++
D(D &&d) : Base(std::move(d)) { /* */ }
```

默认情况下，派生类构造函数调用基类构造函数。

拷贝和移动构造函数、赋值运算符都必须显式的为基类赋值。

对象销毁的顺序与创建的顺序相反。

如果构造函数或析构函数调用了某个虚函数版本，我们应该执行与所属类型对应的虚函数版本。

## 模拟虚函数

当容器保存基类指针时，如果只是new Base(b)，对于派生类对象可能不正确，解决办法是给每个类加上clone()虚函数。

## 悖论

我们无法直接使用对象进行面向对象编程，必须使用指针和引用。

# 第十六章 模板与泛型编程

## 非类型模板参数

非类型模板参数表示一个值而非一个类型，这些值必须是常量表达式，从而允许编译器在编译时实例化模板。

绑定到指针或引用非类型参数的实参必须具有静态的生存期。

```c++
template <unsigned N, unsigned M>
int compare(const char (&p1)[N], const char (&p2)[M]) {
	return strcmp(p1, p2);
}
//编译器会自动在末尾插入空字符
compare("hi", "mom");
//相当于
int compare(const char (&p1)[3], const char (&p2)[4]);
```

## 模板编译

只有当我们实例化出模板的一个特定版本时，编译器才会产生代码。

大多数编译错误在实例化期间报告。

函数为了生成一个实例化版本，编译器需要掌握模板的定义，所以模板的头文件通常既包含声明也包含定义。

（只要让编译器能够在某一次编译时看到模板函数的定义并将其实例化出来，最后把这些编译得到的目标文件链接在一起，就不会有模板函数链接失败的问题。例如可以：（1）将模板函数的定义直接写在头文件内；（2）可以写在源文件B内，并将这个源文件B包含在源文件A内；（3）在源文件A内想办法触发这个模板函数的实例化（例如可以显式实例化：template void  f<int>();）在编译A时将实例化结果写入生成的.obj文件内，让B的.obj文件与A的.obj文件链接时能够找到实例化结果。

​     上面的三种方法，前两种都是在编译期让所有用到模板函数的地方直接实例化函数定义，这样做会使每一个编译结果都包含实例化结果，导致目标文件较大，链接的时候需要把重复的实例化定义去重，编译链接的时间也会长一些。使用第三种方法，可以用一个特定的.cc文件显示实例化所有会被用到的模板实例，单独编译这个文件，最后让它参与链接，用这种方法，不会产生巨大的头文件，加快编译速度。而且头文件本身也显得更加“干净”和更具有可读性。但这个方法不能得到惰性实例化的好处，即它将显式地生成所有的成员函数，另外还要维护一个这样的文件。所以为了容易使用，几乎总是在头文件中放置全部的模板声明和定义。）

### call f

call f这行指令其实并不是这样的，它实际上是所谓的stub，也就是一个jmp  0x23423[这个地址可能是任意的，然而关键是这个地址上有一行指令来进行真正的call  f动作。也就是说，这个.obj文件里面所有对f的调用都jmp向同一个地址，在后者那儿才真正”call”f。这样做的好处就是连接器修改地址时只要对后者的call XXX地址作改动就行了。

### 显示实例化

我们可以通过显示实例化来避免这种开销。

由于编译器在使用一个模板时自动对其实例化，extern声明必须出现在任何使用此实例化版本的代码之前。

```c++
//实例化声明c++
extern template int compare(const int&, const int&);
extern template class Blob<string>;
//实例化定义
template int compare(const int&, const int&);
template class Blob<string>;
```

实例化定义会实例化该模板的所有成员，包括内联的成员函数（不包括普通的成员函数）。

## 类模板

与函数模板不同，编译器不能为类模板推断模板参数类型。

在类模板自己的作用域中，我们可以直接使用模板名而不提供实参。

### 类模板的成员函数c++

类模板的成员函数具有和模板相同的模板参数。

```c++
template <typename T>
ret-type Blob<T>::member-name(parm-list);
```

类模板的成员函数只有当程序用到它时才进行实例化。

### 类模板的友好关系

为了让所有实例成为友元，友元声明中必须使用与类模板本身不同的模板参数。

```c++
template <typename T>
class C2 {
    friend class Pal<T>;
    //Pal2所有实例都是C2所有实例的友元
    template <typename X> friend class Pal2;
    friend class Pal3;
}
```

## 模板类型别名

```c++
template <typename T> using twin = pair<T, T>;
twin<string> authors;
```

## typename

默认情况下，C++语言假定通过作用域运算符访问的名字不是类型。

使用typename来显示告诉编译器该名字是一个类型。

## 类型转换与模板类型参数

顶层const无论在形参中还是实参中，都会被忽略。

将实参传递给带模板类型的函数形参时，能够自动应用的类型转换只有const转换及数组或函数到指针的转换。

不涉及模板类型参数的类型，正常转换为对应形参的类型。

### 进行类型转换的标准库模板类

为了获得元素类型，我们可以使用标准库的类型转换。

```c++
remove_reference<int&>::type;	//int
```

## 显式模板实参

我们可以定义表示返回类型的模板参数，从而使用户控制返回类型。

```c++
//编译器无法推导T1
template <typename T1, typename T2, typename T3>
T1 sum(T2, T3);
//调用者必须为T1提供一个显式模板实参
//显式模板实参按由左至右的顺序与对应的模板参数匹配
auto val3 = sum<long long>(i, long);
```

## 尾置返回类型

尾置返回类型允许我们使用函数实参。

```c++
template <typename It>
auto fcn(It beg, It end) -> decltype(*beg) {
    return *beg;
}
```

## 引用折叠和右值引用参数

第一个例外规则：将左值传递给右值引用参数，且右值引用指向模板类型参数(T&&)，编译器推断模板类型参数为实参的左值类型。

第二个例外规则：间接创建一个引用的引用，发生引用折叠。

X& &、X& &&、X&& & -> X&

X&& && -> X&&

## move、forwarlist

move返回remove_reference<T>::type&&

forwarlist返回T&&

move可以用static_cast代替

## 转发

某些函数需要将其实参连同类型不变地转发给其他函数。

如果函数实参为T&&，对应实参的const属性和左值/右值将得到保持。

## 重载模板

函数模板可以被另一个模板或**普通非模板函数**重载。

优先选择非模板函数、最特例化的模板。

当有多个重载模板对一个调用提供同样好的匹配时，应选择最特例化的版本。

## 可变参数模板

为了终止递归，还需要定义一个非可变参数的函数。

定义时...写类型后面。

使用sizeof...获取包中元素数量。

### 包拓展

通过在模式右边放一个省略号来出发拓展操作。

扩展中的模式会独立应用于包中的每个元素。

```c++
//正确
print(os, debug_rep(rest)...);
//错误，debuf_rep没有那么多参数
print(os, debug_rep(rest...));
```

## 模板特例化

### 全特化

```c++
//函数模板全特化
template <>
int f(int a, int b) {
    return a + b;
}
//类模板偏特化
template <>
class hash<Sales_date> {
    /* */
};
```

### 偏特化

只能偏特化类模板，而不能偏特化函数模板。

可以只偏特化某个成员函数。