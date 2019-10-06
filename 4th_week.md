# 4th_week

[TOC]

---
## 运算符重载  Overloading Operators
1. 只有已经存在的运算符可以被重载。
2. 运算符重载必须：
    1. 保持运算的成员数量不变
    2. 运算优先级不变
3. 运算符重载的目的，就是扩展运算符的使用范围，使之能够直接作用于对象。  
    最简单的例子就是实现**复数**的运算。因为复数有实部和虚部，所以普通的运算符不能直接使用。那么我们就可以将复数作为一个类，利用运算符重载来完成运算。
4. 运算符重载的实质是**函数重载**。可以重载成普通的全局函数，也可以重载为成员函数。
    > 实际上在使用运算符时，就变成了对运算符函数的调用。
5. 形式
    ```
    返回值类型 operator 运算符 (形参表)
    {
        ...
    }
    ```
6. 例子
    ```C++
    class Complex
    {
    public:
    	double real, imag;
    	Complex(double r = 0.0, double i = 0.0) :real(r), imag(i) {}
    	//以成员函数的形式重载**减号**
    	Complex operator-(const Complex& c)
    	{
    		return Complex(real - c.real, imag - c.imag);
    	}
    };
    //以全局函数的形式重载**加号**
    Complex operator+(const Complex& a, const Complex& b)
    {
    	return Complex(a.real + b.real, a.imag + b.imag);
    }
    ```
    仔细观察两个运算符重载函数，可以发现，作为成员函数的重载比作为全局函数的重载少一个参数。原因在于它们被调用时的实际形式不一样，其中**成员函数**是作用于**对象**的，其中有**运算符接收成员**(receiver)和**运算符算子**的概念，所以少一个参数。
    ```C++
    c = a + b;    -->    c = operator+(a, b);
    d = a - b;    -->    d = a.operator-(b);  此处a是减号运算符的receiver，b是算子。
    ```
    注意！这里有一个不同类型成员之间运算的问题。如果使用**成员函数**进行运算符重载，那么以运算符会以**receiver的运算符**为主，且**算子会类型转换至与receiver同类型**，然后再进行运算符运算。
    > 前提！！！  
    > 类中**有相应的构造函数**，能够使不同类型的算子生成同类型的临时对象。
    ```C++
    //假设x, y, z均为类A的对象，加号运算符以A的成员函数形式重载.
    z = x + y;  //OK.
                //x和y是同类型，那么直接z = x.operator+(y).
    z = x + 3;  //OK.
                //x是receiver，3是算子。其中3属于int，那么就会调用类A中
                //相应的构造函数，将其生成为一个与x类型相同的临时对象，
                //然后使用类A重载的operator+.
    z = 3 + x;  //Error！
                //此处3是receiver，那么会使用int类型的operator+，所以需要x去转换
                //为int型，但是int类型不包含类型转换至A的构造函数，所以此运算出错。
    ```
    而如果是使用**全局函数**进行运算符重载，那么对于双目运算符（binary operators），函数会有两个参数；对于单目运算符(unary operators)，函数会有一个参数。所以它会确定
    ```C++
    z = x + y;    //OK. x和y直接触发operator+(x, y).
    z = x + 3;    //OK. 3进行类型转换后，执行operator+
    z = 3 + y;    //OK. int型没有执行这两种类型的加法，
                  //但是重载的全局函数能够实现，所以3进行类型转换，执行operator+.
    z = 3 + 7;    //OK. 会先执行int类型的加法，然后对计算结果进行相应的类型转换。
    ```
7. 那么应该把运算符以什么形式重载呢？成员函数or全局函数。
    - 单目运算符最好以**成员函数**来进行重载。
    - =（）[] -> ->* 必须以**成员函数**来重载。
    - 其它双目运算符最好以**非成员函数**进行重载
    - 有时候运算符的重载函数需要访问到类的私有成员时，可以将其在类中声明为友元`friend`。
