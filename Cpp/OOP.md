# Object-oriented Programming in C++
## OOP：概述
* **数据抽象**

* **继承**
    + 对于某些函数，基类希望它的派生类各自定义适合自身的版本，此时基类就将这些函数声明成**虚函数**
    ```
    class Quote {
    public:
        std::string isbn() const;
        virtual double net_price(std::size_t n) const;
    };
    ```
    + 派生类必须通过使用**类派生列表**明确指出它是从哪个基类继承而来的
    ```
    class Bulk_quote : public Quote {
    public：
        double net_price(std::size_t) const override;
    };
    ```
* **动态绑定**
    + 通过**动态绑定**，可以用同一段代码分别处理基类与派生类对象

    + 当使用基类的引用（或指针）调用一个虚函数时将发生动态绑定

---
## 定义
* **基类**
    + 基类通常都应该定义一个虚析构函数，即使该函数不执行任何实际操作

    + 将基类希望派生类进行覆盖的函数定义为虚函数

    + 任何构造函数之外的非静态函数都可以是虚函数

    + 非虚函数的成员函数解析过程发生在编译时而非运行时

* **派生类**
    + 派生类必须通过使用*类派生列表*指出继承自哪个基类
    
        `class Bulk_quote : public Quote {···};`
    + 访问说明符的作用是控制派生类从基类继承而来的成员是否对派生类的用户可见

    + 可以将共有派生类型的对象绑定到基类的引用或指针上

    + 可以显示注明派生类要覆盖自基类继承的虚函数
        
        `double net_price(std::size_t) const override;`

    + 派生类对象中含有其基类对应的组成部分，我们可以把派生类的对象当成基类对象来使用，**可以将基类的指针或引用绑定到派生类对象中的基类部分上**
    ```
    Quote item;         // 基类对象
    Bulk_quote bulk;    // 派生类对象
    Quote* p = &item;   // p指向Quote对象
    p = &bulk;          // p指向bulk的Quote部分
    Quote &r = bulk;    // r绑定到bulk的Quote部分
    ```
    + **编译器会隐式执行派生类到基类的类型转换，我们可以把派生类对象或者派生类对象的引用、指针用在需要基类引用或指针的地方**

    + 首先初始化基类部分，然后按照声明的顺序依次初始化派生类的成员
    ```
    Bulk_quote(const std::string& book, double p,
                std::size_t qty, double disc) :
                Quote(book, p), min_qty(qty), discount(disc) { }
    ```
    + **遵循类的接口，即使是派生类，也应该遵循基类的接口，通过调用基类的构造函数来初始化从基类继承而来的成员**
    
    + 如果基类定义了一个*静态成员*，则在整个继承体系中只存在该成员的唯一定义。不论从基类中派生出多少派生类，对于每个静态成员来说都只存在**唯一**的实例

    + 一条声明语句是让程序知晓某个名字的存在以及该名字表示一个怎么样的实体

        ` class Bulk_quote;     //正确的声明派生类方式`
    + **为了防止继承发生，在类名后跟关键字 final**

        `class NoDerived final {/* */};//无法作为基类`
---
## 类型转换
* 只要不含底层const, 均可以使用static_cast
    
    `double slope = static_cast<double>(j) / i;`
* 和内置指针一样，智能指针类也支持派生类向基类的类型转换，这意味着**可以将一个派生类对象的指针存储在一个基类的智能指针内**
* **表达式的静态类型在编译时总是已知的，它是变量声明时的类型或表达式生成的类型；动态类型是变量或表达式表示的内存中的对象中的类型**

* 如果表达式既不是引用也不是指针，则它的静态类型与动态类型永远一致；**基类的指针或引用的静态类型可能与其动态类型不一致**

* **不存在从基类向派生类的隐式类型转换**
    
    编译器不允许基类向派生类隐式转换，如果需要覆盖编译器工作，可以：

    * 如果在基类中含有一个或者多个虚函数，可以使用dynamic_cast请求类型转换，该转换的类型检查将在运行时执行

    * 如果已知某个基类向派生类的转换是安全的，则可以使用static_cast来强制覆盖编译器的检查工作 

* **对象之间不存在类型转换**
    
    * 当初始化或者赋值一个类类型的对象时，其实是调用某个函数

    * 当用一个派生类对象为一个基类对象初始化或者赋值时，只有该派生类对象中的基类部分会被拷贝、移动或赋值，它的派生类部分将被切掉

---
## 移动操作
* 标准库容器、string和shared_ptr类既支持移动也支持拷贝。IO类和unique_ptr类可以移动但不能拷贝

