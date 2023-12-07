---
title: C++ 冒泡排序
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 0
date: 2021-11-26 22:32:23
categories: CPP
tags: 排序
---
``` cpp
#include <iostream>
#include <vector>
using namespace std;

void BubbleSort(vector<int>& vec);
void Print(const vector<int>& vec);

int main()
{
    vector<int> vec = { 49, 38, 65, 97, 76, 13, 27, 49 };
    cout << "before sort:" << endl;
    Print(vec);
    cout << "sort:" << endl;
    BubbleSort(vec);
    cout << "after sort:" << endl;
    Print(vec);
}

void BubbleSort(vector<int>& vec)
{
    int length = vec.size();
    bool flag = true;   //flag用来标记某一趟排序是否发生交换
    for( int i = 0; i < length - 1; i++ )
    {
        flag = false;   //flag置为false，如果本趟排序没有发生交换，则不会执行下一趟排序
        for( int j = 0; j < length - 1 - i; j++ )
        {
            if( vec[j] > vec[j + 1] )
            {
                int temp = vec[j];
                vec[j] = vec[j + 1];
                vec[j + 1] = temp;
                flag = true;    //flag置为true，表示本趟排序发生了交换
            }
        }
        Print(vec);
        if( flag == false ) break;
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

原本8个元素应该进行7趟排序，因为加入了flag，最后进行了6趟，第6趟发现没有进行任何元素的交换，说明已经有序，退出排序。