---
## 赋值运算符的重载
> 此部分建议观看翁恺老师的介绍。  
1. `=` 赋值运算符只能重载为成员函数！这与前面的运算符不同。
2. 赋值运算符的重载其实是**希望让其实现类型转换，以简化代码。**
3. 下面通过自己写一个不完整的String类来体会赋值运算符的重载。
    ```C++
    class MyString
    {
    public:
    	char* str;
    	MyString() :str(new char[1]) { str[0] = 0; }
    	const char* c_str() { return str; }
    	MyString& operator= (const char* s);
    	~MyString() { delete[] str; }
    };
    MyString& MyString::operator=(const char * s)
    {
    	delete[] str;
    	str = new char[strlen(s) + 1];
    	strcpy(str, s);
    	return *this;
    }
    int main()
    {
    	MyString s1;
    	s1 = "this";//等价于 --> s1.operator=("this");
    	cout << s1.c_str() << endl;
    }
    ```
    对于这个例子，有几个点：
    1. 可以看见`s1 = "this";`这个语句属于赋值语句，而`"this"`本身是一个`char[]`，所以它会调用`MyString`里面的赋值运算符重载函数来实现；
    2. 在这里，不能写`MyString s1 = "this";`，因为这样写**属于初始化语句而不是赋值语句**。如果这句话能正确执行，那么它是这样一个流程：
        - 首先会调用`MyString`里相应的构造函数来将`"this"`构造成一个临时的`MyString`对象。
        - 然后会调用相应的拷贝构造函数，利用临时对象来初始化对象`s1`。（详见`2nd_week_类型转换构造函数`）
    3. 但是，显而易见，`MyString`类中没有编写相应的构造函数。所以可以补充如下的构造函数来实现：
    ```C++
    MyString(const char* c)
	{
		if (!c) 
		{ 
			cout << "null initial" << endl;
			return;
		}
		delete[] str;
		str = new char[strlen(c) + 1];
		strcpy(str, c);
	}
    ```
    > 在这里这么婆婆妈妈，其实就想提醒一下**初始化**和**赋值**之间的差别。:shit:
4. 这个部分还有两个额外的思考的点：
    1. 对于上面`MyString`的例子，如果出现了如下的调用：
        ```C++
        MyString s;
        s = "hello";
        s = s;
        ```
        那么会出现错误。原因在于，调用赋值运算符重载函数时，我们第一个做的事情就是`delete[] str;`，这样就会把`s`原本的内存直接delete，导致出错。所以需要加个判断。
        > 要把使用者当傻子
    2. 讨论类型转换函数的一种形式。（**重载类型转换运算符**）  
        看看下面这么一个会引起**错误**:x:的例子。
        > 首先说明一下，下面例子中对后面定义的类进行前向声明。在使用**前向声明的类**或者 **结构体**的时候，只能用作指针或者引用。如果直接定义实例，是编译无法通过的。
       ```C++
       class B;//前向声明
       class A
        {
        public:
        	A(B*) {...};//因为类B前向声明未定义，故只能使用指针，不能使用实体对象
        };
        class B
        {
        public:
        	operator A() const {...} ;//重载类型转换运算符
        };
        void f(A) {}
        int main()
        {
            B b;
            f(&b);
        }
       ```
       可以看见A中有一个构造函数，可以实现B到A的类型转换。而B中重载了一个`A()`的强制类型转换运算符`operator A() const`，称作operator conversion，这种形式的定义没有返回类型，其**返回类型与函数名一致**。这个函数会在需要时被**自动调用(be called automatically)**。
       > 比如也可以写`operator double() const`，表示重载double类型的转换运算符。  
       > 
       同时，我们还定义了一个接受参数为A的空函数`f()`。  
       接下来看一下main函数，定义了一个B的对象，将其送入`f()`。此时，编译器既可以使用构造函数`A(B)`来实现类型转换，生成临时对象；也可以自动调用`operator A() const`来实现相应的类型转换。这两者没有优先级之分，所以编译器不知道怎么做，导致错误。  
       简单的解决方法：在构造函数`A(B)`前添加`explicit`关键字，以此声明此函数不能被编译器用于自动转换类型，只能做构造函数。  
       再来一个例子说明一下。
       ```C++
       class B;
       class A
        {
        public:
        	explicit A(B*) {...};
        };
        class B
        {
            //NULL
        };
        void f(A) {}
        int main()
        {
            B b;
            f( A(b) );
        }
       ```
       这个例子是OK的。可以看到，由于构造函数是`explicit`，所以为了实现`f(b)`，我们需要手动完成类型转换`A(b)`。  
       > 总结：这一个小点，除了介绍`explicit`和自动的类型转换函数，还想要说明的一点就是减少使用这种自动类型转换的东西。因为有时候可能你自己并没有想在这里进行类型转换，可是编译器却自动执行，这样子有可能会有风险。:innocent:

---
## 利用运算符重载来实现可变长整形数组
1. 其实好像就是简单实现一下`vector`。
2. 要实现的功能：
    1. 实现`push_back()`。使用动态分配的内存来存放数组元素。
    2. 实现同类型的赋值。重载`等号=`。
    3. 实现`int n = a[1]`和`a[1] = 10`。重载`方括号[]`。
    4. 实现`length()`。返回数组当前长度。
    5. 自己重写拷贝构造函数。以此实现**深拷贝**，而不是默认拷贝函数所实现的**浅拷贝**（只复制了指针）。
