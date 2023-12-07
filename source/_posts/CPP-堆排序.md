---
title: C++ 堆排序
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 0
date: 2021-12-06 22:17:14
categories: CPP
tags: 排序
---
堆排序（英语：Heapsort）是指利用堆这种数据结构所设计的一种排序算法。堆是一个近似完全二叉树的结构，并同时满足堆的性质：即子节点的键值或索引总是小于（或者大于）它的父节点。

{% raw %}<article class="message is-info"><div class="message-body">{% endraw %}
以下代码实现的是大根堆
{% raw %}</div></article>{% endraw %}

``` cpp
#include <iostream>
#include <vector>
using namespace std;

void HeapSort(vector<int>& vec);
void AdjustHeap(vector<int>& vec, int start, int end);
void Print(const vector<int>& vec);

int main()
{
    vector<int> vec = { 49, 38, 65, 97, 76, 13, 27, 49 };
    cout << "before sort:" << endl;
    Print(vec);
    cout << "sort:" << endl;
    HeapSort(vec);
    cout << "after sort:" << endl;
    Print(vec);
}

void HeapSort(vector<int>& vec)
{
    int length = vec.size();

    //构建初始堆
    for( int i = length / 2 - 1; i >= 0; i-- )  //从最后一个非叶子节点开始到根节点，从下往上构建堆，1️⃣
    {
        AdjustHeap(vec, i, length - 1); //第三个参数表示堆最后元素的索引，构建初始堆时，堆规模不需要变，所以都用length-1就行
    }

    for( int i = length - 1; i > 0; i-- )   //i表示每次待确认的元素索引，从最后一个元素到第二个元素
    {
        swap(vec[0], vec[i]);               //堆顶元素和待确认元素进行交换
        AdjustHeap(vec, 0, i - 1);          //堆规模变成从第一个元素到待确认元素的前一个元素
    }
}

void AdjustHeap(vector<int>& vec, int start, int end)   //end主要就是为了判断子节点是否存在
{
    int current = start;
    int left = 2 * current + 1;     //2️⃣
    int right = 2 * current + 2;

    int largestChild = left;        //先默认子节点之间左节点最大

    while( left <= end )            //子树存在
    {
        if( right <= end && vec[right] > vec[left] )    //right<=end，右节点存在
        {
            largestChild = right;
        }

        if( vec[largestChild] > vec[current] )
        {
            swap(vec[largestChild], vec[current]);

            current = largestChild;                     //以子节点为下一个节点
            left = 2 * current + 1;
            right = 2 * current + 2;
            largestChild = left;
        }
        else
        {
            break; //从堆顶上往下调整堆时，如果根节点大于子节点，则不需要调整，也就不影响接下去的节点，而这些节点已经是排好了的，因此可以退出循环
        }
    }
}

void Print(const vector<int>& vec)
{
    for( const int v : vec )
    {
        cout << v << " ";
    }
    cout << endl;
}
```

![](running_result-0.png)

# 1️⃣ for( int i = length / 2 - 1; i >= 0; i-- )

为什么最后一个非叶子节点的索引是 `length/2-1`，这里得先知道父节点的索引为 $\color{blue}{(i-1)/2}$，这个结论下面有进行推导。最后一个元素的父节点就是最后一个非叶子节点，代入公式，所以最后一个非叶子节点的索引是 $\color{blue}{(lenth-1-1)/2 \Rightarrow (length-2)/2 \Rightarrow length/2-1}$。

# 2️⃣ int left = 2 * current + 1; int right = 2 * current + 2;

![](heap.png)

索引为 `i` 的元素位于第 `n` 层，第 `n` 层位于该元素前面的元素个数为 `x` 个。假设索引为 `i` 的元素是第`y`个，`y` 就等于第1层到第 `n-1` 层所有的节点数+`x`+1，即 $\color{blue}{y=(2^{0}+2^{1}+\cdots+2^{n-1})+x+1}$，通过等比数列求和公式 $\color{blue}{S_{n}=\cfrac{a_{1} *\left(q^{n}-1\right)}{q-1}}$ 可以得到 $\color{blue}{y=(2^{n-1}-1)+x+1}$。因为`i`是`y`的索引，$\color{blue}{i=y-1}$，所以$\color{blue}{i=2^{n-1}-1+x}$;

`j` 指向 `i` 的左节点，可以看出因为完全二叉树的性质，第 `n+1` 层位于 `j` 索引元素前面的元素个数为 `2x` 个，可以推出位于第 `n+1` 层的 `j` 元素，$\color{blue}{j=2^{(n+1)-1}-1+2x}$，想知道两者的关系，所以往 `i` 的表达式方向去凑，提取个2，$\color{blue}{j=2 \times (2^{n-1}-1+x)+1}$，也就是 `2i+1`，`k` 自然就是 `2i+2`，所以左节点索引为 `2i+1`，右节点索引为 `2i+2`;

同理求父节点 `p`，$\color{blue}{p=2^{(n-1)-1}-1+x/2}$，提取个1/2，$\color{blue}{p=(2^{n-1}-1+x-1)/2 \Rightarrow (i-1)/2}$，所以父节点的索引为 $\color{blue}{(i-1)/2}$;

