---
title: C# 方法隐藏(new)和重写(override)方法
comments: true
toc: true
excerpt: '方法隐藏相当于在子类中定义新方法，而方法重写则是重新定义父类中方法的内容。'
date: 2020-01-03 17:07:14
categories: CSharp
tags:
sticky: 0
recommend: 0
---
``` csharp
using System;
 
namespace Test
{
    class A
    {
        public virtual void Print()
        {
            Console.WriteLine("A");
        }
    }
    class B : A
    {
        public new void Print()
        {
            Console.WriteLine("B");
        }
    }
    class C : A
    {
        public override void Print()
        {
            Console.WriteLine("C");
        }
    }
 
    class Program
    {
        static void Main(string[] args)
        {
            A a1 = new B();
            a1.Print();    // 输出"A"
            A a2 = new C();
            a2.Print();    // 输出"C"
            A a = new A();
            a.Print();    //输出"A"
        }
    }
}
```
{% raw %}<article class="message is-danger"><div class="message-body">{% endraw %}
以下的理解主要是记录方便自己日后回忆，不太懂术语，可能会表达错误或者不严谨。
{% raw %}</div></article>{% endraw %}

**方法隐藏相当于在子类中定义新方法，而方法重写则是重新定义父类中方法的内容**。

没有去了解C#底层如何实现子类实例转换成父类实例，不过经过测试，子类实例转换成父类实例时，我的理解就是，把父类实例里的属性方法遍历一下，如果子类实例中也有对应的，以子类实例的为准，把子类实例的拿来"替换"掉，所以有重写的话，它们都是指向同一个方法，就会替换掉，而方法隐藏相当于在子类中定义新方法，因此不算，也就不会进行"替换"。而且强调实例是因为这个所谓的"替换"，并不会真正影响到父类，只在生成实例时影响，因为最后`a.print()`，依旧输出了"`A`"。