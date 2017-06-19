# Effective C++
+ **导读**

    * 定义式是编译器为此对象拨发内存的地点
    * 除非有更好的理由允许构造函数被用于隐式转换，否则将其声明为 explict

+ **将C++分成四部分：**

    * C 
    * 面型对象
    * 泛型
    * STL

+ **对于单纯常量，尽量以const , enum 替换 #define; 对于形似函数的宏, 用inline函数**

    * 用常量替换宏
        ```
        #define ASPECT_RATIO 1.653
        // 可能未计入记号表，替换为
        const double AspectRatio = 1.653;
        ```
    * 注意class的专属常量，**为了限制常量作用域并确保至多有一个实体，将其声明为static成员**\
        `static const int Num = 5;`\
    **还必须在实现文件中定义**\
        `const int Game::Num;`
        `//因为其在声明中已经获得初值，所以定义不用`
    * 一个属于枚举类型的数值可以权充Int被使用(the enum hack)
        ```
        class GamePlayer{
        private:
            enum{ Num = 5 };
            int scores[Num];
            ···
        };
        ```
+ **尽可能使用const**

    * 除非你需要改动参数或local对象, 否则将它们声明为const
    * const成员函数：
        + 它们使class借口比较容易理解：得知哪个函数可以改动对象内容哪个函数不行
        + 它们使得"操作const对象"成为可能
    * 编译器强制实施bitwise constness, 但我们应该使用"概念上的常量性"\
        *bitwise: 成员函数只有在不更改对象的任何成员变量时才可以是const*
        ```
        class CTextBlock {
        public:
            ...
            std::size_t length() const;
        private:
            char* pText;
            std::size_t textLength;    //最近一次计算的文本区块长度
            bool lengthIsValid;        //目前的长度是否有效
            //mutable std::size_t textLength;
            //mutable bool lengthIsValid;
        };
        std::size_t CTextBlock::length() const
        {
            if (!lengthIsValid)
            {
                textLength = std::strlen(pText);    //错误, const成员函数内无法赋值成员
                lengthIsValid = true;              
            }
            return textLength;
        }
        ```
        + **解决方案是利用一个与const相关的摆动场：mutable**
    * 在const和non-const成员函数中避免重复\
      为了避免代码的重复,利用**转型**实现：
        ```
        class TextBlock{
        public:
        ···
        const char& operator[](std::size_t position) const
        {
            ···
            return text[position];
        }
        char& operator[](std::size_t position)
        {
            return
                const_cast<char&>(                          //将op[]返回值的const移除
                    static_cast<const TextBlock&>(*this)    //为*this加上const
                        [position]                          //调用const op[]
                );
        }
        ···
        };
        ```
        + **当const和non-const成员函数有着实质等价的实现时, 令non-const版本调用const版本可避免代码重复**

+ **确定对象被使用前已先被初始化**
    * 确保每一个构造函数都将对象的每一个成员初始化
    * 构造函数使用列表初始化, 避免赋值操作
    * 为免除"跨编译单元之初始化次序"问题, 以local static对象替换non-local static对象
        ```
        问题:

        //FileSystem.h
        class FileSystem {
        public:
            ···
            std::size_t numDisks() const;
        };
        extern FileSystem tfs;


        //Diretory.h
        #include"FileSystem.h"
        class Directory {
        public:
            Directory( params );
            ···
        };
        Directory::Directory( params )
        {
            ···
            std::size_t disks = tfs.numDisks();
            ···
        }


        //main.cpp
        Directory tempDir( params );    //除非tfs在tempDir之前先被初始化, 否则无法执行


        解决方案：(Singleton模式)

        //FileSystem.h
        class FileSystem { ... };
        FileSystem& tfs()
        {
            static FileSystem fs;                   //定义并初始化一个local static对象
            return fs;                              //返回一个reference指向上述对象
        }


        //Directory.h
        #include"FileSystem.h"
        class Directory{ ... };
        Directory::Directory( params )
        {
            ···
            std::size_t disks = tfs().numDisks();
            ···
        }
        Directory& tempDir()
        {
            static Directory td;                    //定义并初始化一个local static对象
            return td;                              //返回一个reference指向上述对象
        }

        
        //main.cpp
        tfs();
        tempDir();
        ```
        + 使用tfs(), tempDir()代替tfs, tempDir
        + 保证所获得的reference指向一个历经初始化的对象
        + 如果从未调用这个"仿真函数", 就**绝不会引发构造和析构成本**
+ **如果不想使用编译器生成的函数，将其设置为delete**
    * 设计类时可能不希望该类进行拷贝操作
        ```
        class HomeForSale {
            HomeForSale();
            ~HomeForSale();
            HomeForSale(const HomeForSale) = delete;
            HomeForSale& operator=(const HomeForSale&) = delete;
        };
        ```
+ **为多态基类声明virtual析构函数**
    * 不要无端的使用virtual析构函数，因为这会增加类的体积，从而造成不必要的麻烦
    * 只有当class内至少含一个virtual函数，才会为它声明virtual析构函数
        ```
        class SpecialString : public std::string {
            ···         //馊主意,因为std::string无虚析构函数
        };
        SpecialString* pss = new SpecialString("Impending Doom");
        std::string* ps;
        ···
        ps = pss;       //用基类指针指向派生类, SpecialString*->std::string*
        ···
        delete ps;      //但是删除时只会出发基类析构函数,造成内存泄漏
        ```
    * 如果不希望继承应当声明为final