# 视C++为一个语言联邦

+ C、Object-Oriented C++、Template C++、STL。
+ 从某个次语言切换到另一个，可能导致高效编程守则改变。

# 尽量以const,enum,inline替换#define

+ 宁可以编译器替换预处理器。

+ 原因：
  + 宏定义的记号也许从未被编译器看到，编译出错时将因为追踪它而浪费时间。
  + 盲目替换可能导致目标码更大。
+ 常量定义式通常被放在头文件内（以便被不同的源码含入）。
+ string通常比其前辈char*-based更合时宜。
+ #define不能提供任何封装性。
+ 编译器不允许“static整数型class常量”完成“in class初值设定”，可改用“enum hack”补偿做法。
+ 如果不想别人获得一个pointer或reference指向某个整数常量，可以使用enum。
+ 用template inline函数代替宏函数（宏函数需要给每个变量加小括号，且变量带++运算符，可能++多次）。
+ 对于单纯常量，最好以const对象或enum替换#define。
+ 对于形似函数的宏，最好改用inline函数替换#define。

# 尽可能使用const

+ 如果希望STL模拟一个const T*指针，应该使用const_iterator而不是T* *const。
+ 令函数返回一个常量值，往往可以降低因客户错误而造成的意外（例如==写成=）。
+ 令non-const成员函数调用const成员函数是一个避免代码重复的安全做法（需要两次转型防止递归）。

# 确定对象被使用前已被初始化

+ 使用初始化列表效率更高（避免先调用default构造函数再调用拷贝赋值函数）。
+ 总是在初始化列表列出所有成员变量（不需要初始值的变量指定无物作为初始化实参即可，即直接加括号）。

+ class的成员变量总是以其声明次序被初始化，当在成员初值列中条列各个成员时，最好总是以其声明次序为次序。

+ C++对“定义于不同的编译单元内的non-local static对象“的初始化相对次序并无明确定义，使用Singleton模式解决这个问题。

# 若不想使用编译器自动生成的函数，就该明确拒绝

+ 为驳回编译器自动提供的技能，可将相应的成员函数声明为private并且不予实现。使用像Uncopyable这样的base class也是一种做法（无成员且拷贝赋值函数和拷贝构造函数声明为private）。

# 为多态基类声明virtual析构函数

+ 当derived class对象经由一个base class指针被删除，而该base class带着一个non-virtual析构函数，其结构未有定义——实际执行时通常发生的是对象的derived成员没被销毁。
+ 有时候希望拥有抽象class，但手上没有任何pure virtual函数。解法是声明一个pure virtual析构函数同时提供一份定义（避免derived class的析构函数报错）。
+ Class的设计目的如果不是作为base class使用，就不该声明virtual析构函数。

## 纯虚函数和虚函数的区别

定义一个函数为虚函数，不代表函数为不被实现的函数。

定义他为虚函数是为了允许用基类的指针来调用子类的这个函数。

定义一个函数为纯虚函数，才代表函数没有被实现。

定义纯虚函数是为了实现一个接口，起到一个规范的作用，规范继承这个类的程序员必须实现这个函数。

# 别让异常逃离析构函数

+ 析构函数绝对不要吐出异常（可能导致未定义的行为）。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下他们或结束程序。
+ 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么class应该提供一个普通函数执行操作（给客户一个处理错误的机会）。

# 绝不在构造和析构过程中调用virtual函数

+ 在构造和析构期间不要调用virtual函数，因为这类调用从不下降至derived class。

# 令operator=返回一个reference to *this

+ 为了实现连锁赋值，令赋值操作符返回一个reference to *this。

# 在operator=中处理“自我赋值”

+ copy-and-swap。

# 复制对象时勿忘其每一个成分

+ Copying函数应该确保复制“对象内的所有成员变量”及“所有base class成分”。
+ 不要尝试以某个copying函数实现另一个copying函数。应该将共同技能放进第三个函数中，并由两个copying函数共同调用。

# 以对象管理资源

+ 把资源放进对象内，我们便可倚赖C++的“析构函数自动调用机制”确保资源被释放。

+ 获得资源后立刻放进管理对象内，“资源取得时机便是初始化时机”。
+ 为了防止资源泄露，请使用RAII(Resource Acquisition Is Initialization)对象，它们在构造函数中获得资源并在析构函数中释放资源。

# 在资源管理类中小心copying行为

+ 复制RAII对象必须一并复制它所管理的资源，所以资源的copying行为决定RAII对象的copying行为（常见的是：抑制copying、施行引用计数法）。

# 在资源管理类中提供对原始资源的访问

+ 一般而言显式转换比较安全，但隐式转换对客户比较方便。

