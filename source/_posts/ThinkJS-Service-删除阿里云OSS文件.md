---
title: ThinkJS Service 删除阿里云OSS文件
comments: true
toc: false
excerpt: ' '
date: 2019-02-19 16:01:10
categories: ThinkJS
tags: 
- ThinkJS
- Node.js
- 阿里云
sticky: 0
recommend: 0
---
``` javascript
'use strict';
 
/**
 * 阿里云OSS
 * Created on 2018-12-21
 */
const OSS = require('ali-oss');
 
const region = 'oss-cn-shenzhen';
// ACCESS_KEY_ID/ACCESS_KEY_SECRET 根据实际申请的账号信息进行替换
const accessKeyId = '****************';
const accessKeySecret = '******************************';
const bucket = '*****************';
 
module.exports = class extends think.Service {
    /**
     * @description 删除OSS文件
     * @param {string} fileName 文件名
     * @return {boolean} 文件不存在也返回true，请求错误返回false
     */
    async deleteFile(fileName) {
        try {
            let client = new OSS({
                region,
                accessKeyId,
                accessKeySecret,
                bucket
            });
            let result = await client.delete(fileName);
            let ret = result.res.status == 204;
            return ret;
        } catch (e) {
            think.logger.info(e);
        }
    }
}
```

# 参考

- [阿里云 对象存储 OSS > SDK 示例 > Node.js > 管理文件 > 删除文件](https://help.aliyun.com/document_detail/111408.html?spm=a2c4g.11186623.6.1113.124e7a74rj1OKP)