* 通过 **&&** 来获得绑定到**右值**的**右值引用**，右值引用只能绑定到一个将要销毁的对象

* 左值引用： 返回左值引用的函数，连同赋值、下标、解引用和前置递增/递减运算符

* 右值引用： 返回非引用类型的函数，连同算术、关系、位以及后置递增/递减运算符。可以用const的左值引用或者右值引用绑定

* **左值持久**，**右值短暂**

* 变量是左值，不能将一个右值引用直接绑定到一个变量上，即使这个变量是右值引用

* 可以通过**move**函数显式地将一个左值转换为对应的右值引用
    
        int &&r2 = std::move(r1);
    调用**move**意味着除了对源对象赋值或销毁外将不再使用它
---
## const成员
* const对象一旦创建后其值无法改变，所以const必须初始化

* 用一个对象去初始化另一个对象，无关const

* 默认情况下，**const对象仅在文件里有效**，如果想多个文件里使用，解决方法
    ```
    //file.cc
    extern const int buff = fcn();
    //file.h
    extern const int buff;
    ```
* 常量引用不能修改所绑定对象

* **想要存放常量对象的地址，只能使用指向常量的指针**

* 指向常量的指针与引用所指或引用的对象不一定是常量

* 底层const指针不允许通过指针改变值；顶层const指针不允许更改指针所指对象

* **auto**忽略顶层const,保留底层const，不过可以加cosnt提醒
    ```
    const int ci = 5;
    const auto f = ci;
    ```
* 任何对类成员的直接访问都被看做是this的隐式引用
    ```
    total.isbn();   // 当isbn使用bookNo, 类似this->bookNo

    Sales_data::isbn(&total)   //伪代码
    {
        this = &total;
    }
    ```

* const成员函数中的const作用是修改隐式this指针的类型，**默认情况下，this的类型是指向类类型非常量版本的常量指针 即ClassType * const**,所以无法将该指针绑定到常量对象上，使得**无法在一个常量对象上调用普通成员函数**，所以将this声明为**const ClassType * const**更具有灵活性，因而存在**const函数**

* **常量对象，以及常量对象的引用或者指针都只能调用常量成员函数，通常，不改变类内成员的函数应该声明为const函数，因为常量对象不会改变类内成员**
---
## 虚函数
* 通常如果不使用某个函数，则无需为其提供定义。但**虚函数必须有定义**

* 当且仅当对通过指针或引用调用虚函数时，才会在运行时解析该调用，也只有在这种情况下对象的动态类型才有可能与静态类型不同

* 一个函数被声明为虚函数，则在所有的派生类中它都是虚函数。当派生类覆盖了某个虚函数时，该函数在基类中的形参必须与派生类中的形参严格匹配

* 用**override**关键字来说明派生类中的虚函数，只有虚函数才能被覆盖
    ```
    struct B {
        virtual void f1(int) const;
        virtual void f2();
        void f3();
    };
    struct D : B {
        void f1(int) const override;    // 正确
        void f2(int) overdide;          // 错误，形式不匹配
        void f3() override;             // 错误，f3不是虚函数
        void f4() override;             // 错误，基类中无f4()
    };
    ```
* 用**final**关键字来将某个函数指定为final，之后任何尝试覆盖该函数的操作都将引发错误
    ```
    struct D2 ：B {
        void f1(int) const final;
    };
    struct D3 : D2 {
        void f2();              // 正确
        void f1(int) const;     // 错误, D2已经将其声明为fianl了
    };
    ```
* **如果虚函数使用了默认实参，则基类和派生类中定义的默认实参最好一致**，否则实际执行基类函数

* 通常情况下，成员函数（或者友元函数）中的代码需要使用基类中的虚函数，这时需要用作用域运算符来回避虚函数的机制，否则将引发无限递归调用
    ```
    double net_price(int num)
    {
        double undiscounted = Quote::net_price(num);
        return undiscounted*0.7;
    }
    double dis = baseP->Quote::net_price(42);
    ```
---
## 抽象基类
* 纯虚函数
    + 在声明语句的分号之前书写 **= 0** 就可以将一个**虚函数**说明为**纯虚函数，= 0 只能出现在类内部的虚函数声明语句处**
    ```
    class Disc_quote : public Quote {
    public:
        Disc_quote() = default;
        Disc_quote(const std::string& book, double price,
                    std::size_t qty, double disc):
                        Quote(book, price),
                        quantity(qty), discount(disc) { }
        double net_price(std::size_t) const = 0;    // net_price()函数在基类中为virtual
    protected:
        std::size_t quantity = 0;
        double discount = 0.0;
    };
    ```
    + **可以为纯虚函数提供定义，不过函数体必须定义在类的外部**

