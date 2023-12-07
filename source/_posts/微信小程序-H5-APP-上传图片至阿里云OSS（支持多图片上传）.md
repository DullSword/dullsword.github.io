---
title: 微信小程序/H5/APP 上传图片至阿里云OSS（支持多图片上传）
comments: true
toc: true
excerpt: '该教程是针对微信小程序平台的文件上传功能， 但是如果是使用uniapp开发的话，在 微信小程序/H5/APP 平台上都能通用使用的，需要把 `wx` 修改为 `uni` 即可使用。'
date: 2019-02-19 15:38:38
categories: 
- [微信小程序]
- [HTML5]
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： 涛々
原文标题： 微信小程序/H5/APP 上传图片至阿里云OSS（支持多图片上传）
链接： https://blog.csdn.net/qq_23375733/article/details/81417296
来源： CSDN博客
转载篇幅： 全文
是否修改： 是
{% raw %}</div></article>{% endraw %}

该教程是针对微信小程序平台的文件上传功能， 但是如果是使用uniapp开发的话，在 微信小程序/H5/APP 平台上都能通用使用的，需要把 `wx` 修改为 `uni` 即可使用

**我们先讲下为什么要把图片文件上传到云服务器呢， 有什么好处呢？**

1、能减轻我们自己服务器的带宽

如果一个程序里有多处地方用到用户上传图片等功能的话，建议还是放到阿里云或者千牛云等其他平台上来存储我们的图片，可以给公司的服务器减少很多压力，磁盘存储也就不会太大。

2、提升用户体验感

我们开发的产品一般都是以用户体验感为主对吧？

当用户在使用我们的小程序上传图片时，如果一次上传了多张，我们的服务器接口肯定是要进行对这些上传文件进行处理，如果同一时间访问量大的话，我们的程序先处理这些请求，在把图片存放到某个文件里，再返回给客服端结果，处理的相对是比较慢的，用户等待的时间也就比较长，体验感不好。

如果换成上传图片到云服务器上，我们就可以直接把这个外网可访问的图片地址在前端拿到，然后后端接收在存入数据库，最后再返回结果就好，处理的时间就大大的减少了。

接下来切入主题，先看下目录结构，也就是我们小程序需要用到的文件如下：

![](directory.png)

`config.js`：配置文件，代码如下：

``` javascript
var fileHost = "https://xxx/";//你的阿里云OSS地址  在你当前小程序的公众号后台的uploadFile 合法域名也要配上这个域名
var config = {
   uploadImageUrl: `${fileHost}`, // 默认存在根目录，可根据需求改
   AccessKeySecret: 'xxx',        // AccessKeySecret 去你的阿里云上控制台上找
   OSSAccessKeyId: 'xxx',         // AccessKeyId 去你的阿里云上控制台上找
   timeout: 80000 //这个是上传文件时Policy的失效时间
};
module.exports = config
```

`fileHost`：把 `xxx` 替换成你的阿里云oss地址

`AccessKeyId`和`AccessKeySecret`怎么拿到？

可以百度搜下或者看下阿里云的帮助文档 地址：https://helpcdn.aliyun.com/knowledge_detail/48699.html

![](access_key.png)

`uploadFile.js`：主要的业务逻辑实现过程，代码如下：

``` javascript
const env = require('config.js'); //配置文件，在这文件里配置你的OSS keyId和KeySecret,timeout:87600;
 
const base64 = require('base64.js');//Base64,hmac,sha1,crypto相关算法
require('hmac.js');
require('sha1.js');
const Crypto = require('crypto.js');
 
/*
 *上传文件到阿里云oss
 *@param - filePath :图片的本地资源路径
 *@param - dir:表示要传到哪个目录下
 *@param - successc:成功回调
 *@param - failc:失败回调
 */
const uploadFile = function (filePath, dir, successc, failc) {
   if (!filePath || filePath.length < 9) {
      wx.showModal({
         title: '图片错误',
         content: '请重试',
         showCancel: false,
      })
      return;
   }
 
   console.log('上传图片.....');
   //图片名字 可以自行定义，     这里是采用当前的时间戳 + 150内的随机数来给图片命名的
   const aliyunFileKey = dir + new Date().getTime() + Math.floor(Math.random() * 150) + '.png';
 
   const aliyunServerURL = env.uploadImageUrl;//OSS地址，需要https
   const accessid = env.OSSAccessKeyId;
   const policyBase64 = getPolicyBase64();
   const signature = getSignature(policyBase64);//获取签名
 
   wx.uploadFile({
      url: aliyunServerURL,//开发者服务器 url
      filePath: filePath,//要上传文件资源的路径
      name: 'file',//必须填file
      formData: {
         'key': aliyunFileKey,
         'policy': policyBase64,
         'OSSAccessKeyId': accessid,
         'signature': signature,
         'success_action_status': '200',
         // 'x-oss-security-token': '如果有发现上传不成功的问题，可能是你的阿里云oss上的accesskey的权限不是允许所有应用访问的，可以把这个参数也一起上传试一下',
      },
      success: function (res) {
         if (res.statusCode != 200) {
            failc(new Error('上传错误:' + JSON.stringify(res)))
            return;
         }
         successc(aliyunServerURL + aliyunFileKey);
      },
      fail: function (err) {
         err.wxaddinfo = aliyunServerURL;
         failc(err);
      },
   })
}
 
const getPolicyBase64 = function () {
   let date = new Date();
   date.setHours(date.getHours() + env.timeout);
   let srcT = date.toISOString();
   const policyText = {
      "expiration": srcT, //设置该Policy的失效时间，超过这个失效时间之后，就没有办法通过这个policy上传文件了 
      "conditions": [
         ["content-length-range", 0, 5 * 1024 * 1024] // 设置上传文件的大小限制,5mb
      ]
   };
 
   const policyBase64 = base64.encode(JSON.stringify(policyText));
   return policyBase64;
}
 
const getSignature = function (policyBase64) {
   const accesskey = env.AccessKeySecret;
   const bytes = Crypto.HMAC(Crypto.SHA1, policyBase64, accesskey, {
      asBytes: true
   });
   const signature = Crypto.util.bytesToBase64(bytes);
   return signature;
}
 
module.exports = uploadFile;
```

