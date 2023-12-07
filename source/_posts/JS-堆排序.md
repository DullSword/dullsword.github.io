---
title: JS 堆排序
comments: true
toc: true
excerpt: ' '
date: 2018-07-25 21:41:21
categories: Javascript
tags: 排序
sticky: 0
recommend: 0
---
{% raw %}<article class="message is-info"><div class="message-body">{% endraw %}
以下代码实现的是大根堆
{% raw %}</div></article>{% endraw %}

``` javascript
    let arr = [49, 38, 65, 97, 76, 13, 27, 49];
    console.log('arr:' + arr);    //打印排序前的数组
    HeapSort(arr);
    console.log('sortArr:' + arr);    //打印排序后的数组
 
    function HeapSort(arr){
        buildHeap(arr);    //建初堆
 
        for(let i=arr.length-1; i>0; i--){    //i=1时调整最后两个数即完成排序，故不需等于0
            swap(i);    //将堆顶元素与堆中最后一个元素交换，完成其中一个数的排序
            headAdjust(arr, 0, i-1);    //调整堆，从堆顶开始到i-1
        }
    }
 
    function buildHeap(arr){
        for(let i=Math.floor(arr.length/2)-1; i>=0; i--){    
            //i是索引值
            //从最后一个非终端节点开始到i=0节点
            headAdjust(arr, i, arr.length-1);    
        }
    }
 
    function swap(i){
        let swap = arr[i];
        arr[i] = arr[0];
        arr[0] = swap; 
    }
 
    
    function headAdjust(arr, i, n){    //i是起始节点索引，n是堆中最后一个元素索引
        let current= arr[i];    //保存当前节点值
        let child = 2*i+1;    //左子树索引
 
        while(child <= n){    //子树存在
            if(child  < n && arr[child] < arr[child + 1]){    //child<n说明不是最后一个元素
                child ++;    //右子树值比左子树值大就指向右子树
            }
            if(arr[i] < arr[child]){    //当前节点值小于子树值
                arr[i] = arr[child];    //把子树值赋给当前节点
                i= child;               //以子树为下一个节点
                child = 2*i + 1;        
            }
            else{        
                /*初建堆是从最后一个非终端节点开始的，所以从堆顶上往下调整堆时，如果根节点大于子        
                节点，则不需要调整，也就不影响接下去的节点，而这些节点已经是排好了的，因此可以退
                出循环*/
                break;
            }
            arr[i] = current;    //把当前节点值放置到对应的子节点上
        }
    }
```

![](running_result-0.png)