# 成对使用new和delete时要采用相同形式

+ 在new表达式中使用[]，必须在相应的delete表达式中也使用[]。

# 以独立语句将newed对象置入智能指针

+ 编译器对“跨越语句的各项操作”没有重新排列的自由（只有在语句内它才拥有那个自由度）。
+ 如果不这样做。一旦异常被抛出，可能导致难以察觉的资源泄露。

# 让接口容易被正确使用，不易被误用

+ 明智而审慎地导入新类型对预防“接口被误用”有神奇疗效。
+ 任何接口如果要求客户必须记得做某些事情，就是有着“不正确使用”的倾向。
+ 较佳接口的设计原则是先发制人。
+ “阻止误用”的办法包括建立新类型、限制新类型上的操作，束缚对象值，以及消除客户的资源管理责任。
+ shared_ptr支持定制型删除器。这可防范DDL问题，可被用来自动解除互斥锁。

# 设计class犹如设计type

+ 带着和“语言设计者当初设计语言内置类型时”一样的谨慎来研讨class的设计。

# 宁以pass-by-reference-to-const替换pass-by-value

+ 这种传递方式的效率高得多：没有任何构造函数和析构函数被调用，因为没有任何新对象被创建。
+ 解决切割问题（引用和指针都能触发多态）。
+ reference往往以指针实现出来，如果有个对象属于内置类型，pass by value往往比pass-by-reference-to-const更高效。

# 必须返回对象时，别妄想返回其reference

+ 绝不要返回pointer或reference指向一个local stack对象，或返回reference指向一个heap-allocated对象，或返回pointer或reference指向一个local static对象而有可能同时需要多个这样的对象。

# 将成员变量声明为private

+ 如果我们有一个public成员变量，而我们最终取消了它，所有使用它的客户代码都会被破坏。
+ 切记将成员变量声明为private。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，**并提供class作者以充分的实现弹性**（解耦合）。

# 宁以non-member、non-friend替换member函数

+ 越多东西被封装，就越少人可以看到它，我们就有越大的弹性去改变它。这就是我们推崇封装的原因：**它使我们能够改变事物而只影响有限客户**。
+ 将所有便利函数放在多个头文件内但隶属同一个命名空间，意味客户可以轻松扩展这一组便利函数。只需要在命名空间内建立一个头文件，内含那些函数的声明即可，而class定义式对客户而言是不能扩展的。
+ 这样做可以增加封装性、包裹弹性和机能扩充性。

# 若所有参数皆需类型转换，请为此采用non-member函数

+ 只有当参数被列于参数列内，这个参数才是隐式类型转换的合格参与者。

# 考虑写出一个不抛出异常的swap函数

+ 所有STL容器也都提供有public swap成员函数和std::swap特化版本（用以调用前者）。

+ C++只允许对class template偏特化，在function template身上偏特化是行不通的。
+ std是个特殊的命名空间，客户可以全特化std内的template，但不可以添加新的template到std里头。
+ 如果你希望你的软件有可预期的行为，请不要添加任何新东西到std里头。
+ 如果你想让你的“class专属版”swap在尽可能多的语境下被调用，需同时在该class所在命名空间内写一个non-member版本以及一个std::swap版本。
+ 如果swap缺省实现版的效率不足，试着做以下事情：
  + 提供一个public swap成员函数，让它高效地置换你的类型的两个对象值。
  + 在你的class或template所在的命名空间内提供一个non-member swap。并令它调用上述swap成员函数
  + 如果你正编写一个class（而非class temolate），为你的class特化std::swap。并令它调用你的swap成语函数。
+ 成员版swap绝不可抛出异常，因为swap的一个最好的应用是帮助class（和class template）提供强烈的异常安全性保障。
+ 调用swap时应针对std::swap使用using声明式，然后调用swap并且不带任何“命名空间资格修饰”。

# 尽可能延后变量定义式的出现时间

+ 这样做可增加程序的清晰度并改善程序效率（如果中途发生异常...）。

# 尽量少做转型动作

+ reinterpret_cast意图执行低级转型，实际动作可能取决于编译器，这也就表示它不可移植。static_cast用来强迫隐式转换。dynamic_cast主要用来执行“安全向下转型”。

+ 任何一个类型转换往往真的令编译器编译出运行期间执行的码。
+ Assignment to cast is illegal, lvalue casts are not supported。

# 避免返回handles指向对象内部成分

+ 遵守这个条款可增加封装性，帮助const成员函数的行为像个const，并将发生dangling handles的可能性降至最低。

# 为“异常安全”而努力是值得的

