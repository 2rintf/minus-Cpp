# 1st_week

[TOC]

---
## 引用 Reference（就是变量的别称/别名，不是对象，必须被初始化)  

```C++
int b = 10;
int& a = b;
```  

1. 定义引用必须**初始化**。
2. 引用只能引用**变量**，不能引用常量和表达式。
3. 参数传递的编写更加方便，不用使用太多指针。
    ```C++
    //1.使用指针
    void swap(int* a, int* b)
    {
        int tmp;
        tmp = *a;
        *a = *b;
        *b = tmp;
    }
    int n1,n2;
    swap(&n1,&n2);
    //2.使用引用
    void swap(int& a, int& b)
    {
        int tmp;
        tmp = a;
        a = b;
        b = a;
    }
    int n1,n2;
    swap(n1,n2);
    ```
4. 引用作为函数返回值（**此前不常使用的特性**）
    ```C++
    int n = 4;
    int& SetValue(){ return n; }
    int main()
    {
        SetValue() = 40;
        cout<<n;
        return 0;
    }
    //输出：40
    ```
    看一下下面这个例子
    ```C++
    int x = 0;
    int& g(){ return x; }
    int main()
    {
        int& fri = g();
        /**此时有两种关系：
        *  1. g()是x的引用
        *  2. fri是g()的引用
        */
        g() = 16;//这里没有错误，意味着g()返回的是一个左值
    }
    //结果x=16。
    ```
    
5. 常引用   `const int& r = n;`
    1. 不能用常引用去修改其引用的内容。
    2. 常引用可以被引用或变量初始化，但是常引用`const int&`或常变量`const int`不可以用来初始化引用。除非进行强制类型转换。
    3. 将对象的常引用作为成员函数的参数，可以**避免调用对象的拷贝构造函数（详见`2nd_week`）**，从而减少开销。
        当然，如果想在函数中修改对象，可以只使用引用，但仍可以起到减少开销的作用。
---
## const
1. 定义形式
    1. 定义**常量**    `const double pai = 3.14;`
    2. 定义**常量指针**（**不可通过常量指针修改其指向的内容**）
        > 关键问题：指针是const？还是指针所指的内容为const？  
        > 1. `const char* q = "abc";`  
        > 2. `char* const q = "abc";`  
        > 3. `Person p1; `  
        >     `const Person* p = &p1;`对象是const  
        >     `Person* const p = &p1;`指针是const  
        >     `Person const* p = &p1;`对象是const  
        函数参数使用常量指针时，可以避免函数内部不小心改变了指针所指内容。
    3. 定义**常引用**（**不能通过常引用修改其引用的变量**）  
        `ERROR: const int& r = n; r = 5;`
2. const的值必须被初始化，除非是在声明`extern`中。
3. 编译器将const定义的东西看作`complier symbol table`中的内容，不是真实的变量。
4. 有时候，一些const的定义其实并不是常量，编译器可能并不知道具体值。
    ```C++
    //example1.
    //编译器此时并不知道size具体大小，尽管它是const，所以无法用来开辟空间
    int x;
    cin>>x;
    const int size = x;
    double a[size];

    //example2.
    //定义完全正确，但是切不能视其为已知的常量，
    //因为在此文件下，编译器仍不知道其具体值.
    extern const int x;
    ```
---
## 动态内存分配
1. `new`运算符（返回相应类型**指针**）
    1. 分配一个变量-->`int* pn = new int;`  
        动态分配出`sizeof(int)`字节的内存空间，返回这段内存空间的起始地址。
    2. 分配一个数组-->`int* pn = new int[N]`  
        动态分配出`sizeof(int)*N`字节的内存空间，返回这段内存空间的起始地址。
        ```C++
        //eg.
        int* pn = new int[20];
        pn[0]=100;
        ```
    3. 分配一个对象或多个对象
        ```
        //单个对象
        Student* q = new Student();
        delete q;//销毁对象，会调用Student的析构函数
        //多个对象
        Student* q  = new Student[10];
        delete q;//只会收回一个对象，其它9个对象的内存都还在
        delete[] q;//会调用所有对象的析构函数
        ```
2. `delete`运算符
    1. 使用`new`动态分配的内存空间，**一定要**用`delete`进行释放（free）。
        > 如果条件符合，以后可以把这个步骤放在析构函数中，保证空间被释放。
        ```C++
        //变量
        int* p = new int;
        *p = 5;
        delete p;
        //数组
        int*p = new int[10];
        p[0] = 5;
        delete[] p;
        ```
     2. 同一片空间不能一次被重复`delete`。
     3. 用`delete`来释放一个`nullptr`是安全的。
        ```C++
        int* p = 0;
        delete p;// It's fine.
        ```
---
## 内联函数
```C++
inline int Max(int a, int b)
{
    ...
}
```
1. Overhead for a **function call** --> 函数调用的额外开销
    1. Push paremeters
    2. Push return address
    3. Prepare return value
    4. Pop all push
2. 使用`inline`来降低运行时的开销。因为`inline`，编译器会减少汇编的操作，直接将整个函数代码插入到调用语句处，而不会产生调用函数的汇编语句。
3. 如果函数本身很简单，但如果被多次执行，则相比之下调用这个函数的开销就会显得比较大，所以用`inline`来解决这个问题。
4. `inline`就是用空间换时间。
5. 内联函数不会产生`.obj`文件。
6. 比C的宏更好，因为`inline`可以有编译器进行类型检查，更安全。
---
## 函数重载 overload
1. 函数名字相同，参数个数或参数类型不相同。
2. 在`类`的部分会再详细谈一下重载。
---
## 函数的缺省参数
1. **定义函数时**，让**最右边**连续若干个参数有缺省值。那么调用时相应位置可以不写参数，默认为缺省值。
2. 切不可只在中间缺省！














