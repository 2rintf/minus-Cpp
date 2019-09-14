# 3rd_week

[TOC]

---
## thisָ��
1. ��ĳ�̶ֳ���˵��`C++`����Cʵ�ֵġ�ǰ��`2nd_week`Ҳ���ᵽ`C++`��������C��`struct`ʵ�ֵġ���ô���ڿ��Կ�һ����������ӣ����һ��C++�ĳ�Ա��������ôʵ�ֵġ�   
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

    C��ʵ�����£�  
    > ��Ҫע����ǣ���C�У��ṹ�岻��ӵ�г�Ա����������C++�У��ṹ���������û���˲��Ҳ�����ڽṹ����д���Ա������
    ```C
    //C++������C�еĽṹ��ʵ��
    struct A
    {
        int i;
    };
    //C��C++����ĳ�Ա������Ϊһ��ȫ�ֺ��������ǲ����ж�������һ��ṹ������ָ�롣
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
    ����������ӿ���֪������ĳ�Ա�����ڱ���ʱʵ����**����һ������**��ʵ�ʲ�������+1��
2. �ڷǾ�̬��Ա�����п���ʹ��`this`ָ�룬������ָ��ó�Ա�������õĶ����ָ�롣
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
        b = a.func();//����thisָ�������ص�ǰ**�������õĶ���a**��Ϊ����ֵ��
        return 0;
    }
    ```
3. �������������һ����˼��
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
        A* p = NULL;//��ָ��
        p->Hello1();//��Ȼ�ǿ�ָ�룬���Ǵ˴���ȫOK��
        p->Hello2();//Error��
    }
    ```
    ����`void Hello()`��ʵ�ȼ���`void Hello(A* this)`������ִ��`p->Hello()`ʱ���ȼ���`Hello(p)`��
4. **��̬��Ա����**�в���ʹ��thisָ��-->��Ϊ��̬��Ա������������������ĳ������
---
## static
1. ��̬��Ա���� & ��̬��Ա����
    1. ��������ȫ�ֱ��� & ȫ�ֺ���������һ�����󶼲����ڣ���ľ�̬��Ա����Ҳ���ڡ�
    2. һ�����**��̬��Ա����**�ɴ������еĶ���������̬��Ա����Ҳ����Ҫ����������ĳ������-->��ˣ���̬��Ա**����Ҫͨ������**���ܷ��ʡ�
        > �����ǰ�᣺�˳�Ա��������`�ɷ��ʵķ�Χ`������`public`֮��`main`��
    3. **��̬��Ա����**��Ҫ�ڶ�������ļ��жԳ�Ա��������һ��˵�����ʼ�������������ͨ�����������Ӳ���ͨ������VS�еı�������ͼ���ɼ�����ȷʵ�����⡣
       ![���ӳ��ִ���](./pic/20190909171545089_16405.png)
    4. **��̬��Ա����**�У�**����ֱ�ӷ���**�Ǿ�̬��Ա������Ҳ**����ֱ�ӵ���**�Ǿ�̬��Ա��������Ϊ��̬��Ա�������Բ�ͨ��������з��ʣ�������а����Ǿ�̬�ĳ�Ա���ͻ���ֻ��ң���֪���˷Ǿ�̬�ĳ�Ա�����ĸ�����
        > ��Ȼ���������ò�����ö�������ã�Ȼ����ȥ������Ӧ�ķǾ�̬��Ա������Ҫǿ�����ǲ���**ֱ��**ȥ���ʡ�
    5. ������һ������չʾ�й�`static`�����ݡ�
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
            B::showCounter();//ֱ�ӷ��ʾ�̬��Ա����
            B b2(10);
            b2.showCounter();//ͨ��������ʾ�̬��Ա����
            b1.printStaticNum();
            cout << B::num << endl;//ֱ�ӷ��ʾ�̬��Ա����
            b1.~B();
            b2.showCounter();
        }
        //�����1  2  55  55  1
        ```
        - ��Bӵ��������̬��Ա����������һ����`private`��һ����`public`�����Ǿ���`22,23`�н��г�ʼ����Ҳ����ֻ����������Bӵ��������̬��Ա������`showCounter()`������ֱ�Ӷ��壬`Print()`��������ж��塣
            > `static`������������������������ĵĳ�ʼ������Ͳ����ټ�`static`�ˡ�
        - ���ʾ�̬��Ա�ķ����ж��֣�
            - ����::��Ա��
            - ������.��Ա��
            - ָ��->��Ա��
            - ����.��Ա��
2. ����������ӻ�����һ�����⡣���Կ�������̬��Ա����`counter`��������**ʵʱ**ͳ���ж��ٸ�`class B`�Ķ�����Ϊ���ڹ��캯���н���`counter++`�����������������н���`counter--`��  
    ���Ǵ˴������ȱ�ݣ�û�п��ǵ���һЩ**��ʱ����**��ͨ��`�������캯��`���ɣ����`2nd_week`�����������ǵ�����û�м��㵽��������ʱ������������������������������`counter`�����Ĳ�׼ȷ��  
    ���Դ˴�����Ҫ�Լ���д�������캯��������ɿ���������ͬʱ������`counter++`������
---
## ��Ա����ͷ����
1. ���壺��**��Ա����**����з����(`enclosing`)��-->��������������Ķ���
2. �κ����ɷ����������䣬��Ҫ�ñ��������ף��˶����еĳ�Ա����Ӧ����γ�ʼ����
    ���������һ������࣬���г�Ա������ô����ʹ��**��ʼ���б�**`(���2nd_week)`����ʼ����Ա���󣬶������ڷ����Ĺ��캯��������ʼ�������Ǵ˳�Ա��������Ȼʹ��Ĭ�Ϲ��캯������ô�����Բ���ʼ����
3. ����
    ```C++
    class Tire//��̥��
    {
    private:
        int radius;
        int width;
    public:
        Tire(int r, int w):radius(r), width(w){}
    };
    class Engine{};//�����ࣨ�˴��Ǹ����࣬��ֻ��Ĭ�Ϲ��캯����
    class Car//���ࣺ����̥��������ɡ�
    {
    private:
        int price;
        Tire tire;//��Ա����
        Engine engine;//��Ա����
    public:
        //���ó�ʼ���б����г�Ա����ĳ�ʼ����
        //��ΪEngine��Ĭ�Ϲ��캯�������Բ����ڴ˳�ʼ����
        Car(int p, int tr, int tw):price(p),tire(tr, tw){}
    };
    int main()
    {
        Car car(20000,17,225);
        return 0;
    }
    ```
4. ����๹�캯��������������**ִ��˳��**��C++��׼ȷ���ģ�
    1. **��Ա����Ĺ��캯��**����**����౾���Ĺ��캯��**ִ�С�ԭ���м�����
        - ��ʼ���б���ִ���ڹ��캯��֮ǰ�����������г�ʼ���ĳ�Ա��������ִ���乹�캯����
        - �߼��Ͻ���������п����ڹ��캯����ʹ�õ���Ա���󣬹ʳ�Ա������ִ�С�
    2. ��Ա����Ĺ��캯�����ô�������**�������Ĵ���**һ�£������ڳ�ʼ���б���Ĵ����޹ء�
    3. ��������������������ִ�з�����������������ִ�г�Ա����������������빹�캯�����ô����෴����Ϊ����������������л��п���ʹ�õ���Ա����
5. ̸һ�·����Ŀ������캯����  
    ��ʵ����Ҳ��һ���ģ�������������ͨ��`������ʼ��`����ʱ������Ӧ��ִ��**�����Ŀ������캯��**��**��Ա����Ŀ������캯��**���Ͳ���ִ�й��캯���ˡ�
---
## �������� & ������Ա����
1. ʹ��`const`�ؼ�����ʵ�֡�
    ```C++
    //��������
    const A a;
    //������Ա����
    class A 
    {
        ...
        void getX() const {...}
        ...
    };
    ```
2. ��������һ��**��������**����˶����е�ֵ��**���ܱ��޸�**��Ҳ����ִ��**�ǳ�����Ա����**��
3. ����Ա������һ��**������Ա����**�������в����޸ĳ�Ա������ֵ�����ܵ��÷ǳ�����Ա������
    > ����̬��Ա�����뾲̬��Ա�����⣬���ǡ���Խ�˶��󡱡�:-)
4. ֻҪ�г������󣬱�����ֻ������ִ�г�����Ա��������ʹ�ǳ�����Ա����û�������޸�ֵ�ľٶ���Ҳ������ִ�С�
5. ����
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
        const A a;     //��������
        a.doubleX();   //ERROR!
        a.nullFunc();  //ERROR!
        a.getX();      //OK.
    }
    ```
6. ������Ա���������ֺͲ�������һ������������һ����`const`����ô��������**����**`overload`��ϵ��
---
## friend  ��Ԫ
1. ��Ԫ����������`A`������ȫ�ֺ�������һ����ĳ�Ա����Ϊ`friend`����ô��Щ�����Ϳ��Է�����`A`��`private`��Ա��
    > ��Ա������Ҳ�������캯��������������
2. ��Ԫ�ࣺ����`A`��������`B`��`friend`�ģ���ô����������`B`�е���`A`����Ϳ���ֱ�ӷ�����`A`��`private`��Ա��
    ```C++
    class A
    {
        friend class B;//����˳������ν��
    private:
        int i;
    ...
    };
    class B
    {
        A a;//���ʼ�����ڳ�ʼ���б�����ɣ����**�����**֪ʶ�㡣
        ...
        void Print() { cout << a.i << endl; }//���Է��ʶ���a��˽�г�Աi��
    };
    ```
3. ��Ԫ��ϵ���ܴ��ݣ����ܼ̳С�  
    ����A��B����Ԫ��B��C����Ԫ������˵A��C����Ԫ��








