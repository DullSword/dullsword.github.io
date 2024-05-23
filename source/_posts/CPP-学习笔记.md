---
title: C++ 学习笔记
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 1
date: 2021-12-14 08:02:05
categories: CPP
tags:
---
## 内存区域

- 栈区
- 堆区
- 常量区
- 全局静态区
- 代码区

---

## 编译过程

1. 预处理（Preprocessing）：在这个阶段，预处理器会处理源代码中的预处理指令，如 `#include` 、 `#define` 等。例如， `#include` 指令会被用来插入对应的头文件内容， `#define` 定义的宏会被展开。
2. 编译（Compilation）：编译器会将预处理后的源代码转换为汇编语言。在这个过程中，编译器会检查代码的语法错误，并进行诸如优化等操作。
3. 汇编（Assembly）：汇编器将编译生成的汇编代码转换为目标文件，也就是机器语言代码。
4. 链接（Linking）：链接器将多个目标文件以及库文件链接成一个可执行文件。在这个过程中，链接器会解析未定义的符号引用，确保所有的函数和变量引用都能找到对应的定义。

---

## 整数

- 为什么整数要有那么多种？
    为了准确表达内存，做底层程序的需要。
- 没有特殊需要，就选择 `int`
    现在的CPU的字长普遍是32位或64位，一次内存读写就是一个 `int`，一次计算也是一个 `int`，选择更短的类型不会更快，甚至可能更慢。现代的编译器一般会设计内存对齐，所以更短的类型实际在内存中有可能也占据一个 `int` 的大小（虽然 `sizeof` 告诉你更小)。

    ![int](int.png)

    <a class="tag is-dark is-medium" style="margin-bottom: 1rem" href="https://www.icourse163.org/learn/ZJU-9001?tid=9001#/learn/content?type=detail&id=84018&cid=101079" target="_blank">
    <span class="icon"><i class="fas fa-camera"></i></span>&nbsp;&nbsp;
    来源自 C语言程序设计 ——6.1.6选择整数类型 翁恺
    </a>

