---
title: C++ 希尔排序
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 0
date: 2021-12-05 22:47:53
categories: CPP
tags: 排序
---
希尔排序是插入排序的改进版本。比起插入排序多了个步骤，先对元素进行分组，再进行插入排序。

``` cpp
#include <iostream>
#include <vector>
using namespace std;

void ShellSort(vector<int>& vec);
void Print(const vector<int>& vec);

int main()
{
    vector<int> vec = { 49, 38, 65, 97, 76, 13, 27, 49 };
    cout << "before sort:" << endl;
    Print(vec);
    cout << "sort:" << endl;
    ShellSort(vec);
    cout << "after sort:" << endl;
    Print(vec);
}

void ShellSort(vector<int>& vec)
{
    int length = vec.size();
    int preIndex;
    int currentValue;

    for( int gap = length / 2; gap > 0; gap /= 2 )  //分组
    {
        for( int i = gap; i < length; i++ )         //就是插入排序，只不过插入排序的gap一直为1，1️⃣
        {
            preIndex = i - gap;
            currentValue = vec[i];
            while( preIndex >= 0 && currentValue < vec[preIndex] )
            {
                vec[preIndex + gap] = vec[preIndex];
                preIndex -= gap;
            }
            vec[preIndex + gap] = currentValue;
        }

        Print(vec);
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

# 1️⃣ for( int i = gap; i < length; i++ )

`i` 从第一组的第二个元素开始，对应的索引就是 `0+gap`， `i++`后对应就是第二组的第二个元素，以此类推，第三组的第二个元素......而且要注意代码实现中，不是把分配到一组内的元素都插入排序后再轮到下一组，而是第一组的第二个元素进行完， `i++`，轮到第二组的第二个元素，每一组的第二个元素都进行完后，轮到第一组的第三个元素进行插入排序。是按元素原本的顺序交叉进行的。

![](shell_sort-0.png)