---
title: Github Actions 自动签到
comments: true
toc: true
excerpt: ' '
date: 2020-11-20 17:25:43
categories: 工具
tags: 
- Github Actions
- Autocheck
sticky: 2
recommend: 2
---

使用有风险，最好是在自己服务器上跑，可以是自己安装环境然后定时运行 `checkin.js`，也可以是用 [自托管运行器](https://docs.github.com/cn/actions/hosting-your-own-runners/about-self-hosted-runners) 。

## 主要代码

### checkin.yml

``` yml
name: CheckIn
on:
  # 可手动触发工作流程
  workflow_dispatch:
  # push时触发工作流程
  push:
    # 分支 现在默认是main，而不是master
    branches: [ main ]
    # 将工作流程配置为在至少一个文件不匹配paths-ignore的paths时运行
    # 忽略README.md和imgs文件夹
    paths-ignore: 
      - 'README.md'
      - 'imgs/**'
  # 定时触发工作流程
  schedule:
    # UTC 1点30分(北京时间 9点30分)
    - cron: 30 1 * * *
  # 标星时触发工作流程
  watch:
    types: [started]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # Checks out a copy of your repository on the ubuntu-latest machine
      # 必须，不然到执行 node checkin.js 时会报错
      - uses: actions/checkout@v2
      - name: setup Node.js
        # 理论上也可以自己命令行安装node
        uses: actions/setup-node@v1
        with:
          node-version: '14.x'
      - name: install axios
        run: npm install axios
        # 安装 puppeteer-extra-plugin-stealth 绕过检测
      - name: install puppeteer-extra-plugin-stealth
        run: npm install puppeteer puppeteer-extra puppeteer-extra-plugin-stealth
      - name: CheckIn
        run: node checkin.js
        # 设置环境变量
        env:
          PUSHPLUS: ${{ secrets.PUSHPLUS }}
          COOKIES: ${{ secrets.COOKIES }}
        # 保持 action 活跃
      - name: keep alive
        uses: gautamkrishnar/keepalive-workflow@master # using the workflow with default settings
        # 删除工作流记录
      - name: Delete workflow runs
        uses: Mattraks/delete-workflow-runs@v2
        with:
          token: ${{ github.token }}
          repository: ${{ github.repository }}
          retain_days: 0
          keep_minimum_runs: 10
          delete_run_by_conclusion_pattern: success
```

### checkin.js

稍微修改下就可以变成其他网站登录的脚本

``` javascript
const axios = require('axios');
const puppeteer = require('puppeteer-extra');
const StealthPlugin = require('puppeteer-extra-plugin-stealth');

puppeteer.use(StealthPlugin());

const INFO = {
    account: '账号',
    leftDays: '天数',
    checkInMessage: '签到情况',
    checkInFailded: '签到失败',
    getStatusFailed: '获取信息失败'
};

const checkCOOKIES = (COOKIES) => {
    const cookies = COOKIES?.split('&&') || [];

    if (!cookies.length) {
        console.error('不存在 COOKIES ，请重新检查');
        return false;
    }

    for (const cookie of cookies) {
        if (!cookie.includes('=')) {
            console.error(`存在不正确的 cookie ，请重新检查`);
            return false;
        }

        const pairs = cookie.split(/\s*;\s*/);
        for (const pairStr of pairs) {
            if (!pairStr.includes('=')) {
                console.error(`存在不正确的 cookie ，请重新检查`);
                return false;
            }
        }
    }

    return true;
}

const rawCookie2JSON = (cookie) => {
    return cookie.split(/\s*;\s*/).reduce((pre, current) => {
        const pair = current.split(/\s*=\s*/);
        const name = pair[0];
        const value = pair.splice(1).join('=');
        return [
            ...pre,
            {
                name,
                value,
                'domain': '******.rocks'
            }
        ];
    }, []);
};

const checkInAndGetStatus = async (cookie) => {
    const browser = await puppeteer.launch({ headless: true });
    const page = await browser.newPage();

    const cookieJSON = rawCookie2JSON(cookie);
    await page.setCookie(...cookieJSON);

    await page.goto('https://******.rocks/console/checkin', {
        timeout: 0,
        waitUntil: 'load'
    });

    page.on('console', msg => {
        if (console[msg.type()]) {
            console[msg.type()](msg.text());
        } else {
            console.log(msg.text());
        }
    });

    const info = await page.evaluate(async (INFO) => {
        const checkIn = () =>
            fetch('https://******.rocks/api/user/checkin', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json;charset=utf-8',
                },
                body: JSON.stringify({
                    token: "******.network"
                })
            }).catch(error => {
                console.warn('checkIn 网络错误。');
                return { reason: '网络错误' }
            });

        const getStatus = () => fetch('https://******.rocks/api/user/status').catch(error => {
            console.warn('getStatus 网络错误。');
            return { reason: '网络错误' }
        });

        let ret = {};

        const checkInRes = await checkIn();
        if (!checkInRes.ok) {
            const reason = checkInRes.reason || `状态码：${checkInRes.status}`;
            console.warn(`checkIn 请求失败，${reason}`);
            ret[INFO.checkInFailded] = reason;
        } else {
            console.info('checkIn 请求成功。');
            const { message } = await checkInRes.json();
            ret[INFO.checkInMessage] = message;
        }

        const statusRes = await getStatus();
        if (!statusRes.ok) {
            const reason = statusRes.reason || `状态码：${statusRes.status}`;
            console.warn(`getStatus 请求失败，${reason}`);
            ret[INFO.getStatusFailed] = reason;
        } else {
            console.info('getStatus 请求成功。');
            const { data: { email, phone, leftDays } = {} } = await statusRes.json();
            let account = '未知账号';
            if (email) {
                account = email.replace(/^(.)(.*)(.@.*)$/,
                    (_, a, b, c) => a + b.replace(/./g, '*') + c
                );
            } else if (phone) {
                account = phone.replace(/^(.)(.*)(.)$/,
                    (_, a, b, c) => a + b.replace(/./g, '*') + c
                );
            }
            ret[INFO.account] = account;
            ret[INFO.leftDays] = parseInt(leftDays);
        }

        return ret;
    }, INFO);

    await browser.close();

    return info;
};

const pushplus = (token, infos) => {
    const data = {
        token,
        title: '签到',
        content: JSON.stringify(infos),
        template: 'json'
    };
    console.log('pushData', {
        ...data,
        token: data.token.replace(/^(.{1,4})(.*)(.{4,})$/, (_, a, b, c) => a + b.replace(/./g, '*') + c)
    });

    return axios({
        method: 'post',
        url: `http://www.pushplus.plus/send`,
        data
    }).catch((error) => {
        if (error.response) {
            // 请求成功发出且服务器也响应了状态码，但状态代码超出了 2xx 的范围
            console.warn(`PUSHPLUS推送 请求失败，状态码：${error.response.status}`);
        } else if (error.request) {
            // 请求已经成功发起，但没有收到响应
            console.warn('PUSHPLUS推送 网络错误');
        } else {
            // 发送请求时出了点问题
            console.log('Axios Error', error.message);
        }
    });
};