3. 类`MyArray`的基本结构。
    ```C++
    class MyArray
    {
    public:
    	MyArray(int l = 0);
    	MyArray(MyArray& a);//拷贝构造函数.
    	~MyArray();
    	
    	void push_back(int element);//在数组尾部添加元素element.
    	MyArray& operator= (const MyArray& a);//重载=. 实现赋值.既能当左值，又能当右值.
    	int& operator[](int index) { return ptr[index]; }//重载[]. 实现下标访问元素.
    	int length() { return size; }//返回数组长度.
    	
    private:
    	int size;//数组长度.
    	int* ptr;//动态分配数组的指针.
    };
    ```  
    
    构造、拷贝构造、析构函数的具体实现：
    ```C++
    MyArray::MyArray(int l) :size(l)
    {
    	if (l == 0)
    		ptr = NULL;
    	else
    		ptr = new int[l];
    }
    //实现深拷贝
    MyArray::MyArray(MyArray & a)
    {
    	if (a.ptr == NULL)
    	{
    		ptr = NULL;
    		size = 0;
    		return;
    	}
    	this->ptr = new int[a.size];
    	this->size = a.size;
    	memcpy(this->ptr, a.ptr, sizeof(int)*a.size);
    }

    MyArray::~MyArray()
    {
    	if (ptr)
    		delete[] ptr;
    }
    ```  
    
    `push_back()`的具体实现：
    ```C++
    /**
      下面例子是较为低效的一种实现方式
      而vector的实现思路类似于，可能一开始就分配32 bits的空间，然后当push_back时发现大小不够时，
      扩展到64 bits.
    */
    void MyArray::push_back(int element)
    {
    	if (this->ptr)
    	{
    		int* tmpPtr = new int[this->size + 1];
    		memcpy(tmpPtr, ptr, sizeof(int)*this->size);
    		delete[] ptr;
    		ptr = tmpPtr;
    	}
    	else
    	{
    		ptr = new int[1];
    	}
    	ptr[size++] = element;
    }
    ```  
    
    `等号重载`的具体实现：
    ```C++
    MyArray& MyArray::operator=(const MyArray & a)
    {
    	//防止自己给自己赋值
    	if (a.ptr == this->ptr)
    		return *this;
    	//赋值一个空的数组
    	if (a.ptr == NULL)
    	{
    		if (this->ptr)
    			delete[] this->ptr;
    		this->ptr = NULL;
    		this->size = 0;
    		return *this;
    	}
    	//如果赋值的数组比原来大，重新开内存空间再memcpy，否则直接memcpy就行
    	if (this->size < a.size)
    	{
    		if (this->ptr)
    			delete[] this->ptr;
    		ptr = new int[a.size];
    	}
    	memcpy(this->ptr, a.ptr, sizeof(int)*a.size);
    	this->size = a.size;
    	return *this;
    }
    ```

---
## 流插入运算符和流提取运算符的重载
1. `<<`和`>>`，本身是左移运算符和右移运算符。
2. 最典型的例子就是在`iostream`中，`cout<<`和`cin>>`。  
    例如，`cout`是`ostream`类的一个**对象**，而`ostream`类中对`<<`进行了重载，从而实现使用`cout<<`输出内容。
3. 利用前面Complex的例子来做个尝试：1.重载`<<`实现cout；2.重载`>>`实现cin.
    ```C++
    class Complex
    {
    public:
    	double real, imag;
    	Complex(double r = 0.0, double i = 0.0) :real(r), imag(i) {}
    	//以成员函数的形式重载**减号**
    	Complex operator-(const Complex& c)
    	{
    		return Complex(real - c.real, imag - c.imag);
    	}
    };
    //以全局函数的形式重载**加号**
    Complex operator+(const Complex& a, const Complex& b)
    {
    	return Complex(a.real + b.real, a.imag + b.imag);
    }
    //重载<<
    ostream& operator<<(ostream& os, Complex& c)
    {
        os<<c.real<<"+"<<c.imag<<"i";
        return os;//返回ostream对象，才能实现<<的叠加使用
                  //eg. cout<<...<<...<<...
    }
    istream& operator>>(istream& is, Complex& c)
    {
    	string s;
    	is>>s;
    	int pos = s.find("+",0);//找到+的位置
    	string sTmp = s.substr(0,pos);//提取出实部
    	c.real = atof(sTmp.c_str());
    	sTmp = s.substr(pos+1,s.length()-pos-2);//提取出虚部
    	c.imag = atof(sTmp.c_str());
    	return is;
    }
    ```
    可以看到，重载的函数是以`ostream`类和'的`istream`类的对象执行的。
---
## 自增、自减运算符的重载
1. 为方便，统一用`a++`和`++a`来描述`后置++`和`前置++`。
2. `++a`属于一元运算符；`a++`属于二元运算符，多了一个参数(没有使用的参数)用来区分。
    ```C++
    //重载为成员函数
    T& operator++();    //++a
    T operator++(int)   //a++
    //重载为全局函数
    T& operator++(T);   //++a
    T operator++(T, int)//a++
    ```
    要注意，`++a`返回的是**a的引用**，故可以作左值；`a++`返回的是一个**临时变量（对象）**，等于自加前的a，故不可以作左值。
3. 因为`++a`返回的是引用，所以当a是一个对象时，比如`迭代器iterator`，那么`++a`的效率会比`a++`的效率高。



