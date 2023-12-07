---
title: 微信小游戏 跳转小程序接口 wx.navigateToMiniProgram
comments: true
toc: false
excerpt: ' '
date: 2018-09-27 23:36:16
categories: 微信小游戏
tags:
sticky: 0
recommend: 0
---
小程序的`navigateToMiniProgram`要弃用了，但小游戏的`navigateToMiniProgram`应该是没事，因为用别的H5引擎的话，用不了它的`<navigator>`组件，所以小游戏`navigateToMiniProgram`接口说明中没提到弃用的问题。

|      |        |     |                                    |
| ---- | ------ | --- | ---------------------------------- |
| path | string | 否  | 打开的页面路径，如果为空则打开首页 |

如果不是要打开首页，而是要直接跳转到小程序的某个页面，那么就要填，详情见[相关](#相关)。

|           |        |     |                                                                                      |
| --------- | ------ | --- | ------------------------------------------------------------------------------------ |
| extraData | object | 否  | 需要传递给目标小程序的数据，目标小程序可在 App.onLaunch，App.onShow 中获取到这份数据 |

在`App.onShow`中要拿到这个`extraData`的话，要`onShow`的`res.referrerInfo.extraData`，比如：

``` javascript
wx.navigateToMiniProgram({
    appId:'******',
    extraData:{
        name:'张三',
        age:18,
    }
})
```

``` javascript
onShow(res){
    console.log(res.referrerInfo.extraData.name)    //输出张三
    console.log(res.referrerInfo.extraData.age)    //输出18
}
```

|                        |        |                                                |
| ---------------------- | ------ | ---------------------------------------------- |
| referrerInfo.extraData | Object | 来源小程序传过来的数据，scene=1037或1038时支持 |

**注意场景为1037或1038才支持**。

|      |                    |                  |
| ---- | ------------------ | ---------------- |
| 1037 | 小程序打开小程序   | 来源小程序 appId |
| 1038 | 从另一个小程序返回 | 来源小程序 appId |

不支持的话可以用另一种方式，就是在路径后跟上要传递的数据。

``` javascript
wx.navigateToMiniProgram({
    appId:'******',
    path:'**/***/****?name=张三&age=18',
})
```

然后在`onShow`中用`query`去获取。

``` javascript
onShow(res){
    console.log(res.query.name);    //输出张三
    console.log(res.query.age);    //输出18
}
```

# 相关

## 文件结构

小程序包含一个描述整体程序的 `app` 和多个描述各自页面的 `page`。

一个小程序主体部分由三个文件组成，**必须放在项目的根目录**，如下：

| 文件     | 必需 | 作用             |
| -------- | ---- | ---------------- |
| app.js   | 是   | 小程序逻辑       |
| app.json | 是   | 小程序公共配置   |
| app.wxss | 否   | 小程序公共样式表 |

一个小程序页面由四个文件组成，分别是：

| 文件类型 | 必需 | 作用       |
| -------- | ---- | ---------- |
| js       | 是   | 页面逻辑   |
| wxml     | 是   | 页面结构   |
| json     | 否   | 页面配置   |
| wxss     | 否   | 页面样式表 |

**注意：为了方便开发者减少配置项，描述页面的四个文件必须具有相同的路径与文件名**。

## 全局配置

`app.json`文件用来对微信小程序进行全局配置，决定页面文件的路径、窗口表现、设置网络超时时间、设置多 tab 等。

`app.json`配置项列表`page`属性

| 属性  | 类型   | 必填  | 描述 | 支持版本     |
| ----- | ------ | ----- | ---- | ------------ |
| pages | String | Array | 是   | 页面路径列表 |

`pages`用于指定小程序由哪些页面组成，每一项都对应一个页面的 路径+文件名 信息。文件名不需要写文件后缀，框架会自动去寻找对于位置的 `.json`, `.js`, `.wxml`, `.wxss` 四个文件进行处理。

数组的第一项代表小程序的初始页面（首页）。小程序中新增/减少页面，都需要对`pages`数组进行修改。

如开发目录为：

```
├── app.js
├── app.json
├── app.wxss
├── pages
│   │── index
│   │   ├── index.wxml
│   │   ├── index.js
│   │   ├── index.json
│   │   └── index.wxss
│   └── logs
│       ├── logs.wxml
│       └── logs.js
└── utils
```

则需要在`app.json`中写

```
{
  "pages":[
    "pages/index/index",
    "pages/logs/logs"
  ]
}
```