const CheckIn = async () => {
    try {
        if (checkCOOKIES(process.env.COOKIES)) {
            const cookies = process.env.COOKIES.split('&&');

            const infos = await Promise.all(cookies.map(cookie => checkInAndGetStatus(cookie)));
            console.log('infos', infos);

            const PUSHPLUS = process.env.PUSHPLUS;

            if (!PUSHPLUS) {
                console.warn('不存在 PUSHPLUS ，请重新检查');
            }

            if (PUSHPLUS && infos.length) {
                const pushResult = (await pushplus(PUSHPLUS, infos))?.data?.msg;
                console.log('PUSHPLUS pushResult', pushResult);
            }
        }
    } catch (error) {
        console.log(error);
    }
};

CheckIn();
```

### Q&A

`checkin.js` 就是简单地使用 `axios` 发送请求，`checkin.yml` 的疑问点，简单的在上面已经做了注释，下面是我自己尝试过程中比较困惑过的：

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
分支名为什么是main，其他很多文章都是master？
{% raw %}</div></article>{% endraw %}

GitHub 官方表示，从 2020 年 10 月 1 日起，在该平台上创建的所有新的源代码仓库将默认被命名为 " `main` "，而不是原先的" `master` "，以避免联想奴隶制。

[Renaming the default branch from master](https://github.com/github/renaming)

![](renaming_the_default_branch_from_master.png)

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
标星为什么是watch而不是star？
{% raw %}</div></article>{% endraw %}

``` yml
# 标星时运行
  watch:
    types: [started]
```

官网文档 [WatchEvent](https://docs.github.com/cn/developers/webhooks-and-events/events/github-event-types#watchevent)

![](WatchEvent.png)

[标星 REST API](https://docs.github.com/cn/rest/reference/activity#starring)

![](star.png)

[关注者API更改帖子](https://developer.github.com/changes/2012-09-05-watcher-api/)

![](upcoming_changes_to_Watcher_and_Star_APIs.png)

我的理解是Github仓库中 `star` 对应之前的 `watch` ，而Github Action还是采用之前的叫法，所以在 `check.yml` 中用的关键词是 `watch` 。

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
\- uses: actions/checkout@v2是干么的？
{% raw %}</div></article>{% endraw %}

``` yml
# Checks out a copy of your repository on the ubuntu-latest machine
# 必须，不然到执行 node checkin.js 时会报错
  - uses: actions/checkout@v2
```

在 官方文档 - Github Actions 快速入门 - [创建第一个工作流程](https://docs.github.com/cn/actions/quickstart#creating-your-first-workflow) 中有这么一条注释：

![](creating_your_first_workflow.png)

`- uses: actions/checkout@v2` 语句的作用是在服务器上检出我们仓库的副本，没有这条语句的话，执行到 `node checkin.js` 时会报错，因为到这步时服务器上相当于只安装了 `node.js` 和 `axios` 请求库，没有我们的项目，所以运行 `node checkin.js` 时会找不到 `check.js` 。

![](build.png)

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
secret&环境变量？
{% raw %}</div></article>{% endraw %}

官方文档中有较为详细的解释 关于[加密密码](https://docs.github.com/cn/actions/security-guides/encrypted-secrets)

着重看下 [在工作流程中使用加密密码](https://docs.github.com/cn/actions/security-guides/encrypted-secrets#using-encrypted-secrets-in-a-workflow)

![](using_encrypted_secrets_in_a_workflow.png)

我使用的是环境变量的做法，所以在 `check.js` 中就要去获得环境变量 `COOKIES` 和 `PUSHPLUS` 。

[如何从 Node.js 读取环境变量](http://nodejs.cn/learn/how-to-read-environment-variables-from-nodejs)

``` js
const cookies = process.env.COOKIES?.split('&&') ?? [];

...

const PUSHPLUS = process.env.PUSHPLUS;
```
