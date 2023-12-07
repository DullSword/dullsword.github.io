---
title: C++ 单链表简单实现
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 0
date: 2021-12-11 20:02:23
categories: CPP
tags: 
---
{% raw %}<article class="message is-info"><div class="message-body">{% endraw %}
不带头节点的简单实现，主要是插入、删除、反转几种类型的操作，具体看头文件就知道了。

可以使用模板、实现迭代器进行改进。
{% raw %}</div></article>{% endraw %}

``` cpp linkedlist.h
#pragma once

class ListNode {
public:
    int data;
    ListNode* next;

    ListNode(int data = 0, ListNode* next = nullptr);
};

class LinkedList {
    ListNode* head;
    ListNode* tail;
    int length;

public:
    LinkedList();
    virtual ~LinkedList();
    void PushBack(int val);
    void PopBack();
    void PushFront(int val);
    void PopFront();
    void Insert(int val, int pos);
    void RemoveByValue(int val);
    void RemoveByPosition(int pos);
    int FindByValue(int val);
    int FindByPosition(int pos);
    void Reverse();
    void clean();
    bool Empty();
    int Size();
    void Print();
};

```
``` cpp linkedlist.cpp
#include <iostream>
#include <exception>
//#include "vld.h"  //visual leak detector
#include "linkedlist.h"

ListNode::ListNode(int data, ListNode* next) :data(data), next(next)
{

}

LinkedList::LinkedList() : head(nullptr), tail(nullptr), length(0)
{

}

LinkedList::~LinkedList()
{
    ListNode* node = head;
    ListNode* temp = nullptr;
    while( node != nullptr )
    {
        temp = node;
        node = node->next;
        delete temp;
    }
}

//尾插
void LinkedList::PushBack(int val)
{
    ListNode* node = new ListNode(val);

    if( head == nullptr )
    {
        head = node;
    }
    else
    {
        tail->next = node;
    }
    tail = node;
    length++;
}

//尾删
void LinkedList::PopBack()
{
    if( head == nullptr ) return;

    if( head == tail )
    {
        head = nullptr;
        delete tail;
        tail = nullptr;
        length--;
        return;
    }

    ListNode* node = head;
    while( node->next != tail )
    {
        node = node->next;
    }
    delete tail;
    tail = nullptr;
    node->next = nullptr;
    tail = node;
    length--;
}

//头插
void LinkedList::PushFront(int val)
{
    ListNode* node = new ListNode(val);

    if( head == nullptr )
    {
        head = node;
        tail = node;
        length++;
        return;
    }

    node->next = head;
    head = node;
    length++;
}

//头删
void LinkedList::PopFront()
{
    if( head == nullptr ) return;

    if( head == tail )
    {
        delete head;
        head = nullptr;
        tail = nullptr;
        length--;
        return;
    }

    ListNode* temp = head;
    head = head->next;
    delete temp;
    temp = nullptr;
    length--;
}

//按位置插入
void LinkedList::Insert(int val, int pos)
{
    if( pos < 0 || pos > length )
    {
        throw std::exception("illegal position");
    }

    if( pos == 0 )
    {
        PushFront(val);
    }
    else if( pos == length )
    {
        PushBack(val);
    }
    else
    {
        int _pos = 0;
        ListNode* node = head;
        ListNode* pre_node = nullptr;
        while( _pos < length && _pos != pos )
        {
            pre_node = node;
            node = node->next;
            _pos++;
        }

        ListNode* insert_node = new ListNode(val);
        insert_node->next = node;
        pre_node->next = insert_node;

        length++;
    }
}

//按值删除
void LinkedList::RemoveByValue(int val)
{
    ListNode* node = head;
    ListNode* pre_node = nullptr;

    while( node != nullptr )
    {
        if( node->data == val )
        {
            if( node == head )
            {
                node = node->next;
                PopFront();
            }
            else if( node == tail )
            {
                node = nullptr;
                PopBack();
            }
            else
            {
                pre_node->next = node->next;
                delete node;
                node = nullptr;
                node = pre_node->next;
                length--;
            }
        }
        else
        {
            pre_node = node;
            node = node->next;
        }
    }
}

//按位置删除
void LinkedList::RemoveByPosition(int pos)
{
    if( pos < 0 || pos >= length )
    {
        throw std::exception("illegal position");
    }

    if( pos == 0 )
    {
        PopFront();
    }
    else if( pos == length - 1 )
    {
        PopBack();
    }
    else
    {
        int _pos = 0;
        ListNode* node = head;
        ListNode* pre_node = nullptr;
        while( _pos < length && _pos != pos )
        {
            pre_node = node;
            node = node->next;
            _pos++;
        }

        pre_node->next = node->next;
        delete node;

        length--;
    }
}

//按值查找
int LinkedList::FindByValue(int val)
{
    int pos = 0;

    if( head == nullptr )
    {
        return -1;
    }

    ListNode* node = head;
    while( node != nullptr && node->data != val )
    {
        node = node->next;
        pos++;
    }
    int ret = ( pos == length ? -1 : pos );
    return ret;
}

//按位置查找
int LinkedList::FindByPosition(int pos)
{
    if( pos < 0 || pos >= length )
    {
        throw std::exception("illegal position");
    }

    int _pos = 0;
    ListNode* node = head;
    while( _pos != pos )
    {
        node = node->next;
        _pos++;
    }

    return node->data;
}

//反转链表
void LinkedList::Reverse()
{
    if( head == nullptr || head == tail ) return;

    ListNode* node = head;
    ListNode* pre_node = nullptr;
    ListNode* _node = nullptr;
    ListNode* _pre_node = nullptr;

    tail = head;

    while( node != nullptr )
    {
        _node = node;
        _pre_node = pre_node;

        pre_node = node;
        node = node->next;

        _node->next = _pre_node;	//其实pre_node->next = _pre_node就行，完全不需要_node，有_node会更便于理解
    }

    head = _node;
}

//清空
void LinkedList::clean()
{
    ListNode* node = head;
    ListNode* temp = nullptr;
    while( node != nullptr )
    {
        temp = node;
        node = node->next;
        delete temp;
    }
}

//判空
bool LinkedList::Empty()
{
    return length == 0;
}

//长度
int LinkedList::Size()
{
    std::cout << length << std::endl;
    return length;
}

//遍历打印节点值
void LinkedList::Print()
{
    ListNode* node = head;
    while( node != nullptr )
    {
        std::cout << node->data << " ";
        node = node->next;
    }
    std::cout << std::endl;
}

//测试
int main()
{
    LinkedList list;

    /*list.PopBack();
    list.PushBack(5);
    list.PushBack(10);
    list.Print();
    list.PopBack();
    list.Print();
    list.PopBack();
    list.Print();
    list.PopBack();
    list.Size();*/

    /*list.PopFront();
    list.PushFront(5);
    list.PushFront(10);
    list.Print();
    list.PopFront();
    list.Print();
    list.PopFront();
    list.Print();
    list.PopFront();
    list.Size();*/

    /*list.PushBack(5);
    list.PushBack(10);
    list.PushBack(15);
    list.Print();
    try
    {
        list.Insert(888, -1);
    }
    catch( std::exception& e )
    {
        std::cout << e.what() << std::endl;
    }
    try
    {
        list.Insert(888, 4);
    }
    catch( std::exception& e )
    {
        std::cout << e.what() << std::endl;
    }
    list.Insert(1, 0);
    list.Print();
    list.Insert(9, 4);
    list.Print();
    list.Insert(7, 3);
    list.Print();
    list.Size();*/

    /*list.PushBack(5);
    list.PushBack(10);
    list.PushBack(15);
    list.Print();
    list.RemoveByValue(10);
    list.Print();
    list.RemoveByValue(5);
    list.Print();
    list.RemoveByValue(15);
    list.Print();
    list.Size();*/

    /*list.PushBack(5);
    list.PushBack(10);
    list.PushBack(15);
    list.Print();
    try
    {
        list.RemoveByPosition(-1);
    }
    catch( std::exception& e )
    {
        std::cout << e.what() << std::endl;
    }
    try
    {
        list.RemoveByPosition(3);
    }
    catch( std::exception& e )
    {
        std::cout << e.what() << std::endl;
    }
    list.RemoveByPosition(1);
    list.Print();
    list.RemoveByPosition(1);
    list.Print();
    list.RemoveByPosition(0);
    list.Print();
    list.Size();*/

    /*list.PushBack(5);
    list.PushBack(10);
    list.PushBack(15);
    std::cout << list.FindByValue(8) << std::endl;
    std::cout << list.FindByValue(5) << std::endl;
    std::cout << list.FindByValue(15) << std::endl;
    std::cout << list.FindByValue(10) << std::endl;*/

    /*list.PushBack(5);
    list.PushBack(10);
    list.PushBack(15);
    try
    {
        std::cout << list.FindByPosition(-1) << std::endl;
    }
    catch( std::exception& e )
    {
        std::cout << e.what() << std::endl;
    }
    try
    {
        std::cout << list.FindByPosition(3) << std::endl;
    }
    catch( std::exception& e )
    {
        std::cout << e.what() << std::endl;
    }
    std::cout << list.FindByPosition(2) << std::endl;
    std::cout << list.FindByPosition(0) << std::endl;
    std::cout << list.FindByPosition(1) << std::endl;*/

    /*list.PushBack(5);
    list.PushBack(10);
    list.PushBack(15);
    list.PushBack(20);
    list.Print();
    list.Reverse();
    list.Print();*/

    return 0;
}
```