title: 基于 rc-tree 的树形数据展示实践
tags: [react,typescript]
categories: [react]
date: 2019-02-13 11:43:00

---

工作中接到一个需求是从模型文件 glb 文件中提取其属性，在页面中通过树形结构来做展示，数据格式大致如下：

```ts
interface dataType {
  autoUpdate:boolean
  background: null
  castShadow: boolean
  uuid: string
  name: string
  userData: {[key:string]: any}
  children?: data[]
  ...
}
```

## 数据精简

这是通过 threejs 解析之后数据，虽然得到的已经是一个树形结构的数据，但在数据中有许多与展示无关的数据，拿到数据后的第一反应是将数据做一下精简，只留下页面展示需要的数据。

可通过如下函数对数据进行精简：

```ts
interface mapTree {
  name: any
  children?: mapTree[]
}

export const mapTree = (data: { [key: string]: any }[]): mapTree[] => {
  return data.map((item: { [key: string]: any }) => {
    if (item.children) {
      return {
        name: item.name,
        children: mapTree(item.children)
      }
    }
    return {
      name: item.name
    }
  })
}
```

接下来就是如何展示的问题了，考虑到如果自己从零开始造轮子从工期和质量上考虑都不好，直接使用 rc-tree 的话要完整实现 ui 提出来的样式和功能比较麻烦，有些情况下还会出现无法实现的情况，综合考量一下决定细读 rc-tree 的源码，将 rc-tree 移植到项目中，在移植过程中结合项目需求再对代码做一些改动。

## 主体目录结构

rc-tree 的主要文件有`contextTypes.js`、`Tree.jsx`、`TreeNode.jsx`、`util.js`4 个文件，`Tree.jsx`是外层树文件，主要负责状态管理和渲染，`TreeNode.jsx`是树节点文件，主要处理单个节点的事件和渲染，`contextTypes.js`是`react context`的类型定义文件，`util.js`主要是一些工具函数。

## 外部函数引用

### toArray

此函数用来保证子节点的 key 不变，并且用来保证正确的返回单个节点的情况，此种情况可以避免 key 不检查的情况

```js
var _react = require('react')

var _react2 = _interopRequireDefault(_react)

function _interopRequireDefault(obj) {
  return obj && obj.__esModule ? obj : { default: obj }
}

/**
 * Use  `toArray` to get the children list which keeps the key.
 * And return single node if children is only one(This can avoid `key` missing check).
 */
function toArray(children) {
  var ret = []
  _react2['default'].Children.forEach(children, function(c) {
    ret.push(c)
  })

  return ret
}

export default toArray
```

注：React.Children 是一个工具类，它提供了对组件不透明数据结构 this.props.children 的处理功能。

React.Children.forEach ，此方法是在每个直接子元素（children）上调用 fn 函数。如果 children 是一个内嵌的对象或者数组，它将被遍历（不会传入容器对象到 fn 中）。如果 children 参数是 null 或者 undefined，那么返回 null 或者 undefined 而不是一个空对象。

```js
React.Children.forEach(object children, function fn [, object thisArg])
```

## Tree.jsx

首先是 Tree 组件，此组件的`getDerivedStateFromProps`是比较核心的一块，此生命周期主要用来接管父级组件传过来的数据以及一些内部状态的更新
