---
title: Unity 协程（Coroutine）
comments: true
toc: false
excerpt: ' '
date: 2020-11-02 15:41:19
categories: Unity
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： huang9012
原文标题： 简要分析unity3d中剪不断理还乱的yield
链接： https://blog.csdn.net/huang9012/article/details/29595747#comments_13359762
来源： CSDN博客
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

协程不是线程，也不是异步执行的。协程和 `MonoBehaviour` 的 `Update` 函数一样也是在 `MainThread` 中执行的。使用协程你不用考虑同步和锁的问题。

协程跟 `Update()` 其实一样的，都是Unity每帧对会去处理的函数（如果有的话）。如果 `MonoBehaviour` 是处于激活（active）状态的而且 `yield` 的条件满足，就会协程方法的后面代码。还可以发现：如果在一个对象的前期调用协程，协程会立即运行到第一个 `yield return` 语句处，如果是 `yield return null` ，就会在同一帧再次被唤醒。如果没有考虑这个细节就会出现一些奇怪的问题。

协程和 `Update()` 一样更新，可以使用 `Time.deltaTime` ，而且这个 `Time.deltaTime` 和在 `Update()` 当中使用是一样的效果（使用 `yield return null` 的情况下）

---

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： Gypsum Shen
原文标题： Unity奇淫巧技 —— Coroutine协程
链接： http://gypsumlabs.github.io/2015/12/08/unity-tips-coroutine/
来源： 石膏实验室
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

# 协程?

当然我早就听说了协程这个东西，可能你也听过。但是我和之前学C#学 `delegate` 时一样，一开始没有很好的理解这个东西，文章看了一些，还是Get不到它正确的应用场景，而一直不愿意去用。其实很多时候就是要去放开了试才能真的理解这些技巧的意义，其瓶颈不过就是一层纸，这次终于把这层纸捅破罢了。

协程（Coroutine）不同于线程（Thread），他目标解决的不是线程阻塞一类的问题。以我个人的理解，他主要解决有**某些必须在游戏循环中运行的代码，而你又不想把他写到 `Update()` 中**的情况。

游戏与一般应用程序的区别在于，一般应用程序是**事件驱动**的，而游戏则拥有**游戏循环**这一特性，在Unity中体现为 `MonoBehaviour` 的 `Update()` 方法。游戏中自然也会有"事件"，基本都可以写在 `Update()` 中，但是由于游戏循环的存在， `Update()` 会风雨无阻的在每一帧调用，为了让代码适时的执行，一般都会设置几个 `bool` 或 `int/float` 来控制 `Update()` 中的代码。

这时你会发现，如果你的 `Update()` 中有太多这样的代码时，代码的可读性会越来越糟。

# 解决问题

来看一个比较典型的情况：**一个事件会在游戏过程中被触发，触发后执行一些动作A，等待5秒后，执行一些动作B，事件结束**。

我原来会这么写：

``` csharp
public class SimpleEvent : MonoBehaviour
{
    private bool event_Start;
    private float event_Timer;
    private bool A_Done;
    private bool B_Done;

    //打开事件开关，同时重置该事件的计时器和A，B动作的标记
    public void StartEvent()
    {
        event_Start = true;
        event_Timer = 0;
        A_Done = false;
        B_Done = false;
    }

    void Update()
    {
        if(event_Start)
        {
            if(!A_Done)
            {
                DoSomething_A();
                A_Done = true;
            }

            if(event_Timer<5)
            {
                event_Timer += Time.deltaTime;
            }
            else
            {
                if(!B_Done)
                {
                    DoSomething_B();
                    B_Done = true;
                }

                //关掉事件开关，结束事件
                event_Start = false;
            }
        }
    }
}
```

需要触发事件时，我只要调用一下这个脚本的 `StartEvent()` 即可，看起来也没有什么问题。

可是如果一个 `Update()` 中有很多个这样的事件，又或是一个事件中有很多等待用的计时器，而每多一个计时器，你的事件逻辑就又多了一个 `if( )else( )` 嵌套。如此一来， `Update()` 会越来越难以阅读。

所以你知道的，协程该出现了：

``` csharp
public class SimpleEvent : MonoBehaviour
{
    IEnumerator Event()
    {
        DoSomething_A();

        yield return new WaitForSeconds(5);

        DoSomething_B();
    }
}
```

触发这个事件我只要：

``` csharp
StartCoroutine(Event());
```

 `IEnumerator` 是C#的一种迭代器，协程实现的具体原理可以找到很多文章分析。但是就算没有彻底理解其原理也没有关系，只要会应用就可以。总之，协程以 `IEnumerator` 协程名(形参){ } 申明，和一般的方法申明类似。

协程中必须要有 `yield return` 值 这样的返回值，例子中的 `yield return new WaitForSeconds(5)` 就表示等待5秒钟。

一句代码就能完成等待功能，且不需要计时器变量和 `if()else()` 判断！这已经很神奇了，当然协程不止于此，他还有另外一个重要的特性:

``` csharp
yield return null;
```

这个表示等待下一帧。可能一下子无法理解这个等待的意义，回头再看一下我对协程的理解：他可以解决有某些必须在游戏循环中运行的代码，而你又不想把他写到 `Update()` 中的情况。

所以说我认为协程应该可以替代 `Update()` ，让代码不写在 `Update()` 中又能跑在游戏循环中的能力。如何实现？请看：

``` csharp
IEnumerator CoroutineUpdate()
{
    while(true)
    {
        //TODO:你的游戏更新

        yield return null;
    }
}
```

 `while(true){ }` 是一个无限循环，在循环结尾处的 `yield return null` ;则意味着这个无限循环的代码，每执行一次则会等到下一帧再继续循环，这不就是一个 `Update()` 嘛！

他有一个相比 `Update()` 额外的好处，就是你可以在想结束循环时， `break` 即可。或者直接使用 `for` 进行有限次的循环，很灵活。

# 其他值得一提的事

启动一个协程的方法为：

- `StartCoroutine( 协程() , 参数 )` ;
- `StartCoroutine( "协程名" , 参数 )` ; //以字符串为参数

停止协程的方法：

- `StopAllCoroutine()` ; 停止所有协程，使用上面第一个重载运行的协程只能以这种方式停止。

- `StopCoroutine("协程名")` ; 停止单个协程只能用这个，且只能停止同样以字符串为参数启动的协程。

另外，协程中 `yield return` 可以使用其他协程做为返回值，比如 `yield return OtherCoroutine()` ; 作用就是等待 `OtherCoroutine()` 的代码全部运行完毕后继续运行，这个特性也是极其好用的，可以实现很多不限时等待的功能。

# 以上
　　
协程是一种更加灵活的 `Update()` 替代品，避免过多的游戏逻辑堆砌在一个 `Update()` 中导致最终的可读性下降。同时还提供了 `yield return new WaitForSeconds(float)` 这样方便的等待功能，不需要在外部申明一堆计时器变量然后在逻辑中使用 `if()else()` 判断时间这样繁琐的流程。

另外，协程也存在一些问题，就是启用多个协程之后，务必注意代码执行顺序的问题，这个问题在以前可以使用 `Update()` 和 `LateUpdate()` 缓解，但是在协程里就不太好解决，所以尽量避免多个协程之间的耦合，在一个协程中完成一个独立的功能是一个良好的思路。