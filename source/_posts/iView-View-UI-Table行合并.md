---
title: iView(View UI) Table行合并
comments: true
toc: true
excerpt: '设置属性 `span-method` 可以指定合并行或列的算法。'
date: 2020-11-26 12:04:15
categories: Vue
tags:
sticky: 0
recommend: 0
---
# 文档

[表格Table 行/列合并](https://www.iviewui.com/components/table#H/LHB)

<span class="tag is-danger">4.0.0</span> 设置属性 `span-method` 可以指定合并行或列的算法。

该方法参数为 4 个对象：

- `row`: 当前行
- `column`: 当前列
- `rowIndex`: 当前行索引
- `columnIndex`: 当前列索引

该函数可以返回一个包含两个元素的数组，第一个元素代表 `rowspan`，第二个元素代表 `colspan`。 也可以返回一个键名为 `rowspan` 和 `colspan` 的对象。

---

但官方文档并没有说明`rowspan`和`colspan`是怎么回事，通过对官方的示例观察以及验证，得出以下结论：

- `rowspan` = 0 隐藏该行
- `rowspan` = 1 显示该行
- `rowspan` > 1 涉及合并的行数或者说合并`rowspan` - 1的行数，需要注意的一点，它是从该行往下合并。比如`rowspan`为3，则从这一行开始算，接下来三行会进行合并。

`colspan`没验证过，但规则应该是一致的，把行换成列，往下合并换成往右合并。

# "通用"的行合并实现

- 基于文档 [表格Table 行/列合并](https://www.iviewui.com/components/table#H/LHB) 的示例代码修改
- 只适用于数据格式为“常见”的对象数组
    ``` json
[
  {
    "name": "Jim",
    "age": 18,
    "address": "Sydney",
    "date": "2016-10-03"
  },
  {
    "name": "Jim",
    "age": 24,
    "address": "London",
    "date": "2016-10-01"
  }
]
```
- 想要忽略某一列的行合并，把 columns 的 key 对应 push 入 ignoreMergeRow 数组即可
- 其他请查看代码注释

## 效果

![](effect.png)

### 忽略name列的行合并

![](ignore_merge_row.png)

# 代码

{% raw %}<article class="message is-danger"><div class="message-body">{% endraw %}
代码仅供参考，如有更优雅的实现方式，请不吝赐教
{% raw %}</div></article>{% endraw %}

``` html
<template>
<div>
    <Table :columns="columns14" :data="data5" border :span-method="handleSpan"></Table>
</div>
</template>
 
<script>
export default {
    data() {
        return {
            columns14: [{
                    title: 'Date',
                    key: 'date'
                },
                {
                    title: 'Name',
                    key: 'name'
                },
                {
                    title: 'Age',
                    key: 'age'
                },
                {
                    title: 'Address',
                    key: 'address'
                }
            ],
            data5: [],
            ignoreMergeRow:[]
        }
    },
    methods: {
        handleMergeRow(data) {
            let ret = JSON.parse(JSON.stringify(data));
            let collection = {};
            let preIndex = -1;
            for (let i in ret) {
                // item = {
                //     "name": "Jim",
                //     "age": 18,
                //     "address": "Sydney",
                //     "date": "2016-10-03"
                // }
                let item = ret[i];
                item.source = {};
                for (let key in item) {
                    if (key == "source") {
                        continue;
                    }
                    // 记录每个key
                    // collection = {
                    //     "name": {}
                    // }
                    if (!collection[key]) {
                        collection[key] = {};
                    }
                    // 首项或者该项此列不等于上一项此列
                    if (preIndex == -1 || ret[preIndex][key] != item[key]) {
                        // 记录每个key对应的所有值以及其起始索引和出现次数
                        // collection = {
                        //     "name": {
                        //         "Jim#0": {
                        //             "firstIndex": 0,
                        //             "count": 1
                        //         }
                        //     }
                        // }
                        collection[key][`${item[key]}#${i}`] = {
                            firstIndex: i,
                            count: 1
                        };
                        // 标注该列的起源index
                        item.source[key] = i;
                    } else {
                        // 通过上一项找到起源index
                        let source = ret[preIndex].source[key];
                        item.source[key] = source;
                        // 合并该列的行数加1
                        collection[key][`${item[key]}#${source}`].count += 1;
                    }
                }
                preIndex = i;
            }
            console.log(collection)
            for (let key in collection) {
                // items = {
                //     "Jim#0": {
                //         "firstIndex": "0",
                //         "count": 3
                //     },
                //     "Jon#3": {
                //         "firstIndex": "3",
                //         "count": 1
                //     }
                // }
                let items = collection[key];
                for (let itemkey in items) {
                    // value = {
                    //     "firstIndex": "0",
                    //     "count": 3
                    // }
                    let value = items[itemkey];
                    // 根据记录的起始索引设置该项的mergeRow
                    // {
                    //     "name": "Jim",
                    //     "age": 18,
                    //     "address": "Sydney",
                    //     "date": "2016-10-03",
                    //     "mergeRow": {
                    //         "address": 1,
                    //         "age": 1,
                    //         "date": 1,
                    //         "name": 3
                    //     }
                    // }
                    if (!ret[value.firstIndex].mergeRow) {
                        ret[value.firstIndex].mergeRow = {};
                    }
                    ret[value.firstIndex].mergeRow[key] = value.count;
                }
            }
            console.log(ret)
            return ret;
        },
        handleSpan({
            row,
            column,
            rowIndex,
            columnIndex
        }) {
            if (row.mergeRow) {
                // 忽略合并该行此列
                if (this.ignoreMergeRow && this.ignoreMergeRow.indexOf(column.key) >= 0) {
                    return {
                        rowspan: 1,
                        colspan: 1
                    };
                }
                // 存在此列的key，则该行此列应显示，向下合并此列的几行（包括自己）则取决于其值
                if (row.mergeRow[column.key]) {
                    return {
                        rowspan: row.mergeRow[column.key],
                        colspan: 1
                    };
                } else {
                    // 说明该行此列应被在此之上的行合并，rowspan和colspan设置为0，不予显示
                    return {
                        rowspan: 0,
                        colspan: 0
                    };
                }
            }
        },
    },
    mounted() {
        let data = [{
                name: 'Jim',
                age: 18,
                address: 'Sydney',
                date: '2016-10-03'
            },
            {
                name: 'Jim',
                age: 24,
                address: 'London',
                date: '2016-10-01'
            },
            {
                name: 'Jim',
                age: 30,
                address: 'Sydney',
                date: '2016-10-01'
            },
            {
                name: 'Jon',
                age: 26,
                address: 'Sydney',
                date: '2016-10-04'
            }
        ];
        this.data5 = this.handleMergeRow(data);
    }
}
</script>
 
<style>
 
</style>
```

![](running_result-0.png)

![](running_result-1.png)