---
title: C++ 插入排序
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 0
date: 2021-11-29 17:17:18
categories: CPP
tags: 排序
---
# 直接插入排序

类似于扑克牌抓牌，假设习惯是牌从左往右升序，那么抓到新的牌后，从右到左找牌，找到小等于抓到的牌的那张牌，则新抓的牌就应该插入到那张牌的后面。

``` cpp
#include <iostream>
#include <vector>
using namespace std;

void InsertionSort(vector<int>& vec);
void Print(const vector<int>& vec);

int main()
{
    vector<int> vec = { 49, 38, 65, 97, 76, 13, 27, 49 };
    cout << "before sort:" << endl;
    Print(vec);
    cout << "sort:" << endl;
    InsertionSort(vec);
    cout << "after sort:" << endl;
    Print(vec);
}

void InsertionSort(vector<int>& vec)
{
    int length = vec.size();
    int preIndex;
    int currentValue;

    for( int i = 1; i < length; i++ )   //i从1开始，因为第一个数默认有序，i <= vec.size()-1 || i < vec.size()
    {
        preIndex = i - 1;       //前一个数的索引
        currentValue = vec[i];  //当前要插入的数的值
        while( preIndex >= 0 && currentValue < vec[preIndex] )  //preIndex >= 0，因为如果第一个数都比currentValue大，preIndex--会变为-1
        {
            vec[preIndex + 1] = vec[preIndex];  //别写成vec[i]=vec[preIndex]，i是不变的，会变成vec[i]被赋成不同值，而不是把比currentValue大的值往后移
            preIndex--;
        }
        vec[preIndex + 1] = currentValue;   //preIndex+1，因为preIndex是前一个数的索引，当不符合条件退出while循环时，说明currentValue应该插入到preIndex所指的数的后面

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

# 折半（二分）插入排序

跟直接插入排序的区别其实只有一处，就是在给当前数找位置的时候，因为已知前面的数字都已经有序，所以可以使用二分查找减少比较次数，速度比直接插入排序快。

因为找到插入位置后，需要将后续对应的数全部往后移动，移动次数不变，所以时间复杂度依旧为O(n^2)。

``` cpp
#include <iostream>
#include <vector>
using namespace std;

void BinaryInsertionSort(vector<int>& vec);
void Print(const vector<int>& vec);

int main()
{
    vector<int> vec = { 49, 38, 65, 97, 76, 13, 27, 49 };
    cout << "before sort:" << endl;
    Print(vec);
    cout << "sort:" << endl;
    BinaryInsertionSort(vec);
    cout << "after sort:" << endl;
    Print(vec);
}

void BinaryInsertionSort(vector<int>& vec)
{
    int length = vec.size();
    int left;
    int right;
    int mid;
    int currentValue;
    for( int i = 1; i < length; i++ )
    {
        left = 0;
        right = i - 1;
        currentValue = vec[i];
        while( left <= right )
        {
            mid = ( left + right ) / 2;

            if( currentValue < vec[mid] )
            {
                right = mid - 1;
            }
            else
            {
                //等于的情况下，继续在后面二分查找，最终插入的值会排在后面，具有稳定性
                left = mid + 1;
            }
        }

        for( int j = i - 1; j >= left; j-- )    //把插入位置到待插入元素的当前位置的元素全部往后移动一位
        {
            vec[j + 1] = vec[j];
        }

        vec[left] = currentValue;

        Print(vec);
    }
}

void Print(const vector<int>& vec)
{ for( int v : vec )  {
        cout << v << " ";
    }
    cout << endl;
}
```

![](running_result-1.png)