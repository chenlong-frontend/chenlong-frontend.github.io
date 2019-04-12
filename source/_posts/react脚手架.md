---
title: 'react脚手架'
author: 陈龙
date: 2019-04-12 09:04:45
tags: [react, 脚手架]
categories: [react]
---

最近有机会研究了一下`wieldy-react`，也算是颇有收获，结合之前创建脚手架的经验，个人依据`wieldy-react`尝试创建了一个`react`工程。

工程目录如下：

```
src                               // 源码根目录
|   index.js                      // 主入口
|   NextApp.js                    // 次入口
|   serviceWorker.js
|
+---apis
|       api.js                     // 接口
|
+---appRedux                       // 状态管理
|   +---actions
|   |       Auth.js
|   |       Chat.js
|   |       index.js
|   |
|   +---reducers
|   |       Auth.js
|   |       Chat.js
|   |       index.js
|   |
|   +---sagas
|   |       Auth.js
|   |       Chat.js
|   |       index.js
|   |
|   \---store                      // store
|           index.js
|
+---assets                         // 资源文件夹
|   \---images                     // 图片资源
+---components                     // 无状态组件文件夹
|   \---CircularProgress
|           index.js
|
+---constants                      // 常量配置
|       ActionTypes.js
|       DomainSetting.js
|
+---containers                     // 有状态组件文件夹
|   |   SignIn.js
|   |   Socket.js
|   |
|   +---App
|   |       index.js
|   |       MainApp.js
|   |
|   +---Chat
|   |       FriendList.js
|   |       index.js
|   |
|   \---Sidebar
|           index.js
|
+---helper                        // 工具函数
|       asyncComponent.js         // 异步加载组件
|
+---routes                        // 页面路由以及页面组件
|   |   index.js
|   |
|   \---Chat
|           index.js
|
\---style                         // 样式
    |   style.less
    |
    +---base                      // 基础样式
    |       base.less
    |       index.less
    |
    +---global                    // 全局样式
    |       index.less
    |       mixin.less
    |       variables.less
    |
    \---layout                    // 布局样式
            ant-layout.less       // antd样式修改
            index.less
```

相比较我之前搭建的脚手架主要的不同点主要有如下几点：

- 将路由集成到了 redux 之中
- 组件拆分到了三个目录当中，分别为`components`，`containers`，`routes`，分别存放无状态组件，有状态容器和路由。

## 局部热更新

虽然`Create React App`已经实现了热更新，但每次修改之后都需要整个重新刷新页面，但很多时候我们只修改了局部的一小块，刷新整个页面会造成不必要的性能浪费，因此可以使用`react-hot-loader`来实现局部刷新。

总体来说，这套开发模式比我之前的个人认为是一种进步，如果对具体细节感兴趣，[请移步](https://github.com/1016482011/chat/tree/save/template)
