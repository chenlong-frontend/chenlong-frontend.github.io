---
title: css模块化
author: 陈龙
date: 2019-03-06 22:21:16
tags: [css, BEM]
categories: [css]
---

目前主流的 css 规范大致可以分为`OOCSS`、`SMACSS`、`BEM`、`CSS Modules`、`CSS in JS`

在以往写 css 的时候比较随意，加上平时主要精力都在`js`开发上，对 css 的写法知之甚少，但随着工作时间的增加（快一年了），发现自己一直在重复的写一些 css 代码，并且这些代码维护起来也比较困难，每次需要改一个样式时，都需要在浏览器中使用控制台的`Element`查看需要修改区域的`dom`的`class`，然后通过在`css`中进行搜索才能定位到问题样式所在。从我个人开发体验上来讲，感到明显的效率低下和维护困难。

后来在学习的过程中发现了`less`和`sass`，但这两个`css`的预编译框架并没有在写法上提出严格的限制，之前的问题依旧存在。

## OOCSS

全称`Object Oriented CSS`，中文为面向对象 css，其主要有两个原则：

1. `Separation of Structure from Skin` 独立的结构和样式
2. `Separation of Containers and Content` 独立的容器和内容

所谓结构就是元素的大小，样式指颜色等等，第二点即是指将 html 与 css 进行分离，并对 class 进行抽象剥离，我的理解就是，不要将 class 与你具体页面做关联，比如你在开发`home`页，在实现其中的导航时，class 命名不要带上`home`

`OOCSS`的典型代表为`Bootstrap`，以其`button`为例：

```html
<button type="button" class="btn btn-primary btn-lg btn-block">
  button
</button>
```

可以很容易的看出，其主要分为其他几部分

1. `btn`：标准按钮样式
2. `btn-primary`：按钮颜色
3. `btn-lg`：按钮大小
4. `btn-block`：展示为块级按钮

通过不同的`class`组合，就可以形成多种按钮

`OOCSS`的优点如下：

1. 减少 css 代码
2. html 标记简洁，逻辑性强
3. 语义标记，有利于 SEO
4. 由于各种 class 可重用，页面加载更快
5. 拓展性强，不用担心加入的新组件会破坏原有的组件
6. 有利于维护

同时`OOCSS`使用时需要注意如下几点：

1. `OOCSS`主要用于大型网站开发，组件复用情况多
2. `OOCSS`在整体规划上难度颇高，需要有一定的经验的积累，如果使用不当，将会毫无复用性可言，维护也将异常艰难
3. 每个组件都需要一个完善的文档

## SMACSS

了解不多，其通则为：

1. 结构分类：Base、Layout、Moudule、State、Theme
2. 命名规则：id 与 class 受限制的使用，名称之间使用分隔线

## BEM

`BEM`全称`Block Element Modifier`，`BEM`吸收了`OOCSS`的优点，并将 class 分块语义化，在可读性和维护性上个人认为高于`OOCSS`

`BEM`是目前我在实际开发中使用体验最好的一个规范，由于`BEM`只是一个规范，有些部分并没有做强制要求，实际使用时我会根据自身经验和项目情况做一些调整，以当前项目使用情况为例：

styles 文件夹的文件可分为四块

```js
index.less
lib - lay.less
mod - modal.less
pag - home.less
```

`lib`开头的是一些基础的样式或者颜色字体等变量，通用布局和样式重置等，`mod`是各个组件样式，`pag`是页面内特殊样式

以 modal 为例：

```css
.m-modal {
}
.m-modal__header {
}
.m-modal__body {
}
.m-modal__body--nofooter {
}
.m-modal__footer {
}
```

`m-modal`是`Block`，`header`、`body`、`footer`是`Element`，`nofooter`为`Modifier`，B 与 M 之间使用`__`分隔，E 与 M 之间使用`--`分隔，class 在命名时，最多三段，相比其他规范在保持简短的同时提供了 class 的可读性。

每个模块之间都是独立的，以组件进行开发，这点与`OOCSS`很像，`BEM`可复用性好。另外`BEM`是功能为导向的，每个模块代表着某个功能，不会出现`OOCSS`或者`SMASS`中像`.pt-20`(`padding-top:20px`)这种让别人看不懂的名称。

有关 BEM 注意事项可见[这里](https://www.smashingmagazine.com/2016/06/battling-bem-extended-edition-common-problems-and-how-to-avoid-them/)

## CSS Modules

在 react 和 vue 中使用过，其做法是给 class 后拼上一串 hash 值，以此保证样式名称不会冲突

## CSS in JS

代表为[styled-components](https://github.com/styled-components/styled-components)，即将样式写到组件内部。

## 总结

`OOCSS`和`BEM`的实施离不开 ui 设计，在开发之前，ui 需要有自己的一套组件库（图片），开发应当在开发具体页面之前首先在代码层面实现 ui 的零散组件，后续开发除非临时需求或者特殊样式，使用这套 ui 库应当就能组合出所需要的页面。

相对比较而言，最新的两种 css 都需要框架或者编译器的支撑，在普适性上不如前三者，但这两者不存在学习成本，上手比较容易。就目前使用体验来看，`BEM`具有高复用、高拓展、易维护、易阅读、无捆绑，但由于其需要一定的学习成本和实践摸索，想要用好有一定难度。
