---
title: 关于设置localStorage时间
author: 陈龙
date: 2019-03-22 22:56:04
tags: [javascript,localStorage]
categories: [javascript]
---

今天被问到如何实现给localStorage添加过期时间。经过慢慢思考，我想到的方法下可以归结为以下几种：

1. 将时间拼接到值之后。比如`set("username","1553264048068;",1553264048068)`，实际存的值为`localStorage.setItem("username","1553264048068;;1553264048068")`。取值的策略时反向寻找到第一个分割符`;`,并将将其截取。
2. 使用对象进行二次组合。比如`set("username","Marcus",1553264048068)`,实际存的值为`localStorage.setItem({"value":"Marcus","expire":1553264048068})`。取值的策略是`JSON.parse`后去取出值和时间。

两种方法的本质都一样，都是将值和时间进行了拼接，在我现在看来这两个方法有个最大的问题就是修改了原来的值，在解析的时候往往是bug产生的源泉，因为你很难去预期使用者会传入什么值。

后来在阅读了[store.js](https://github.com/marcuswestin/store.js)后，恍然大悟，`namespace`，我无法预期用户值的类型，但我可以预期name的类型，在name上加上一个同为string类型的命名空间便可解决这个问题。`set("username","Marcus",1553264048068)`，实际存为`localStorage.setItem("username":"Marcus")`和`localStorage.setItem("expire_username":"1553264048068")`。