- `unsigned` 与否只是输出的不同，内部计算是一样的
  - [The real difference between "int" and "unsigned int"](https://stackoverflow.com/a/16381089/12478805)
  - [C中unsigned int和int有什么区别？](https://segmentfault.com/q/1010000009639067)

---

## 指针

### 用指针来做什么

- 需要传入较大的数据用作参数时
- 传入数组后对数组做操作
- 函数想返回不止一个结果，也就是需要用函数来修改不止一个变量
- 接收动态申请的内存

可以对指针用下标运算符[]，是因为可以把指针当作数组的首元素地址，但只有[0]是正确的，如果将后续数据当作数组元素则是错误的。就像可以把一个整数当作是地址来看待，也可以将地址当作整数来看待，但不一定是正确的。

### 双重指针

常见于在函数中修改指针：

```cpp
#include <iostream>

using namespace std;

void allocateMemory(int** ptr) {
    *ptr = new int(10);
}

int main() {
    int* p = nullptr;

    allocateMemory(&p);  // 传递指针的地址

    cout << *p << endl;  // 输出 10

    delete p;

    return 0;
}
```

通过将指针的地址传递给函数，我们可以在函数中修改指针的值。

---

## 数组

数组名是一个常量对象，不是指针，在传参的时候会退化为指针。用法上 `a` 和 `&a[0]` 一致，但数组名 `a` 不是指针。

- int a[5]
  - `sizeof(a)` 得到的是数组长度20，而不是 `int` 指针的长度4
  - `&a` 得到的是 `int (*)[5]`

---

## 字符串

### char*

```cpp
char* str = "hello";
```

其中 `"hello"` 是一个字符串常量，它被存储在内存的只读部分。而 `str` 是一个指针，它指向这个字符串常量的首地址，不能通过它来修改 `"hello"` 。

### char str[]

```cpp
char str[] = "hello";
```

其中 `"hello"` 被视为一个字符数组，它被存储在栈上。这个数组的每个元素都可以被修改。

### string

```cpp
#include <iostream>
#include <string>

using namespace std;

int main () {
    string str1 = "hello";

    str2 = str1;
    cout << "str2 : " << str2 << endl;
    
    str3 = str1 + str2;
    cout << "str1 + str2 : " << str3 << endl;

    return 0;
}
```

---

## static

### 变量

静态变量实际上是特殊的全局变量，和全局变量位于相同的内存区域

- 静态本地变量
  - 具有全局的生存期，函数内的局部作用域
- 静态成员变量
  - 具有全局的生存期，只不过只有类的局部作用域。
  - 类中的静态成员变量只是声明，需要在cpp中对它进行定义。只能在定义处对静态成员变量进行初始化。
    {% raw %}<article class="message is-info"><div class="message-body">{% endraw %}
    C++17 可以添加 `inline` 让静态成员变量成为内联的静态成员变量，就可以在类定义时初始化这个静态成员变量了。
    `inline static int i = 10;`
    {% raw %}</div></article>{% endraw %}

    ``` cpp
    #include <iostream>
    using namespace std;

    class A {
    public:
        A(){    //初始化列表只能对非静态成员变量进行初始化
            i = 10;
        }
        
        void Print(){
            cout << i << endl;
        }
        
    private:
        static int i;   //声明i，实际上只是告诉编译器，有一个全局变量i，它是我的成员变量
    };

    int A::i; //定义i（可以在这里做初始化 int A::i = 10 ），缺少这行代码会编译报错 "undefined reference to `A::i'"

    int main()
    {
        A a;
        a.Print();

        return 0;
    }
    ```

### 函数

- 静态成员函数
  - 没有 `this` 指针（不能在函数内使用 `this` ），只能访问静态成员（包括静态成员变量和静态成员函数）。

---

## #include

`#include` 会把那个文件的全部文本内容原封不动地插入到它所在的地方。
`#include` 只是为了让编译器知道函数的原型，保证你调用时给出的参数值是正确的类型。

- `#include "xx.h"`：优先在当前目录中搜索，如果没有，到编译器指定的目录去找
- `#include <xx>`：在标准库头文件目录搜索

---

## 头文件结构

运用条件编译或宏，保证这个头文件在一个编译单元中只会被 `#include` 一次。`#pragma once` 也能起到相同的作用，但不是所有的编译器都支持。

``` cpp
#ifndef __LIST_HEAD__
#define __LIST_HEAD__

#include "node.h"

typedef struct _list {
    Node* head;
    Node* tail;
} List;

#endif
```

---

## this：隐藏的参数

`this` 是所有非静态成员函数的隐藏参数，具有类的类型。

`void Point::print()` 可视为 `void Point::print(Point *p)`。

---

## 对象内存分配

内存空间是进了作用域就分配了，但构造函数要运行到那一行才调用。

---

## 字节对齐

- 按照数据成员中字节最长的长度来对齐
- 如果数据成员中字节最长的长度超过操作系统字长（32位为4，64位为8），则按照系统字长来对齐
- 数据成员的起始地址必须是其自身大小的整数倍
- 不满情况，需要填充字节

---

## 访问限制

访问控制只是对类来说，同一个类的不同对象是可以通过类函数互相访问到对方的私有属性。

``` cpp
#include <iostream>
using namespace std;

class A {
    int data;

public:
    A(int data) :data(data) { }

    void getData(const A* a) const
    {
        cout << a->data << endl;
    }
};

class B {
    int data;

public:
    B(int data) :data(data) { }

    //报错，无法访问 private 成员(在“A”类中声明)
    void getData(const A* a) const
    {
       cout << a->data << endl;
    }
};

int main()
{
    A a(10);
    A aa(20);
    //输出20
    a.getData(&aa);
    //报错，无法访问 private 成员(在“A”类中声明)
    //cout << a.data << endl;
    return 0;
}
```

---

## 类 vs 结构体

`class` 里的成员变量和成员函数默认为 `private` 访问类型， `struct` 则相反。

---

## const

### const int * p

`const` 在 `*` 号前修饰类型，表明不能通过该指针修改所指向的对象，指针本身可以再指向其他对象，指针所指向的对象如果不是 `const` 类型，则该对象是可以通过其他方式（比如赋值）进行修改。

``` cpp
const int * p = &i;
*p = 26; //ERROR
i = 26;  //OK
p = &j;  //OK
```

### int * const p

`const` 在 `*` 号后修饰指针，表明该指针不能再指向其他对象。

``` cpp
int * const q = &i;
*q = 26; //Ok
q++;     //ERROR
```

### 函数中的const

- `const` 修饰返回值，不能被当作左值，即不能被赋值。多用在返回值是引用或指针类型的情况下，为了避免所指向的对象被修改
- `const` 修饰参数，表明该函数不会修改参数对象。多用在参数值是引用或指针类型的情况下。因为传值的话已经是在操作副本，不影响原数据，但在函数内尝试对该形参进行修改，依旧会编译报错
- `const` 修饰 `this`，也就是在函数名()后加 `const` 关键词，表明是只读函数（又有称常量函数），即函数不会修改调用对象，其实是不会修改调用对象的成员变量（成员函数并不在对象内存中）；普通的类对象可以调用只读成员函数，而类只读对象（`const A a`）**只能**调用只读成员函数(`void Print() const {}`)

---

### constexpr

用于指示常量表达式。它的主要目标是通过在编译时而不是运行时进行计算来提高程序的性能。

虽然在一些情况下，不使用 `constexpr` ，编译器也可能在优化级别足够高的情况下，会进行"常量折叠"优化，即把可以在编译时计算的表达式替换为表达式计算后的结果。但是使用 `constexpr` 可以使程序不依赖于编译器优化，其次使用 `constexpr` 可以表明意图，使代码更清晰、更易读。而且由于 `constexpr` 表达式的值在编译时就已经确定，因此任何错误（如类型错误或溢出）都会在编译时被捕获，而不是在运行时，这可以更早地发现和修复错误。

#### constexpr 函数

函数被声明为 `constexpr` ，但当它的参数在编译时不能确定值，那么这个函数就会像普通函数一样在运行时执行。

#### 何时应该使用 constexpr

何时应该使用 `constexpr` 的具体判断标准，可以自行查阅其他文章。鉴于缺乏经验，所以我采用一种简单的判断方法：如果一个表达式所依赖的所有值在编译时期就已经确定，那么这个表达式就可以被声明为 `constexpr` 。

---

## 引用

本质上是 `const pointer`，不能再指向其他变量，类似于一种别名，必须初始化。没有引用的引用，`int x; int &a=x; int &b=a;` 是非法的。

### 使用场景

函数参数，使用引用的话就相当于直接使用原始数据，而不是复制一份数据；传值会用实参拷贝构造形参。使用引用可以减少开销，提高运行效率。

---

## 运算符优先

- `int *p[10]` ， `p` 先和 `[]` 结合， `p` 是一个数组，里面存了10个 `int` 类型指针
- `int (*p)[10]` ，`p` 是一个指针，指向一个大小为10的 `int` 数组
- `int *p(int)` ，是一个函数声明，函数名为 `p`，返回类型为 `int*` 类型，参数为 `int` 类型
- `int (*p)(int)` ，是一个指向函数的指针，该函数返回类型为 `int` 类型，参数为 `int` 类型

---

## 运算符重载

我的理解就是运算符重载是为了方便，其实也可以不用运算符重载，而使用显式的函数调用，例如：

`T Add(const T& addend) const { return this.data + addend.data; }`

运算符重载有两种方式，一种是非成员函数重载，即使用友元函数形式；一种是成员函数重载。

下列运算符只能通过成员函数进行重载：

- `=`：赋值运算符
- `( )`：函数调用运算符
- `[ ]`：下标运算符
- `->`：箭头运算符

### 使用场景

两个同类对象进行赋值（不是初始化，初始化是调用拷贝构造），会按成员复制，如果类成员包含指针，这时可以重载赋值运算符进行处理。

### 区分前缀和后缀

后缀采用 `int` 参数，编译器将传入 `0`。跟类无关，这个参数只是起标识作用，在函数内不使用。

``` cpp
class Integer(){
public:
    ...
    const Integer& operator++();    //prefix++
    const Integer operator++(int);  //postfix++
    const Integer& operator--();    //prefix--
    const Integer operator--(int);  //postfix--
    ...
}
```

``` cpp
const Integer& Integer::operator++(){
    *this += 1;
    return *this;
}

const Integer Integer::operator++(int){
    Integer old = *this; //用同类型对象初始化，触发拷贝构造
    ++( *this );
    return old;
}
```

前置运算符理论上性能更好点，尤其是自定义数据类型。但如果没有涉及赋值行为，编译器**可能**会将后置优化成前置。

---

## 函数对象

如果一个类将 `()` 运算符重载为成员函数，这个类就称为函数对象类，这个类的对象就是函数对象。函数对象是一个对象，但是使用的形式看起来像函数调用，实际上也执行了函数调用，因而得名。大多时候可以被 `Lambda` 表达式代替。

函数对象可以有成员变量，可以用来记录调用状态。

---

## 拷贝构造

`T::T(const T&)`

以下情况会触发拷贝构造：

- 按值将对象传递给函数
- 函数按值返回对象
- 用对象初始化同一类的对象时

如果类成员包含指针，默认的拷贝构造只会按成员复制，也就是浅复制，即2个对象的成员指针都指向同一块内存，那么就会出现重复释放的问题，这时候就得自定义拷贝构造。

``` cpp
class A{};

class B{
    A* a;
public:
    B(){
        a = new A();
    }
    ~B(){
        delete a;
    }
};

int main(){
    B b;
    B bb = b; //触发默认拷贝构造函数
}

//ERROR     free(): double free detected in tcache 2
```

---

## 构造函数内赋值 vs 构造函数初始化列表

内置类型没有差别(存疑)，自定义类可能有较大差别。

``` cpp
public : Thing(int _foo, int _bar): member1(_foo), member2(_bar){}

public : Thing(int _foo, int _bar) : member1(), member2(){
    member1 = _foo;
    member2 = _bar;
}
```

区别在于初始化列表的部分：

- 构造函数初始化列表：直接在初始化列表部分调用拷贝构造函数进行初始化
- 构造函数内赋值：先在初始化列表部分调用对应类型的默认构造函数进行初始化，然后在构造函数内再对成员变量进行赋值

所以构造函数初始化列表的方式更有效率。而且 **`const` 成员变量或引用类型的成员变量**必须使用构造函数初始化列表进行初始化，其次**父类的构造函数**只能在初始化列表中调用。

相关：

- [Initializing fields in constructor - initializer list vs constructor body](https://stackoverflow.com/questions/9903248/initializing-fields-in-constructor-initializer-list-vs-constructor-body)
- [Difference between initializer and default initializer list in c++](https://stackoverflow.com/questions/5816218/difference-between-initializer-and-default-initializer-list-in-c/5816311)
- [Should my constructors use “initialization lists” or “assignment”?](https://isocpp.org/wiki/faq/ctors#init-lists)

---

## 类内初始化 vs 构造函数初始化列表

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： jk_v1
原文标题： C++类内初始化与初始化列表
链接： <https://segmentfault.com/q/1010000003837264>
来源： SegmentFault 思否
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

如@GAO 所说，C++11的类内初始化允许`非static` 成员的初始化，可以用 `{}` 或 `=` 号（*笔者注：详情见下列相关*）。
构造函数的初始化列表与类内成员初始化没有谁好谁不好，谁来替代谁，两种方法可相互补充使用。类内初始化有一些好处：

1. 当你有多个构造函数时，如果使用初始化列表，每个构造函数都要写一遍，烦人不说，同时产生重复代码，修改易漏。如果把这些成员都用类内初始化，初始化列表就不用再列出它们了。
2. 类内初始化，成员之间的顺序是隐式的，会有些便利。如果使用初始化列表，它是有顺序之分的，顺序不对，编译器会警告。
3. 对于简单的类或结构，没有构造函数的，可以直接用类内初始化在成员声明的同时直接初始化，方便。

对于一些类类型的成员初始化要小心，如果成员之间有依赖关系，这时使用初始化列表显式的指明这些成员的构造（初始化）顺序是比较稳妥的。

如果成员已经使用了类内初始化，但在构造函数的初始化列表又列出来，编译器以后者优先，类内初始化会被忽略。如果某些成员使用不同构造函数时，会有不同的默认值，这种情况就要用初始化列表。同时，其它成员依然可以使用类内初始化。

类内初始化绝对不是解决什么内置类型默认初始化时未定义问题。面向对象编程一个很重要的原则，程序员有责任要保证对象产生出来，它的每个成员都必须是初始化的，这是设计问题以及基本意识，无论是使用哪种方法初始化。

相关：

- [Non-static data member initializers - modern-cpp-features](https://github.com/AnthonyCalandra/modern-cpp-features/blob/master/CPP11.md#non-static-data-member-initializers)
- [Static Data Member Initialization](https://stackoverflow.com/questions/11300652/static-data-member-initialization)
- [(Non) Static Data Members Initialization, from C++11 till C++20](https://www.cppstories.com/2015/02/non-static-data-members-initialization/)

---

## new对象 vs 直接声明对象

一般直接声明对象就行，因为使用 `new` 申请内存空间有开销，而且需要 `delete`，但直接声明的形式不适合以下情况：

- 需要大量内存的对象
- 该对象的生存周期应当比当前作用域更长
- 需要动态申请内存的情况（比如数组的长度是未知的）。

---

## 函数重载

一个类中出现同名不同参数列表的函数。

构不构成函数重载，与函数的返回类型没有关系。因为如果只是返回类型不同就构成重载的话，那么在你只调用函数却不接收它的返回值的情况下，编译器就不知道你需要调用的是哪个返回类型的函数。

``` cpp
#include <iostream>
using namespace std;

class A{
public:
    int Print(){
        return 1;
    }
    
    float Print(){
        return 2.33f;
    }
};

int main() {
    A a;
    a.Print();
    //error: 'float A::Print()' cannot be overloaded with 'int A::Print()'
    //如果构成重载，则此时编译器不知道该调用返回1的Print函数还是调用返回2.33的Print函数
    return 0;
}
```

## 方法隐藏

如果子类出现和父类同名的方法时会触发方法隐藏，无论参数和返回值类型是否相同，也无论是否为虚函数。在子类中父类的方法以及重载方法全部会被隐藏，也就是子类对象是无法调用该方法的重载方法的，因为找不到该重载方法。

``` cpp
#include <iostream>
#include <string>
using namespace std;

class A {
public:
    void Print()
    {
        cout << "A Print" << endl;
    }
    void Print(int a)
    {
        cout << "A Print int" << endl;
    }
};

class B : public A {
public:
    string Print(string b)
    {
        cout << "B Print" << endl;
        return b;
    }
};

int main()
{
    B b;
    b.Print("bb");  //输出：B Print
    b.Print(10);    //error: no matching function for call to ‘B::Print(int)’

    return 0;
}
```

可以通过两种方式解决:

``` cpp
int main(){
    B b;
    b.A::Print(10);
}
```

或者

``` cpp
class B : public A {
public:
    using A::Print;

    string Print(string b)
    {
        cout << "B Print" << endl;
        return b;
    }
};
```

---

## 默认参数值

只能在函数原型里写 `default argument`，定义里不能再重复一遍。

---

## 内联函数

- 内联函数在声明和定义都要写上 `inline`，也可以直接把函数体写在声明中
- 内联函数就是把函数体部分直接替换到调用它的地方，运行时节省调用函数所带来的额外开销，但会使源代码大小增加，绝大多数情况下是值得的，用廉价的空间换取时间
- 很小的函数或者循环调用的函数，适合写成内联函数；函数内容很大或者含有递归，则不适合写成内联函数。但 `inline` 关键词只算是给编译器一种建议，真正是否视为内联函数进行编译，是由编译器决定的。例如编写的内联函数内容很大或者含有递归，会被编译器拒绝，因为空间的增长会非常迅速

---

## 向上造型

将派生引用或指针转换为基类引用或指针的行为。

向上造型的转换，只是换个看法去看待，本身数据未改变。而基础类型的隐式转换则是数据有可能会遭到改变，改变不可逆。

---

## 虚函数

`virtual` 告诉编译器，通过指针或引用调用这个函数的时候，不能相信是什么类型，得等到运行时才能确定。这个指针所指的那个对象是什么类型，你再调用那个类型的函数。

- 继承中一个函数是 `virtual`，那么所有的子类下的这个函数都是 `virtual`
- 如果父类和子类的两个函数是 `virtual` 的，**函数名相同**，**参数表相同**，构成 `override`（可以在函数后加 `override` 关键字让编译器帮忙检查是否正确重写）。
  - `this` 不同，构不成重写：

    ```cpp
    #include <iostream>
    using namespace std;

    class Base {
    public:
        virtual void Print() const {
            cout << "Base" << endl;
        }
    };

    class Derivied : public Base {
    public:
        virtual void Print() {
            cout << "Derivied" << endl;
        }
    };

    void Test(Base &base) {
        base.Print();
    }

    int main() {
        Derivied d;
        Test(d); // Base
    }
    ```

- 如果重写了父类中有重载函数的函数，则必须在子类中重写所有变体，不能只重写其中一个，因为在子类中父类函数以及变体都将会被隐藏
- 类有一个 `virtual` 函数，析构函数就必须是 `virtual`（最好总是将析构函数声明为虚函数）
  - 析构不声明为`virtual`：

    ``` cpp
    #include <iostream>
    using namespace std;

    class A{
    public:
        A(){
            cout<<"A constructor"<<endl;
        }
        virtual void Test(){
            cout<<"A Test"<<endl;
        }
        ~A(){
            cout<<"A destructor"<<endl;
        }
    };

    class B :public A{
    public:
        B(){
            cout<<"B constructor"<<endl;
        }
        void Test() override {
            cout<<"B Test"<<endl;
        }
        ~B(){
            cout<<"B destructor"<<endl;
        }
    };

    int main(){
        A* a = new B();
        a->Test();
        delete a;
        //输出
        //A constructor
        //B constructor
        //B Test
        //A destructor
    }
    ```

  - 析构声明为`virtual`：

    ``` cpp
    #include <iostream>
    using namespace std;

    class A{
    public:
        A(){
            cout<<"A constructor"<<endl;
        }
        virtual void Test(){
            cout<<"A Test"<<endl;
        }
        virtual ~A(){
            cout<<"A destructor"<<endl;
        }
    };

    class B : public A{
    public:
        B(){
            cout<<"B constructor"<<endl;
        }
        void Test() override {
            cout<<"B Test"<<endl;
        }
        ~B(){
            cout<<"B destructor"<<endl;
        }
    };

    int main(){
        A* a = new B();
        a->Test();
        delete a;
        //输出
        //A constructor
        //B constructor
        //B Test
        //B destructor
        //A destructor
    }
    ```

    创建子类时，编译器会自动调用父类的构造函数，而析构子类时，同样在调用子类析构函数之后也会自动调用父类的析构函数。所以关键在于调用了谁的析构函数，如果是调用子类的析构函数，那么子类、父类的析构函数会被前后调用到；如果是调用父类的析构函数，那么就只调用父类的析构函数。如上， `A` 析构函数不声明为 `virtual` 的话， `delete a` 就会根据指针 `a` 所指的类型 `A` 来调用 `A` 的析构函数；反之，`delete a`时会根据所指的实际对象类型 `B` 来调用 `B` 的析构函数。

---

## 虚函数表

![vtable](vtable.png)

{% raw %}<article class="message is-info"><div class="message-body">{% endraw %}
dtor是destructor缩写
{% raw %}</div></article>{% endraw %}

<a class="tag is-dark is-medium" style="margin-bottom: 1rem" href="https://study.163.com/course/courseLearn.htm?courseId=271005#/learn/video?lessonId=394167&courseId=271005" target="_blank">
<span class="icon"><i class="fas fa-camera"></i></span>&nbsp;&nbsp;
来源自 面向对象程序设计 C++ ——多态的实现 翁恺
</a>

- 定义了虚函数的类对象的内存会大一些，因为需要多存储一个指向虚函数表 `vtable` 的指针 `vptr`
- 执行哪个虚函数取决与虚函数表里存储的函数地址。常见的向上造型，并没有改变 `vptr` 的指向，所以执行的是子类的虚函数，而强转的话只是把内存对象按强转类型进行解析，也不改变内容，所以仍然执行的是强转前该类型的虚函数

---

## 抽象类

把共有特征提取出来、至少有一个纯虚函数的类为抽象类。

- 抽象类不能被实例化
- 纯虚函数是通过在声明中使用 "`= 0`" 来指定的

---

## 接口

C++ 接口是使用抽象类来实现的，差别在于：

- 抽象类更多是为了继承而使用，可以有静态成员变量和方法的实现
- 接口更多是为了规范和约束，只将公开的方法声明出来

---

## 匿名函数（lambda 表达式）

```text
[ captures ] ( params ) specs requires(optional) { body }
```

- `captures`：捕获外部变量
  - `[]`：不捕获任何外部变量
  - `[=]`：按值捕获所有外部变量
  - `[&]`：按引用捕获所有外部变量
  - `[i, &j]`：`i` 按值捕获，`j` 按引用捕获
  - `[=, &i]`：除了 `i` 按引用捕获，其他按值捕获
  - `[&, i]`：除了 `i` 按值捕获，其他按引用捕获
  - `[this]`：按引用捕获当前对象
  - `[*this]`：按值捕获当前对象
- `params`：参数列表
- `specs`：由说明符、异常、属性和后置返回类型依次组成，这些都是可选的
- `requires`：约束
- `body`：函数体

``` cpp
[](int param)   //不捕获任何外部变量，函数只有一个int类型参数，省略了specs和requires
{
    cout << param << endl;
};
```

其实是函数对象，重载了函数调用符 `()` 。

---

## 强制转换

编译器可以帮忙进行检查这四种新型转换。

### static_cast

通常使用 `static_cast` 转换数值数据类型，例如将枚举型转换为整型或将整型转换为浮点型，而且同时明确表达需要转换类型的意图。

`static_cast` 可用于将指向基类的指针转换为指向派生类的指针等操作，但此类转换并非始终安全。 `static_cast` 转换安全性不如 `dynamic_cast` 转换，因为 `static_cast` 不执行运行时类型检查。

### const_cast

`const_cast` 最常见的用途是移除 `const` 属性，使得可以修改原本被声明为 `const` 的变量。

但是移除 `const` 属性后修改**常量**对象会导致未定义行为，因此只有在确定最终修改到的对象不是 `const` 时才应进行此操作。

```cpp
// const_cast < typename > (expression)

int i = 10;

const int *a = &i;
// *a = 11; // error: assignment of read-only location ‘* a’
int *pa = const_cast<int *>(a);
*pa = 12;   // ok: i = 12

const int j = 20;

const int *b = &j; 
int *pb = const_cast<int *>(b);
*pb = 22;   // 未定义行为， j 可能仍为 20 ，因为编译器可能对 const 对象进行优化，假设其值不会改变
```

#### 使用场景

有这样一个指针或引用，它大多时候需要是 `const`，但有时需要它是不带 `const` 的类型（比如第三方库的函数入参没要求是 `const` ）。

### dynamic_cast

运行时类型检查，能知道动态对象的类型，转换失败会返回 `nullptr`。可以用来向下造型。

### reinterpret_cast

重新解释转换，就是不管三七二十一进行转换。

---

## 左右值

### 左值

能取地址有名字的对象，表达式后仍存在的持久对象

### 右值

取不了地址的匿名对象，表达式结束后就不存在的临时对象，又分纯右值和将亡值。

- 纯右值：要么是纯粹的字面量，例如 `10`, `true`； 要么是求值结果相当于字面量或匿名临时对象，例如 `1+2`
- 将亡值：即将被销毁、却能够被移动的值（可能原本是左值）

``` cpp
std::vector<int> foo() {
    std::vector<int> temp = {1, 2, 3, 4};
    return temp;
}

std::vector<int> v = foo();
```

`v` 是左值； `foo()` 返回的值（是一个临时对象，即 `temp` 的副本）是右值（纯右值）； `temp` 在 `foo()` 内本身是左值，但返回时由于马上要被销毁了但其实可以通过移动利用起来，所以此时 `temp` 为将亡值。

希望把将亡值利用起来，所以就有了**移动语义**。

---

## 移动语义

是为了在一些情况下避免拷贝的开销，直接转移所有权更有效率。

### std::move

`std::move(x)` 将左值 `x` 转换成右值，转换后不应再对 `x` 进行操作。

### 右值引用

`T func(T&& x){}`，参数 `x` 是一个右值引用，也就是函数 `func` 需要传入一个右值。需要注意的是 `x` 本身是个左值，因为它是个变量，取得到地址，可以在等式的左边。

### 移动构造函数/移动赋值运算符

```cpp
class A {
    int* pointer;

public:
    // 移动构造函数
    A(A&& a) : pointer(a.pointer) {
        a.pointer = nullptr;        //将a.pointer设置为nullptr，这样a释放进行析构时就不会回收到现在被转移控制权的这块内存区域
    }

    // 移动赋值运算符
    A& operator=(A&& a) {
        if (this != &a) {
            delete pointer;         // 释放当前对象的资源
            pointer = a.pointer;    // 移动资源
            a.pointer = nullptr;    // 将原对象的资源设置为安全的默认状态
        }
        return *this;               // 返回 *this 以支持链式赋值
    }
};
```

### std::move能提高运行效率？

`std::move()` 只是将变量转换成右值引用而已，本身并不能提高效率，而是在类有实现移动构造函数或重载了移动赋值运算符，也就是提供移动语义的情况下才提高了效率。 C++ 语言内建的数据类型和 STL 标准模板库的容器已经实现了移动语义。

---

## 模板

模板是泛型编程的基础，泛型编程即以一种独立于任何特定类型的方式编写代码。

`template <typename T>`
是模板函数接下来就正常写函数，是模板类接下来就正常写类，把其中的类型用 `T` 代替就行了。

一般模板的声明和实现都写在头文件中。

函数模板的底层实现还是函数重载，在编译代码时，编译器会帮我们生成函数的重载实现。

---

## 列表初始化（List-initialization）

不要跟 [initializer list](https://zh.cppreference.com/w/cpp/language/constructor) 混淆了。

列表初始化是 C++11 引入的一种新的初始化方式，可以用于初始化变量、数组、容器等。它使用花括号 `{}` 进行初始化，提供了更统一和安全的初始化方式，避免了窄化转换（narrowing conversion）。

### 直接列表初始化

```cpp
int x{10};
MyClass obj{10, 20, ref_var};
std::vector<int> vec{1, 2, 3, 4};
```

### 拷贝列表初始化

```cpp
int x = {10};
MyClass obj = {10, 20, ref_var};
std::vector<int> vec = {1, 2, 3, 4};
```

在编译器未优化的情况下，这种形式可能会：

1. 调用一次构造函数来创建临时对象。
2. 调用一次拷贝构造函数来复制临时对象。
3. 最后销毁临时对象。

### 禁止窄化转换

```cpp
int x{3.14}; // 错误，不能从 double 窄化转换为 int
```

---

## 智能指针

利用栈元素离开作用域自动回收内存的特点。

### RAII

用类把资源及相关操作封装起来，在构造函数中获取资源，在析构函数中释放资源。使用时只要声明该类的对象，进行操作，在出了作用域后，就会自动调用它的析构函数对资源进行释放，就不需要显式地去释放。

```cpp
template<typename T>
class SimpleUniquePtr {
private:
    T* _ptr;

public:
    explicit SimpleUniquePtr(T* ptr = nullptr) : _ptr(ptr) {}

    ~SimpleUniquePtr() {
        delete _ptr;
    }

    SimpleUniquePtr(const SimpleUniquePtr&) = delete;
    SimpleUniquePtr& operator=(const SimpleUniquePtr&) = delete;

    SimpleUniquePtr(SimpleUniquePtr&& other) : _ptr(other.release()) {}

    SimpleUniquePtr& operator=(SimpleUniquePtr&& other) {
        reset(other.release());
        return *this;
    }

    T& operator*() const { 
        return *_ptr;
    }
    T* operator->() const {
        return _ptr;
    }

    T* get() const {
        return _ptr;
    }

    T* release() {
        T* oldPtr = _ptr;
        _ptr = nullptr;
        return oldPtr;
    }

    void reset(T* ptr = nullptr) {
        delete _ptr;
        _ptr = ptr;
    }
};
```

#### 直接声明资源对象不就行了？

类似上述 [new对象 vs 直接声明对象](#new对象-vs-直接声明对象)提到的情况。

- 资源对象很大，放到栈上不适合，使用 RAII ，栈上就只存有管理资源的对象，该对象持有资源的引用，相对的占用空间就小
- 作用域这点就不一样了，因为 RAII 就是要利用这点，让资源对象的生命周期等于管理资源的对象的生命周期
- 有些资源对象需要动态申请，没办法直接声明对象

### unique_ptr

一种独占的智能指针，它禁止其他智能指针与其共享同一个对象，从而保证代码的安全。既然是独占，换句话说就是不可复制。但是，我们可以利用 `std::move` 将其转移给其他的 `unique_ptr`；

- `make_unique()`：能够用来消除显式地使用 `new`，C++11没有提供，C++14才加入标准库的
- `get()`：可以通过该方法来获取原始指针
- `release()`：返回当前 `unique_ptr` 所指的堆内存，让普通指针/智能指针来接管，但该存储空间并不会被销毁。单独使用则会导致丢了控制权的同时，原来的内存得不到释放
- `reset(p)`：该函数会释放当前 `unique_ptr` 管理的对象，然后获取 `p` 所指对象的所有权。这点挺容易理解，要接受新的，就要把旧的释放掉，不然就内存泄露了

### shared_ptr

共享对象所有权的智能指针。多个 `shared_ptr` 对象可占有同一对象。

- `make_shared()`：能够用来消除显式地使用 `new`
- `get()`：可以通过该方法来获取原始指针
- `reset()`：当函数没有实参时，该函数会使当前 `shared_ptr` 所指堆内存的引用计数减 1，同时将当前 `shared_ptr` 重置为一个空指针；当为函数传递一个新申请的堆内存时，则调用该函数的 `shared_ptr` 对象会共享该存储空间的所有权，并且引用计数的初始值为 1。要跟 `unique_ptr` 的 `reset()` 进行区分，`shared_ptr` 的 `reset()` 只是减少引用，并不释放内存，因为其他 `shared_ptr` 还要用呢
- `use_count()`：查看所指对象的引用计数

#### 使用场景

- 有多个使用者共同使用同一个对象，而没有一个明显的拥有者
- 一个对象的复制操作很昂贵
- 要把指针存入标准库容器/要传递对象给软件库或从软件库获取对象，而这些对象没有明确的所有权

### weak_ptr

是一种可以共享但不拥有对象（不控制所指向对象生命周期）的智能指针。可以作为 `shared_ptr` 的辅助指针，因为 `share_ptr` 用的是引用计数，循环引用会有问题，所以引入了 `weak_ptr`。当 `weak_ptr` 的指向和某一 `shared_ptr` 相同时，`weak_ptr` 并不会使所指堆内存的引用计数加 1；同样，当 `weak_ptr` 被释放时，之前所指堆内存的引用计数也不会因此而减 1。因为 `weak_ptr` 不计入引用计数，同时另外记录弱引用，所以遇到循环引用问题时，只要把其中合适的一个 `shared_ptr` 改为 `weak_ptr` 就行了，会从这个 `weak_ptr` 指向的对象开始回收。

- `use_count()`：查看指向和当前 `weak_ptr` 相同的 `shared_ptr` 的数量
- `expired()`：判断当前 `weak_ptr` 为否过期（指针为空，或者指向的堆内存已经被释放）
- `lock()`：如果当前 `weak_ptr` 已经过期，则该函数会返回一个空的 `shared_ptr` ；反之，该函数返回一个和当前 `weak_ptr` 指向相同的 `shared_ptr`
- `reset()`：将当前 `weak_ptr` 置为空指针

---

## STL标准模板库

### 迭代器

迭代器就是一种对象，它提供了一种统一的接口，使得我们能够方便地对容器内的元素进行遍历。通过迭代器，我们可以在不需要了解容器内部实现细节的情况下，以一种统一的方式来访问容器中的元素。这种抽象的设计使得我们可以编写通用的代码，同时提高了代码的灵活性和可重用性。常见的实现方式就是指针，所以跟指针的操作方式很相似。

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： Khellendros
原文标题： 迭代器（iterator）和指针（pointer）区别在哪？
链接： <https://www.zhihu.com/question/54047747/answer/137783330>
来源： 知乎
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

迭代器实际上是对“遍历容器”这一操作进行了封装。在编程中我们往往会用到各种各样的容器，但由于这些容器的底层实现各不相同，所以对他们进行遍历的方法也是不同的。例如，数组使用指针算数就可以遍历，但链表就要在不同节点直接进行跳转。这是非常不利于代码重用的。例如你有一个简单的查找容器中最小值的函数findMin，如果没有迭代器，那么你就必须定义适用于数组版本的findMin和适用于链表版本的findMin，如果以后有更多容器需要使用findMin，那就只好继续添加重载……而如果每个容器又需要更多的函数例如findMax，sort，那简直就是重载地狱……我们的救星就是迭代器啦！如果我们将这些遍历容器的操作都封装成迭代器，那么诸如findMin一类的算法就都可以针对迭代器编程而不是针对具体容器编程，工作量一下子就少了很多！至于指针，由于指针也可以用来遍历容器(数组)，所以指针也可是算是迭代器的一种。但是指针还有其他功能，并不只局限于遍历数组。因为使用指针变量数组的操作太深入人心，c++stl中的迭代器就是刻意仿照指针来设计接口的。

### vector

- `size()`：返回容器中当前存储的元素数量
- `resize(n)`：改变容器中的元素数量为 `n` 。如果 `n` 小于当前的 `size()`，则删除超出的元素；如果 `n` 大于当前的 `size()`，则在末尾添加默认构造的元素直到数量达到 `n`
- `capacity()`：返回容器当前已为之分配空间的元素数（即可以存储的元素数量）
- `reserve(n)`：用于预分配内存，增加 `vector` 的容量到等大于 `n` 的值。若 `n` 大于当前的 `capacity()`，则分配新存储，否则该方法不做任何事。正确使用该方法能避免不必要的分配

### 容器底层实现

- `array`、 `vector`：都是申请了一块连续的内存空间。`vector` 长度可变是因为进行了3个步骤，第一步新申请一块内存空间，第二步把数据都复制到新内存空间中，第三步把旧的内存空间释放
- `deque` ：由一段一段的连续空间组成的，先申请一块连续的内存空间，里面存储了指向一块连续空间的指针
- `list` ：双向链表
- `forward_list` ：单向链表
- `map`、 `multimap`、 `set`、 `multiset` ：红黑树，默认按键值的大小进行升序排序
- `unordered_map`、 `unordered_set` ：哈希表，会先申请一整块连续的存储空间，跟 `deque` 有点像，不过是存储各个链表的头指针，各键值对真正的存储位置是各个链表的节点

### 自定义排序

返回值为 `true`，`ab`不交换；返回值为`false`，`ab`交换。可以理解为返回结果表示前者按着规则是否应该在后者前面。

``` cpp
#include <array>
#include <algorithm>
#include <iostream>
 
int main()
{
    std::array<int, 10> s = {5, 7, 4, 2, 8, 6, 1, 9, 0, 3}; 
 
    // 用 lambda 表达式排序
    std::sort(s.begin(), s.end(), [](int a, int b) {
        return a < b;   
    });
    for (auto a : s) {
        std::cout << a << " ";
    } 
    std::cout << '\n';  //输出：0 1 2 3 4 5 6 7 8 9
}
```

## 参考

- [面向对象程序设计 C++ - 翁恺](https://study.163.com/course/introduction/271005.htm)
- [现代 C++ 教程 - 欧长坤](https://changkun.de/modern-cpp/)
- [C++ 参考手册](https://zh.cppreference.com/w/cpp)