* **含有（或者未经覆盖直接继承）纯虚函数的类是抽象基类。抽象基类负责定义接口，后续的类可以覆盖接口。不能（直接）创建一个抽象基类的对象。派生类除非覆盖了纯虚函数，否则仍将是抽象基类**

* 派生类构造函数只初始化它的直接基类
    ```
    class Bulk_quote : public Disc_quote {
    public:
        Bulk_quote() = default;
        Bulk_quote(const std::string& book, double price,
                    std::size_t qty, double disc):
                    Disc_quote(book, price, qty, disc) { }
        double net_price(std::size_t) const override;
    };
    ```
    + 在继承体系中增加Disc_quote类是**重构**的典例。

    + **重构负责重新设计类的体系以便将操作和/或数据从一个类移动到另一个类中**

    + **即使重构了类的体系，使用类的代码也无须进行任何改动，只需要重新编译**
---
## 访问控制与继承
* protected成员

    + protected成员对于类的用户来说不可访问

    + protected成员对于派生类的成员和友元来说是可访问的
 
    + **派生类的成员和友元只能访问派生类对象中的基类部分的受保护成员；对于普通的基类对象中的成员不具有特殊的访问权限**
    ```
    class Base {
    protected:
        int prot_mem;
    };
    class Sneaky : public Base {
        friend void clobber(Sneaky&);   // 可访问Sneaky::prot_mem
        friend void clobber(Base&);     // 不可访问Base::prot_mem
        int j;
    };
    // 正确
    void clobber(Sneaky& s) { s.j = s.prot_mem = 0; }
    // 错误 无法访问基类的protected成员
    void clobber(Base& b) { b.prot_mem = 0; }
    ```
* public、protected和private继承

    + **派生访问说明符对于派生类的成员（及友元）能否访问其直接基类的成员没什么影响。对基类成员的访问权限只与基类中的访问说明符有关**
    ```
    class Base {
    public:
        void pub_mem();
    protected:
        int prot_mem;
    private:
        char priv_mem;
    };
    struct Pub_Derv : public Base {
        // 正确
        int f() { return prot_mem; }
        // 错误
        char g() { return priv_mem; }
    };
    struct Prot_Derv : protected Base {
        // 错误
        char g1() { return priv_mem; }
    }
    struct Priv_Derv : private Base {
        // 正确
        int f1() const { return prot_mem; }
    };
    ```
    + **派生访问说明符的目的是控制派生类用户（包括派生类的派生类在内）对于基类成员的访问权限**
    ```
    Pub_Derv d1;
    Prot_Derv d2;
    Priv_Derv d3;
    d1.pub_mem();   // 正确：pub_mem()在派生类中是public的
    d2.pub_mem();   // 错误：pub_mem()在派生类中是protected的
    d3.pub_mem();   // 错误：pub_mem()在派生类中是private的
    struct Derived_from_Public : public Pub_Derv {
        // 正确：Base::prot_mem在Pub_Derv中仍是protected
        int use_base() { return prot_mem; }
    };
    struct Derived_from_Protected : public Prot_Derv {
        // 正确：Base::prot_mem在Pub_Derv中仍是protected
        int use_base() { return prot_mem; }
    };
    struct Derived_from_Private : public Priv_Derv {
        // 错误：Base::prot_mem在Pub_Derv中是private
        int use_vase() { return prot_mem; }
    }
    ```
* 派生类向基类转换的可访问性

    + **对于代码中的某个给定结点来说，如果基类的公有成员是可访问的，则派生类向基类的类型转换也是可访问的；反之则不行**

* 友元与继承

    + **不能继承友元关系；每个类负责控制各自成员的访问权限**

    + 友元能够访问源类对象的成员，这种可访问性包括了源类对象内嵌在其派生类对象中的情况
    ```
    class Base {
        friend class Pal;
    };
    class Pal {
    public:
        int f(Base b) { return b.prot_mem; }        // 正确：Pal是Base的友元
        int f2(Sneaky s) { return s.j; }            // 错误：Pal不是Sneaky的友元
        // 对基类访问的权限由基类本身控制，即使对于派生类的基类部分也是如此
        int f3(Sneaky s) { return s.prot_mem; }     // 正确：Pal是Base的友元
    }
    ```
* 改变个别成员的可访问性

    + 通过**using声明**可以改变派生类继承的某个名字的访问级别
    ```
    class Base {
    public:
        std::size_t size() const { return n; }
    protected:
        std::size_t n;
    };
    class Derived : private Base {
    public:
        using Base::size;
    protected:
        using Base::n;
    };
    ```
    + **派生类只能为那些它可以访问的名字提供using声明**

