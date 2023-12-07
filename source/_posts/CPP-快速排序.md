---
title: C++ 快速排序
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 0
date: 2021-11-26 16:29:39
categories: CPP
tags: 排序
---
``` cpp
#include <iostream>
#include <vector>
using namespace std;

void QuickSort(vector<int>& vec, int left, int right);
int Partition(vector<int>& vec, int left, int right);
void Print(const vector<int>& vec);

int main()
{
    vector<int> vec = { 49, 38, 65, 97, 76, 13, 27, 49 };
    cout << "before sort:" << endl;
    Print(vec);
    cout << "sort:" << endl;
    QuickSort(vec, 0, vec.size() - 1);
    cout << "after sort:" << endl;
    Print(vec);
    return 0;
}

void QuickSort(vector<int>& vec, int left, int right) //注意&
{
    if( left >= right ) return;

    int mid = Partition(vec, left, right);

    QuickSort(vec, left, mid - 1);  //在这可以判断mid-1小等于left，说明左侧只有0或者1个数字，不需要再进行排序，节省函数调用开销
    QuickSort(vec, mid + 1, right); //同上
}

int Partition(vector<int>& vec, int left, int right)
{
    int pivot = vec[left];

    while( left < right )
    {
        while( left < right && vec[right] >= pivot ) right--;   //注意是大等于，因为比基准值小的才需要换位
        if( left < right ) vec[left] = vec[right];              //当left等于right退出循环时，避免自己赋值给自己
        while( left < right && vec[left] <= pivot ) left++;
        if( left < right ) vec[right] = vec[left];
    }

    vec[left] = pivot;  //将基准值放入找到的位置

    Print(vec);

    return left;
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