注释都写在文件里头了，你们慢慢看哈！

'`x-oss-security-token`': '如果有发现上传不成功的问题，可能是你的阿里云oss上的`accesskey`的权限不是允许所有应用访问的，可以把这个参数也一起上传试一下哈，希望可以帮助到你们'

主要说下图片的命名，我是采用时间戳 + 随机数的方式和后缀为.png格式的图片文件；

![](name.png)

你们可以替换成你自己想要的格式； 可以把上面的这句话 从 `new Date()` 开始往后进行修改

其他的文件都是一些算法文件，我都打包了，最后再给你们源文件的地址链接 0.0

**如何使用？**

在你需要用的页面上，比如我们`index.wxml`里有个按钮如下：

``` html
<button type='primary' bindtap='choose'>选择照片</button>
```

在`index.js`文件里引入：

``` javascript
//获取应用实例
const app = getApp()
var uploadImage = require('../../utils/uploadFile.js');//地址换成你自己存放文件的位置
var util = require('../../utils/util.js');
 
Page({
   data: {
 
   },
 
   //选择照片
   choose: function () {
      wx.chooseImage({
         count: 9, // 默认最多一次选择9张图
         sizeType: ['original', 'compressed'], // 可以指定是原图还是压缩图，默认二者都有
         sourceType: ['album', 'camera'], // 可以指定来源是相册还是相机，默认二者都有
         success: function (res) {
            // 返回选定照片的本地文件路径列表，tempFilePath可以作为img标签的src属性显示图片
            var tempFilePaths = res.tempFilePaths;
            var nowTime = util.formatTime(new Date());
 
            //支持多图上传
            for (var i = 0; i < res.tempFilePaths.length; i++) {
               //显示消息提示框
               wx.showLoading({
                  title: '上传中' + (i + 1) + '/' + res.tempFilePaths.length,
                  mask: true
               })
 
               //上传图片
               //你的域名下的/cbb文件下的/当前年月日文件下的/图片.png
               //图片路径可自行修改
               uploadImage(res.tempFilePaths[i], 'images/' + nowTime + '/',
                  function (result) {
                     console.log("======上传成功图片地址为：", result);
                     //做你具体的业务逻辑操作
 
                     wx.hideLoading();
                  }, function (result) {
                     console.log("======上传失败======", result);
                     //做你具体的业务逻辑操作
 
                     wx.hideLoading()
                  }
               )
            }
         }
      })
   }
})
```

{% raw %}<article class="message is-info"><div class="message-body">{% endraw %}
需要注意的是在阿里云OSS 存储空间（Bucket）中：

- **同一个存储空间的内部是扁平的，没有文件系统的目录等概念，所有的对象都直接隶属于其对应的存储空间**。

所以上述例子中的"`你的域名下的/images文件下的/当前年月日文件下的/图片.png`"，既是图片的地址也是它对应的文件名，不要误以为它的文件名是"`图片.png`"。文件名会在SDK管理文件的API中用到，比如删除OSS上的某个文件：{% post_link ThinkJS-Service-删除阿里云OSS文件 %}
{% raw %}</div></article>{% endraw %}

以上是可以支持多图上传的，传一张也是可以的；上传图片后具体存放在哪的位置可自行定义！

![](console.png)

好了，上传成功拿到图片地址，就可以为所欲为的做你想做的事了~~

ps：如果检查了配置都对但一直报403错误，可以试试把插件里面 `crypto.js` 中这句话注释掉试下

``` javascript
if (typeof btoa == "function") return btoa(util.bytesToString(bytes));
```

最后贴下我这项目的GitHub地址，所有的文件都在这：[hujinchen/Image_upload_aliyun](https://github.com/hujinchen/Image_upload_aliyun)