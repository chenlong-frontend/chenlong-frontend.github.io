---
title: kview
author: 陈龙
date: 2020-11-05 19:01:02
tags: ['kview', 'vue', 'typescript']
categories: ['kview']
---

今年年中接手了一个全新的项目，定位是半解决方案半sdk，之所以加个半是因为，我们需要对外提供一个开箱即用的应用，但同时第三方也可以屏蔽掉我们ui，通过我们暴露的接口实现一套和我们不同风格但功能一样的应用。

我的方案是，将项目切成三块
 - 接口sdk
 - ui基础组件库
 - 业务功能实现

接口层使用typescript实现，ui组件库和业务层使用vue + typescript。

由于这个项目特殊地方在于需要上大屏展示，用普通的ui 库简单改改无法满足ui 要求，所以在组件上面要求我从 0开始。

这次要讲的 kview 便是 ui 基础组件库。为了避免后续的重复劳动，和规范 ui的设计，我在项目之初，在实现完接口sdk后，并没有急着实现业务功能，而是拆解ui图纸上的一个个小组件，在实现完这些小组件后再进行业务功能的拼装。

由于我之前也没有过实现一个ui 库的经验，所以我选择了以同样是 vue的element ui为老师，遇到不清楚的地方，便会一点点的去阅读 element ui的源码，然后应用到自身的项目中（因为我们项目是 typescript的，直接复制源码是无法使用的）。

在这过程中，我印象比较深刻的有 `select`组件中 `emitter`的使用，完美解决了`select`和`option`之前的事件触发问题。同样的还有`dialog`中的`popup`下拉的`popper`以及`form`组件，这些都让我收益良多。

在样式方面，我之前使用 scss或者less基本就只使用了个嵌套写法，其他和写css一样，这次通过研读element ui的BEM函数的使用，并将至成功应用到了kview中。

通过这次kview的折腾，对 ui的库的开发和实现有了更深刻的理解和认识。下面附上地址。

[kview github](https://github.com/chenlong-frontend/vue-technology-ui)

[kview 文档](http://119.3.156.49:9090/#/zh-CN/)