* 默认的继承保护级别

    + **默认情况下，使用class关键字定义的派生类是私有继承；使用struct关键字定义的派生类是公有继承的**

    + **struct 与 class 唯一的差别就是默认成员访问说明符以及默认派生访问说明符**

    + **私有继承的类最好显式声明**
---
## 继承中的类作用域
* 派生类的作用域位于基类作用域之内
    ```
    {
        A               // 基类
        {
            B           // 派生类
        }
    }                   // B中可使用A中可访问成员
    ```
* 派生类的成员将隐藏同名的基类成员，通过作用域运算符来使用被隐藏的基类成员

* **除了覆盖继承而来的虚函数，派生类最好不重用其它定义在基类中的名字**

* 成员函数无论是否为虚函数都可以被重载；基类可能重载了许多函数，派生类可以覆盖重载函数；如果派生类希望所有的重载版本都是可见的，可以为重载的成员提供一条**using**声明，using声明指定一个名字，所以一条基类成员函数的using声明就可以把该函数的所有重载实例添加到派生类作用域中
---
## 构造函数与拷贝控制
* 虚析构函数
    + 析构函数的虚属性也会被继承

    + **如果基类的析构函数不是虚函数，则 delete 一个指向派生类对象的基类指针将产生未定义的行为**

    + 基类不必遵从三五法则

    + 虚析构函数将阻止合成移动操作

* 合成拷贝控制与继承
    + 如果基类中的默认构造函数、拷贝构造函数、拷贝赋值运算符或析构函数是被删除的函数或者不可访问，则派生类中对应的成员将是被删除的

    + 定义虚析构函数的基类默认不含有合成的移动操作，除非显式定义；一旦定义了移动操作，必须显式定义拷贝构造操作，否则会被定义为删除函数
    ```
    class Quote {
    public:
        Quote() = default;
        Quote(const Qoute&) = default;
        Quote(Quote&&) = default;
        Quote& operator(const Quote&) = default;
        Quote& operator(Quote&) = = default;
        virtual ~Quote() = = default;
    };
    ```
* 派生类的拷贝构造成员
    + **当派生类定义了拷贝或移动操作时，该操作负责拷贝或移动包括基类部分成员在内的整个成员，析构函数则只负责销毁派生类自己分配的资源**

    + **默认情况下，基类默认构造函数初始化派生类对象的基类部分。如果我们想拷贝（或移动）基类部分，则必须在派生类的构造函数初始值列表中显式地使用基类的拷贝（或移动）构造函数**
    ```
    class Base { /*...*/};
    class D : public Base {
    public:
        D(const D& d) : Base(d)
        { /*...*/ }
        D(D&& d) : Base(std::move(d))
        { /*...*/ }
    };
    ```
    + 赋值运算符显式为基类部分赋值,**基类的运算符应该可以正确处理自赋值情况**
    ```
    D& D::operator=(const D& rhs)
    {
        Base::operator=(rhs);
        //为派生类成员赋值
        //酌情处理自赋值情况
        return *this;
    }
    ```
    + **派生类析构函数只负责销毁派生类自己分配的资源；对象销毁顺序正好与创建顺序相反：派生类析构函数首先执行，然后是基类的析构函数**

    + 如果构造函数或析构函数调用了某个虚函数，则我们应该执行与构造含糊或析构函数所属类型相对应的虚函数版本

* 继承的构造函数
    + 通常情况下，using声明语句只是令某个名字在当前作用域可见。而当作用于构造函数时，using声明语句将令编译器产生代码。对于基类的每个构造函数，编译器都生成一个与之对应的派生类构造函数，如果派生类含有自己的数据成员，这些成员将被默认初始化
    ```
    derived(parms) : base(args) { }
    ```
    + 与普通成员using声明不一样，构造函数的using声明不会改变该构造函数的访问级别，而且，一个using声明语句不能指定explicit或constexpr
---
## 容器与继承
* 当派生类对象被赋值给基类对象时，其中的派生类部分将被“切掉”，因此容器和存在继承关系的类型无法兼容

* **在容器中放置智能指针而非对象**
    ```
    vector<shared_ptr<Quote>> basket;

    basket.push_back(make_shared<Quote>("0-201-82470",50));
    basket.push_back(
        make_shared<Bulk_quote>("0-201-82470",50,10,0.25) );
    cout << basket.back()->net_price(15) << endl;
    ```
    
