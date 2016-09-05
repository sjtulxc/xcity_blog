---
title: "C++对象内存布局"
tags: [C++]
---



最近对C++对象模型进行了比较全面的学习，以下为学习时做的一些笔记。

### 虚函数表

C++中虚函数的作用主要是为了实现多态机制。多态，简单来说，是指在继承层次中，父类的指针可以具有多种形态——当它指向某个子类对象时，通过它能够调用到子类的函数，而非父类的函数。

我们定义一个基类：

{% highlight c++ %}
class Base
{
public:
 
    Base(int i) :baseI(i){};
 
    virtual void print(void){ cout << "调用了虚函数Base::print()"; }
 
    virtual void setI(){cout<<"调用了虚函数Base::setI()";}
 
    virtual ~Base(){}
 
private:
 
    int baseI;
 
};
{% endhighlight %}

当类定义了虚函数，或者**其父类有虚函数**时，为了支持多态机制，编译器将为该类添加一个虚函数指针（vptr）。**虚函数指针一般都放在对象内存布局的第一个位置上**，这是为了保证在多层继承或多重继承的情况下能以最高效率取到虚函数表。

因为vptr位于对象内存最前面，我们可以通过获得对象地址来获取虚函数指针的地址。

{% highlight c++ %}
Base b(1000);
int * vptrAdree = (int *)(&b);  
{% endhighlight %}

或者通过强制转换为函数指针访问函数：

{% highlight c++ %}
typedef void(*Fun)(void);
Fun vfunc = (Fun)*( (int *)*(int*)(&b));
vfunc();
{% endhighlight %}

同理第二个虚函数地址为` (int *)(*(int*)(&b)+1)`

### 对象模型

在C++中有static, nonstatic两种数据成员以及static, nonstatic, virtual三种类成员函数。

C++编译器一般采用如下模型处理类对象：

nonstatic 数据成员被置于每一个类对象中，而static数据成员被置于类对象之外。static与nonstatic函数也都放在类对象之外，而对于virtual 函数，则通过虚函数表+虚指针来支持，具体如下：

- 每个类生成一个表格，称为虚表（virtual table，简称vtbl）。虚表中存放着一堆指针，这些指针指向该类每一个虚函数。虚表中的函数地址将按声明时的顺序排列，不过当子类有多个重载函数时例外，后面会讨论。
- 每个类对象都拥有一个虚表指针(vptr)，由编译器为其生成。虚表指针的设定与重置皆由类的复制控制（也即是构造函数、析构函数、赋值操作符）来完成。vptr的位置为编译器决定，传统上它被放在所有显示声明的成员之后，不过现在许多编译器把vptr放在一个类对象的最前端。关于数据成员布局的内容，在后面会详细分析。
- 另外，虚函数表的前面设置了一个指向type_info的指针，用以支持RTTI（Run Time Type Identification，运行时类型识别）。RTTI是为多态而生成的信息，包括对象继承关系，对象本身的描述等，只有具有虚函数的对象在会生成。

假如我们有如下基类：

{% highlight c++ %}
class Base
{
public:
 
    Base(int i) :baseI(i){};
  
    int getI(){ return baseI; }
 
    static void countI(){};
 
    virtual void print(void){ cout << "Base::print()"; }
 
    virtual ~Base(){}
 
private:
 
    int baseI;
 
    static int baseS;
};
{% endhighlight %}

那么Base类的对象模型如下图

