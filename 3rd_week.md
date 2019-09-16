# 3rd_week

[TOC]

---
## this指针
1. 从某种程度来说，`C++`是由C实现的。前面`2nd_week`也有提到`C++`的类是由C的`struct`实现的。那么现在可以看一下下面的例子，体会一下C++的成员函数是怎么实现的。   
    ```C++
    class A
    {
    public:
        int i;
        void setI(int x);
    };
    void A::setI(int x)
    {
        i = x;
    }
    int main()
    {
        A a;
        a.setI(100);
        return 0;
    }
    ```

    C的实现如下：  
    > 需要注意的是，在C中，结构体不能拥有成员函数；但在C++中，结构体与类基本没有了差别，也可以在结构体中写入成员函数。
    ```C
    //C++的类由C中的结构体实现
    struct A
    {
        int i;
    };
    //C将C++中类的成员函数作为一个全局函数，但是参数中多增加了一项结构体对象的指针。
    void setI(struct A* this, int x)
    {
        this->i = x;
    }
    int main()
    {
        struct A a;
        setI(&a, 100);
        return 0;
    }
    ```
    由上面的例子可以知道，类的成员函数在编译时实际上**还有一个参数**，实际参数个数+1。
2. 在非静态成员函数中可以使用`this`指针，来代表指向该成员函数作用的对象的指针。
    ```C++
    class A
    {
    public:
        int i;
        void Print(){ cout<<i<<endl; }
        A(int x):i(x){}
        A func()
        {
            i++;
            Print();
            return *this;
        }
    };
    int main()
    {
        A a(10),b(0);
        b = a.func();//利用this指针来返回当前**函数作用的对象a**作为返回值。
        return 0;
    }
    ```
3. 下面这个例子有一点意思。
    ```C++
    class A 
    {
    private:
        int i;
    public:
        void Hello1(){ cout << "hello" << endl; }
        void Hello2(){ cout << this->i << "hello" <<endl; }
    };
    int main()
    {
        A* p = NULL;//空指针
        p->Hello1();//虽然是空指针，但是此处完全OK。
        p->Hello2();//Error！
    }
    ```
    其中`void Hello()`其实等价于`void Hello(A* this)`。而在执行`p->Hello()`时，等价于`Hello(p)`。
4. **静态成员函数**中不能使用this指针-->因为静态成员函数并不具体作用于某个对象。
---
## static
1. 静态成员变量 & 静态成员函数
    1. 本质上是全局变量 & 全局函数。哪怕一个对象都不存在，类的静态成员变量也存在。
    2. 一个类的**静态成员变量**由此类所有的对象共享。静态成员函数也不需要具体作用于某个对象。-->因此，静态成员**不需要通过对象**就能访问。
        > 必须的前提：此成员对象属于`可访问的范围`，比如`public`之于`main`。
    3. **静态成员变量**需要在定义类的文件中对成员变量进行一次说明或初始化。否则编译能通过，但是链接不能通过，在VS中的报错如下图，可见链接确实有问题。
       ![链接出现错误](./pic/20190909171545089_16405.png)
    4. **静态成员函数**中，**不能直接访问**非静态成员变量，也**不能直接调用**非静态成员函数。因为静态成员函数可以不通过对象进行访问，如果其中包含非静态的成员，就会出现混乱，不知道此非静态的成员属于哪个对象。
        > 当然，可以利用参数获得对象的引用，然后再去访问相应的非静态成员。这里要强调的是不能**直接**去访问。
    5. 下面用一个例子展示有关`static`的内容。
        ```C++
        class B {
        private:
        	int i;
        	static int counter;
        public:
        	static int num;
        	B(int x) :i(x) { counter++; }
        	static void showCounter()
        	{
                    cout << counter << endl;
        	}
        	void printStaticNum()
        	{
                    cout << num << endl;
        	}
        	static void Print();
        	~B()
        	{
                    if (counter!=0){ counter--; }
        	}
        };
        int B::counter = 0;
        int B::num = 55;
        void B::Print()
        {
            cout << "czdpzc" << endl;
        }
        int main()
        {
            B b1(100);
            B::showCounter();//直接访问静态成员函数
            B b2(10);
            b2.showCounter();//通过对象访问静态成员函数
            b1.printStaticNum();
            cout << B::num << endl;//直接访问静态成员变量
            b1.~B();
            b2.showCounter();
        }
        //输出：1  2  55  55  1
        ```
        - 类B拥有两个静态成员变量，其中一个是`private`，一个是`public`。它们均在`22,23`行进行初始化（也可以只声明）。类B拥有两个静态成员函数，`showCounter()`在类中直接定义，`Print()`在类外进行定义。
            > `static`在类中声明变量或函数后，类外的的初始化或定义就不能再加`static`了。
        - 访问静态成员的方法有多种：
            - 类名::成员名
            - 对象名.成员名
            - 指针->成员名
            - 引用.成员名
2. 上面这个例子还引出一个问题。可以看到，静态成员变量`counter`可以用来**实时**统计有多少个`class B`的对象。因为我在构造函数中进行`counter++`，并且在析构函数中进行`counter--`。  
    但是此处有设计缺陷：没有考虑到有一些**临时对象**是通过`拷贝构造函数`生成（详见`2nd_week`），不仅它们的生成没有计算到，而且临时对象的消亡还会调用析构函数，导致`counter`计数的不准确。  
    所以此处还需要自己重写拷贝构造函数：在完成拷贝工作的同时，进行`counter++`计数。
---
## 成员对象和封闭类
1. 定义：有**成员对象**的类叫封闭类(`enclosing`)。-->类里面有其它类的对象。
2. 任何生成封闭类对象的语句，都要让编译器明白，此对象中的成员对象应该如何初始化。
    所以如果是一个封闭类，即有成员对象，那么必须使用**初始化列表**`(详见2nd_week)`来初始化成员对象，而不是在封闭类的构造函数中来初始化。除非此成员对象本身仍然使用默认构造函数，那么它可以不初始化。
3. 例子
    ```C++
    class Tire//轮胎类
    {
    private:
        int radius;
        int width;
    public:
        Tire(int r, int w):radius(r), width(w){}
    };
    class Engine{};//引擎类（此处是个空类，且只有默认构造函数）
    class Car//车类：由轮胎和引擎组成。
    {
    private:
        int price;
        Tire tire;//成员对象
        Engine engine;//成员对象
    public:
        //利用初始化列表进行成员对象的初始化。
        //因为Engine是默认构造函数，所以不用在此初始化。
        Car(int p, int tr, int tw):price(p),tire(tr, tw){}
    };
    int main()
    {
        Car car(20000,17,225);
        return 0;
    }
    ```
4. 封闭类构造函数和析构函数的**执行顺序**（C++标准确定的）
    1. **成员对象的构造函数**先于**封闭类本身的构造函数**执行。原因有几个：
        - 初始化列表的执行在构造函数之前，所以在其中初始化的成员对象首先执行其构造函数。
        - 逻辑上讲，封闭类有可能在构造函数中使用到成员对象，故成员对象先执行。
    2. 成员对象的构造函数调用次序，与其**被声明的次序**一致，和其在初始化列表里的次序无关。
    3. 对于析构函数，则是先执行封闭类的析构函数，再执行成员对象的析构函数，与构造函数调用次序相反。因为封闭类在析构函数中还有可能使用到成员对象。
5. 谈一下封闭类的拷贝构造函数。  
    其实道理也是一样的，当封闭类对象是通过`拷贝初始化`定义时，会相应地执行**封闭类的拷贝构造函数**和**成员对象的拷贝构造函数**，就不会执行构造函数了。
---
## 常量对象 & 常量成员函数
1. 使用`const`关键字来实现。
    ```C++
    //常量对象
    const A a;
    //常量成员函数
    class A 
    {
        ...
        void getX() const {...}
        ...
    };
    ```
2. 若对象是一个**常量对象**，则此对象中的值都**不能被修改**，也不能执行**非常量成员函数**。
3. 若成员函数是一个**常量成员函数**，则函数中不能修改成员变量的值、不能调用非常量成员函数。
    > 除静态成员变量与静态成员函数外，它们“超越了对象”。:-)
4. 只要有常量对象，编译器只允许其执行常量成员函数。即使非常量成员函数没有做出修改值的举动，也不允许执行。
5. 例子
    ```C++
    class A
    {
    public:
        int x;
        void getX() const { cout << x << endl; } //OK.
        void setX(int a) const { x = a; }        //ERROR!
        void doubleX() { x++; }
        void nullFunc() { }
    };
    int main()
    {
        const A a;     //常量函数
        a.doubleX();   //ERROR!
        a.nullFunc();  //ERROR!
        a.getX();      //OK.
    }
    ```
6. 两个成员函数，名字和参数表都一样，但是其中一个是`const`，那么它们属于**重载**`overload`关系。
---
## friend  友元
1. 友元函数：在类`A`中声明全局函数或另一个类的成员函数为`friend`，那么这些函数就可以访问类`A`的`private`成员。
    > 成员函数里也包括构造函数和析构函数。
2. 友元类：在类`A`中声明类`B`是`friend`的，那么存在于在类`B`中的类`A`对象就可以直接访问类`A`的`private`成员。
    ```C++
    class A
    {
        friend class B;//放置顺序无所谓。
    private:
        int i;
    ...
    };
    class B
    {
        A a;//其初始化需在初始化列表中完成，详见**封闭类**知识点。
        ...
        void Print() { cout << a.i << endl; }//可以访问对象a的私有成员i。
    };
    ```
3. 友元关系不能传递，不能继承。  
    例：A是B的友元，B是C的友元，不能说A是C的友元。