+ 异常安全函数即使发生异常也不会泄露资源或允许任何数据结构败坏。

# 透彻了解inlining的里里外外

+ inline是个申请，编译器可以忽略。
+ 编译器通常不对“通过函数指针而进行的调用”实施inlining（如果要取地址通常必须为此函数生成一个函数本体）。
+ inline函数无法随着程序库的升级而升级（一旦改变必须重新编译，如果是non-inline函数只需重新连接就好）。
+ 将大多数inlining限制在小型、被频繁调用的函数身上。

# 将文件间的编译依存关系降至最低

+ 连串编译依存关系会对许多项目造成难以形容的灾难。
+ 如果使用reference或pointer可以完成任务，就不要使用objcet。你可以只靠一个类型声明式就定义出指向该类型的reference和pointer；但如果定义某类型的object，就需要用到该类型的定义式。
+ 为声明式和定义式提供不同的头文件。

# 确定你的public继承塑模出is-a关系

+ 好的接口可以防止无效的代码通过编译。
+ 宁可采取“在编译期拒绝企鹅飞行”的设计，而不是“只在运行期才能侦测它们”的设计。
+ “public继承”意味is-a。适用于base class身上的每一件事情一定也使用于derived classed身上，因为每一个derived class对象也都是一个base class对象。

# 避免遮掩继承而来的名称

+ 为了让被遮掩的名称再见天日，可使用using声明式或转交函数。

# 区分接口继承和实现继承

+ 我们可以为pure virtual函数提供定义，但调用它的唯一途径是“调用时明确指出其class名称”。
+ pure virtual函数只具体指定接口继承，impure virtual函数具体指定接口继承及缺省实现继承，non-virtual函数具体指定接口继承以及强制性实现继承。

# 考虑virtual函数以外的其他选择

+ Non-Virtual Interface手法实现Template Method模式：

  保留原函数为public成员函数，但让它成为non-virtual，并调用一个private virtual函数进行实际工作。

  优点：wrapper确保得以在一个virtual函数被调用之前设定适当场景，并在调用结束之后清理场景。

+ Function Pointers实现Strategy模式：

  构造函数接受一个函数指针，同一类型的不同实体可以有不同的函数，实体的函数可在运行期变更。

+ tr1:function完成Strategy模式：

  该对象可持有任何callable entity（函数指针、函数对象或成员函数指针）。

# 绝不重新定义继承而来的non-virtual函数

+ 适用于基类对象的每一件事，也适用于派生类对象，因为每个派生类对象都是一个基类对象。如果派生类重新定义non-virtual函数，你的设计便出现矛盾。

# 绝不重新定义继承而来的缺省参数值

+ 静态绑定下不会继承缺省参数值，动态绑定会（缺省参数不是动态绑定的，这是C++做出的取舍）。
+ 解决方法是NVI手法：我们可以让public non-virtual函数指定缺省参数，而private virtual函数负责真正的工作。

# 通过复合塑模出has-a或“根据某物实现出”

+ 在应用域，复合意味has-a。在实现域，复合意味着is-implemented-in-terms-of。

# 明智而审慎地使用private继承

+ 由private base class继承而来的所有成员，在derived class中都会变成private属性。
+ 尽可能使用复合，必要时才使用private继承（当derived class需要访问protected base class的成员，或需要重新定义继承而来的virtual函数，这么设计是合理的）。
+ 如果是继承，派生类被编译时基类必须可见；如果只是内含一个指针，可以只带一个简单的声明式。
+ 由于技术上的理由，C++规定凡是规定对象都必须有非零大小（通常C++官方勒令默默安插一个char到空对象内）。因此和复合不同（还有可能有齐位需求），private继承可以造成empty base最优化。

# 明智而审慎地使用多重继承

+ 多重继承比单一继承复杂。它可能导致新的歧义性（菱形继承），以及对virtual继承的需要。
+ virtual继承会增加大小、速度、初始化复杂度等等成本。
+ 多重继承的确有正当用途。其中一个情节涉及“public继承某个Interface class”和“private继承某个协助实现的class”的两相组合。

# 了解隐式接口和编译期多态

+ “以不同的template参数具现化function template”会导致调用不同的函数，就便是所谓的编译期多态。

# 了解typename的双重意义

+ template内出现的名称如果相依于某个template参数，称之为从属名称。
+ 任何时候当你想要在template中指出一个**从属类型名称**，就必须在紧邻它的前一个位置放上关键字typename。
+ typename只被用来检验从属类型。这一规则的例外是，typename不可以出现在base classes list内，也不可在成员初值列作为base class修饰符。

# 学习处理模板化基类内的名称

+ 编译器知道base class template有可能被特化，而那个特化版本可能不提供和一般性template相同的接口。因此它往往拒绝在templatized base class内寻找继承而来的名称。
+ 有三个办法：
  + 第一是在base class函数调用动作之前加上“this->”。
  + 第二是使用using声明式（这里的情况并不是base class名称被derived class名称遮掩，而是编译器不进入base class作用域内查找，于是我们通过using告诉它，请它那么做）。
  + 第三个做法是，明确指出被调用的函数位于base class内（如果调用virtual函数会关闭“virtual绑定行为”。

# 将与参数无关的代码抽离templates

+ 因非类型模式参数而造成的代码膨胀，往往可消除，做法是以函数参数或class成员变量替换template参数。
+ 因类型参数而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表示的具现类型共享实现码（许多平台上long/int、指针等等）。

# 运用成员函数模板接受所有兼容类型

+ 真实指针做得很好的一件事是，支持隐式转换。但是，同一个template的不同具现体之间并不存在什么与生俱来的固有关系。
+ 成员模板并不改变语言规则，如果程序需要一个copy构造函数，你却没有声明它，编译器会暗自为你生成一个。所以想要控制copy构造的方方面面，必须同时声明泛化copy构造函数和“正常的”copy构造函数。

# 需要类型转换时请为模板定义非成员函数

+ 请将那些函数定义为“class template内部的friend函数”。

# 请使用traits class表现类型信息

+ Traits class使得“类型相关信息”在编译期可用。它们以templates和“templates特化”完成实现。
+ 整合重载技术后，traits class有可能在**编译期**对类型执行if...else测试。

# 认识template元编程

+ TMP是编写template-based C++程序并执行于**编译期**的过程。
+ TMP可将工作从运行期转移到编译期。这导致的一个结果是，某些错误原本通常在运行期才能侦测到，现在可在编译期找出来。
+ 程序如果使用TMP，其编译时间可能远长于不使用TMP的对应版本。
+ traits解法就是TMP。
+ C++模板参数不可以是double。
+ TMP可被用来生成“基于政策选择组合”的客户定制代码，也可用来避免生成对某些特殊类型并不适合的代码。

# 了解new-handler的行为

+ 当operator new抛出异常以反映一个未获满足的内存需求之前，它会先调用new-handler（使用set_new_handler指定）。
+ 一个设计良好的new-handler必须做以下事情：
  + 让更多内存可被使用。
  + 安装另一个new-handler。
  + 卸除new-handler（将null传给set_new_handler）。一旦没有安装任何new-handler，operator new会在内存分配不成功时抛出异常。
  + 抛出bad_alloc的异常。
  + 不返回，通常调用abort或exit。

+ 实现class专属之new-handler：令每一个class提供自己的set_new_handler和operator new即可。operator new确保在分配class对象内存的过程中以class专属之new-handler替换global new-handler。

# 了解new和delete的合理替换时机

+ 需要自定义new和delete的原因：

  + 用来检测运用上的错误：自定义operator new，超额分配内存，以额外空间放置特定的byte patterns。自定义operator delete得以检查上述签名是否原封不动，若否就表示在分配区的某个生命时间点发生了overrun或underrun。
  + 为了强化效能。
  + 为了收集使用上的数据。
  + 为了降低缺省内存管理器带来的空间额外开销（cookies）。
  + 为了弥补缺省分配器中的非最佳齐位：x86体系结构上double的访问最是快速，但编译器自带的operator new并不保证对动态分配而得的double采取8-byte齐位。

  # 编写new和delete时需固守常规

  + operator new应该内含一个无穷循环，并在其中尝试分配内存，如果它无法满足内存需求，就应该调用new-handler。它也应该有能力处理0 bytes申请。Class专属版本则还应该处理“比正确大小更大的（错误）申请”。
  + operator delete应该在收到null指针时不做任何事。Class专属版本则还应该处理“比正确大小更大的（错误）申请”。

  # 写了placement new也要写placement delete

  + 构造函数抛出异常时，运行期系统寻找“参数个数和类型都与operator new相同”的某个operator delete，如果找不到就什么也不做。
  + 如果没有这样做，你的程序可能会发生内存泄露。

  # 不要轻忽编译器的警告

  + 努力在你的编译器的最高警告级别下争取“无任何警告”的荣誉。
  + 不同的编译器对待事情的态度并不相同。

  # 让自己熟悉包括TR1在内的标准程序库

  + TR1添加了智能指针、一般化函数指针、hash-based容器、正则表达式以及另外10个组件的支持。

  # 让自己熟悉Boost

  + Boost在C++标准化过程中扮演深具影响力的角色。