![](http://i.imgur.com/9rtgoFs.png)

假如现在有Base类对象p，那么

- 通过`(int *)(&p)`可取得虚函数表的地址
- 通过`((int)(int*)(&p) – 1))`可取得type_info信息，可强制转换为`RTTICompleteObjectLocator`对象
- 虚表指针的下一位置为nonstatic数据成员，可通过`(int*)(&p) + 1`获取地址


### 单继承

在C++对象模型中，对于一般继承（这个一般是相对于虚拟继承而言），若子类重写（overwrite）了父类的虚函数，则子类虚函数将覆盖虚表中对应的父类虚函数(注意子类与父类拥有各自的一个虚函数表)；若子类并无overwrite父类虚函数，而是声明了自己新的虚函数，则该虚函数地址将扩充到虚函数表最后（在vs中无法通过监视看到扩充的结果，不过我们通过取地址的方法可以做到，子类新的虚函数确实在父类子物体的虚函数表末端）。而对于虚继承，若子类overwrite父类虚函数，同样地将覆盖父类子物体中的虚函数表对应位置，而若子类声明了自己新的虚函数，则编译器将为子类增加一个新的虚表指针vptr，这与一般继承不同,在后面再讨论。

我们定义如下派生类：

{% highlight c++ %}
class Derive : public Base
{
public:
    Derive(int d) :Base(1000),      DeriveI(d){};
    //overwrite父类虚函数
    virtual void print(void){ cout << "Drive::Drive_print()" ; }
    // Derive声明的新的虚函数
        virtual void Drive_print(){ cout << "Drive::Drive_print()" ; }
    virtual ~Derive(){}
private:
    int DeriveI;
};
{% endhighlight %}

则对于这个派生类而言，它的对象模型应该如下图所示：

![](http://i.imgur.com/uyxDzil.png)

需要注意的一点是，当父类的函数为虚函数时，子类重写的函数便默认为虚函数，不需要加virtual字段声明。

### 多继承

在这里只讨论简单的多继承（非菱形继承），在多继承中，如果我们有两个基类Base和Base_2，那么多继承的子类的对象模型大概如下图所示：

![](http://i.imgur.com/0EaMb23.png)

具体在这里就不详细展开了。

### 虚继承

虚继承的派生类的内存布局与普通继承很多不同，主要体现在：

- 虚继承的子类，如果本身定义了新的虚函数，则编译器为其生成一个虚函数指针（vptr）以及一张虚函数表。该vptr位于对象内存最前面。
（非虚继承：直接扩展父类虚函数表）
- 虚继承的子类也单独保留了父类的vprt与虚函数表。这部分内容接与子类内容以一个四字节的0来分界。
- 虚继承的子类对象中，含有四字节的虚表指针偏移值（该类没有vptr时该值为0，有vptr时该值为-4）。

假如现在有派生类B1虚继承基类B，则B1类对象的对象模型应该如下：

![](http://i.imgur.com/hey6Rwa.png)

而对于菱形继承，最派生类D类的对象模型又有不同的构成了。在D类对象的内存构成上，有以下几点：

- 在D类对象内存中，基类出现的顺序是：先是B1（最左父类），然后是B2（次左父类），最后是B（虚祖父类）
- D类对象的数据成员id放在B类前面，两部分数据依旧以0来分隔。
- 编译器没有为D类生成一个它自己的vptr，而是覆盖并扩展了最左父类的虚基类表，与简单继承的对象模型相同。
- 超类B的内容放到了D类对象内存布局的最后。

菱形虚拟继承下派生类D的C++对象模型为：

![](http://i.imgur.com/NIHFxOs.png)

### 关于C++封装成本

由于成员函数虽然在类中声明，但却不出现在类对象中，所以类封装相比起结构体并没有带来任何空间或执行期的效率影响。而在下面这种情况下，C++的封装额外成本才会显示出来：

- 虚函数机制（virtual function） , 用以支持执行期绑定，实现多态。
- 虚基类 （virtual base class） ，虚继承关系产生虚基类，用于在多重继承下保证基类在子类中拥有唯一实例。

此外，C++中处在同一个访问标识符（指public、private、protected）下的声明的数据成员，在内存中必定保证以其声明顺序出现。而处于不同访问标识符声明下的成员则无此规定。

### 关于RTTI

最后再对C++的RTTI(Runtime Type Information)做个简单总结，对于C++的RTTI主要使用就是typeid函数和dynamic_cast函数。

typeid()实际上返回type_info类，类定义如下:

{% highlight c++ %}
class type_info
{
public:
    virtual ~type_info();
    bool operator==(const type_info& _Rhs) const; // 用于比较两个对象的类型是否相等
    bool operator!=(const type_info& _Rhs) const; // 用于比较两个对象的类型是否不相等
    bool before(const type_info& _Rhs) const;

    // 返回对象的类型名字，这个函数用的很多
    const char* name(__type_info_node* __ptype_info_node = &__type_info_root_node) const;
    const char* raw_name() const;
private:
    void *_M_data;
    char _M_d_name[1];
    type_info(const type_info& _Rhs);
    type_info& operator=(const type_info& _Rhs);
    static const char * _Name_base(const type_info *,__type_info_node* __ptype_info_node);
    static void _Type_info_dtor(type_info *);
};
{% endhighlight %}

关于typeid，常用方式为使用name()函数返回对象类型名称，有一点需要注意：**当类中不存在虚函数时，typeid是编译时期的事情，也就是静态类型。当类中存在虚函数时，typeid是运行时期的事情，也就是动态类型。**

关于typeid的另一种使用方式为使用type_info类中重载的==和!=比较两个对象的类型是否相等
这个会经常用到，通常用于比较两个带有虚函数的类的对象是否相等，具体就不展开了。

而dynamic_cast主要用于在多态的时候，它允许在运行时刻进行类型转换，从而使程序能够在一个类层次结构中安全地转换类型，把基类指针（引用）转换为派生类指针（引用）。如以下代码：

{% highlight c++ %}
A *pA = new C;
//C *pC = pA; // Wrong
C *pC = dynamic_cast<C *>(pA);
{% endhighlight %}

如果直接转换编译器会提示错误，这时候就需要使用dynamic\_cast来做类型转换，如果转换失败，指针转换则返回NULL，引用转换则抛出bad\_cast异常。