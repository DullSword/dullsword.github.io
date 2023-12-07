---
title: C++ 选择排序
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 0
date: 2021-12-01 02:36:06
categories: CPP
tags: 排序
---
跟冒泡排序有点相似，冒泡排序是符合条件直接进行交换，而选择排序则是将索引记录下，一轮比较完成后再进行交换。

# 找最小元素

``` cpp
#include <iostream>
#include <vector>
using namespace std;

void SelectionSort(vector<int>& vec);
void Print(const vector<int>& vec);

int main()
{
    vector<int> vec = { 49, 38, 65, 97, 76, 13, 27, 49 };
    cout << "before sort:" << endl;
    Print(vec);
    cout << "sort:" << endl;
    SelectionSort(vec);
    cout << "after sort:" << endl;
    Print(vec);
}

void SelectionSort(vector<int>& vec)
{
    int length = vec.size();
    int minIndex;
    for( int i = 0; i < length - 1; i++ )   //每次确认一个最小数，只要length-1趟就可以完成排序
    {
        minIndex = i;   //将待排的第一个元素索引赋值给minIndex
        for( int j = i + 1; j < length; j++ )   //从待排的第二个元素开始比较，直到待排的最后一个元素
        {
            if( vec[j] < vec[minIndex] )
            {
                minIndex = j;    //将最小数的索引保存
            }
        }

        if( minIndex != i )
        {
            swap(vec[i], vec[minIndex]);
        }

        Print(vec);
    }
}

void Print(const vector<int>& vec)
{
    for( int v : vec )
    {
        cout << v << " ";
    }
    cout << endl;
}
```

![](running_result-0.png)

# 找最大元素

跟上者的过程相反，代码看起来会没上者的简洁。

``` cpp
#include <iostream>
#include <vector>
using namespace std;

void SelectionSort(vector<int>& vec);
void Print(const vector<int>& vec);

int main()
{
    vector<int> vec = { 49, 38, 65, 97, 76, 13, 27, 49 };
    cout << "before sort:" << endl;
    Print(vec);
    cout << "sort:" << endl;
    SelectionSort(vec);
    cout << "after sort:" << endl;
    Print(vec);
}

void SelectionSort(vector<int>& vec)
{
    int length = vec.size();
    int maxIndex;
    for( int i = 0; i < length - 1; i++ )   //每次确认一个最大数，只要length-1趟就可以完成排序
    {
        maxIndex = length - 1 - i;  //将待排的最后一个元素索引赋值给maxIndex
        for( int j = length - 1 - i - 1; j >= 0; j-- )  //从待排的倒二个元素开始比较，直到待排的第一个元素
        {
            if( vec[j] > vec[maxIndex] )
            {
                maxIndex = j;   //将最大数的索引保存
            }
        }

        if( maxIndex != length - 1 - i )
        {
            swap(vec[length - 1 - i], vec[maxIndex]);
        }

        Print(vec);
    }
}

void Print(const vector<int>& vec)
{
    for( int v : vec )
    {
        cout << v << " ";
    }
    cout << endl;
}
```

![](running_result-1.png)

打印出来的排序过程跟上者是不同的。
