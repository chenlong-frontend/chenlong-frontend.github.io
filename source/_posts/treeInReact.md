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

rc-tree 的主要文件有`contextTypes`、`Tree`、`TreeNode`、`util`4 个文件，`Tree`是外层树文件，主要负责状态管理和渲染，`TreeNode`是树节点文件，主要处理单个节点的事件和渲染，`contextTypes`是`react context`的类型定义文件，`util`主要是一些工具函数。

## 外部函数引用

### toArray

此函数用来保证子节点的 key 不变，并且用来保证正确的返回单个节点的情况，此种情况可以避免 key 不检查的情况

```js
var _react = require('react')

var _react2 = _interopRequireDefault(_react)

function _interopRequireDefault(obj) {
  return obj && obj.__esModule ? obj : { default: obj }
}

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

### getDerivedStateFromProps

Tree 组件中的`getDerivedStateFromProps`是比较核心的一块，此生命周期主要用来接管父级组件传过来的数据以及一些内部状态的更新。

```ts
// Tree.tsx
function needSync(name: string) {
  return (
    (!prevProps && name in props) ||
    (prevProps && prevProps[name] !== props[name])
  )
}
```

此函数用来检查 props 中指定属性是否发生变化以此避免掉不必要的更新。

getDerivedStateFromProps 中主要分为`Tree Node`、`checkedKeys`四个模块。

首先是`Tree Node`模块，这里用到了不少`util`里的函数。简单过一下

```ts
// util.ts
function isTreeNode(
  node: null | undefined | { type?: { isTreeNode?: boolean } }
) {
  return node && node.type && node.type.isTreeNode
}

function getNodeChildren(children: types.treeNode) {
  return toArray(children).filter(isTreeNode)
}
```

`getNodeChildren`函数用来过滤掉非 tree 节点的元素，注意`node.type.isTreeNode`中的`isTreeNode`是在`TreeNode`组件中的自定义属性，值为`1`。

```ts
// util.ts
function traverseTreeNodes(
  treeNodes: types.treeNode,
  callback: (item: types.traverseTreeNodesData) => void
) {
  function processNode(
    node: React.ReactElement<any> | null,
    index: number,
    parent: { node: React.ReactElement<any> | null; pos: string }
  ) {
    const children = node ? node.props.children : treeNodes
    const pos = node ? getPosition(parent.pos, index) : '0'
    const childList = getNodeChildren(children)
    if (node) {
      const data: types.traverseTreeNodesData = {
        node,
        index,
        pos,
        key: node.key || pos,
        parentPos: parent.node ? parent.pos : null
      }
      callback(data)
    }
    Children.forEach(childList, (subNode, subIndex) => {
      if (typeof subNode !== 'string' && typeof subNode !== 'number')
        processNode(subNode, subIndex, { node, pos })
    })
  }
  processNode(null, 0, {} as any)
}

function convertTreeToEntities(treeNodes: types.treeNode) {
  const posEntities: { [key: string]: types.treeToEntities } = {} as any
  const keyEntities: { [key: string]: types.treeToEntities } = {} as any
  let wrapper = {
    posEntities,
    keyEntities
  }
  traverseTreeNodes(treeNodes, (item: types.traverseTreeNodesData) => {
    const { node, index, pos, key, parentPos } = item
    const entity: types.traverseTreeEntityData = { node, index, key, pos }
    posEntities[pos] = entity
    keyEntities[key] = entity
    entity.parent = parentPos && posEntities[parentPos]
    if (entity.parent) {
      entity.parent.children = entity.parent.children || []
      entity.parent.children.push(entity)
    }
  })
  return wrapper
}
```

`traverseTreeNodes`用于递归出所传入节点下的所有子节点，并通过`React.Children.forEach`得出子节点的深度，再函数`convertTreeToEntities`将树中的节点展平，将封过后的结果返回给外界使用。

再返回 Tree 组件的 `Tree node`部分

```ts
// Tree.tsx
let treeNode: types.treeNode = null

// 检查children是否发生了变化，如果变化了更新state
if (needSync('children')) {
  treeNode = toArray(props.children)
}

if (treeNode) {
  newState.treeNode = treeNode

  // 计算出节点的实体数据以便快速匹配
  const entitiesMap = convertTreeToEntities(treeNode)
  newState.posEntities = entitiesMap.posEntities
  newState.keyEntities = entitiesMap.keyEntities
}
```

此处部分简单的调用了`convertTreeToEntities`方法，并把得到的数据存到 state 中，`let treeNode: types.treeNode = null`说明可以出此操作只会在 children 发生变化才会触发。

下面是`checkedKeys`，这一块保证外部状态变化之后，能够及时检查选中的 key

```ts
// Tree.tsx
let checkedKeyEntity: {
  checkedKeys: string[]
  halfCheckedKeys: string[]
} | null = null
if (treeNode) {
  checkedKeyEntity = {
    checkedKeys: prevState.checkedKeys,
    halfCheckedKeys: prevState.halfCheckedKeys
  }
}
if (checkedKeyEntity) {
  let { checkedKeys = [], halfCheckedKeys = [] } = checkedKeyEntity
  const conductKeys = conductCheck(checkedKeys, true, keyEntities)
  checkedKeys = conductKeys.checkedKeys
  halfCheckedKeys = conductKeys.halfCheckedKeys
  newState.checkedKeys = checkedKeys
  newState.halfCheckedKeys = halfCheckedKeys
}
```

如果 treeNode 发生变化，及时检查已选和半选择的 keys

最后通过返回 `newState` 更新 `Tree`组件的内部状态

### renderTreeNode

`renderTreeNode`是一个比较重要的函数，主要负责节点的渲染

```tsx
// Tree.tsx
renderTreeNode = (
  child: React.ReactElement<Props>,
  index: number,
  level = 0
) => {
  const { expandedKeys, halfCheckedKeys, selectedKeys } = this.state
  const pos = getPosition(level, index)
  const key = child.key || pos
  const newProps = {
    key,
    eventKey: key,
    expanded: expandedKeys.indexOf(key) !== -1,
    pos,
    checked: this.isKeyChecked(key),
    halfChecked: halfCheckedKeys.indexOf(key) !== -1,
    selected: selectedKeys.indexOf(key) !== -1
  }
  return React.cloneElement<Props>(child, newProps)
}
```

此函数的`newProps`较为重要，节点的主要状态，像选中、半选中、展开等信息均在此处注入到了`props`里

`Tree`先暂时告一段落，下面是转到`TreeNode`组件

## TreeNode

`TreeNode`从`render`函数开始解读，代码如下:

```tsx
// TreeNode.tsx
public render() {
  return (
    <li>
      {this.renderSwitcher()}
      {this.renderCheckbox()}
      {this.renderSelector()}
      {this.renderChildren()}
    </li>
  )
}
```

主要分为四块，`renderSwitcher`是控制列表展开与折叠的按钮，`renderCheckbox`是`checkbox`框，`renderSelector`是列表的主体内容部分，同时用作选择区，`renderChildren`如果此节点存在子节点的情况下，使用此函数来进行渲染子元素列表。

在解读源码时，我删除了样式部分的内容，以便阅读。所以最终效果会有点 low

### renderSwitcher

```tsx
// TreeNode.tsx
onExpand = (e: React.MouseEvent<HTMLSpanElement, MouseEvent>) => {
  const {
    rcTree: { onNodeExpand }
  } = this.context
  onNodeExpand(e, this)
}
renderSwitcher = () => {
  return <span onClick={this.onExpand}>点击展开</span>
}
```

上述代码比较简单，主要功能实现代码在于`onNodeExpand`，这是在`Tree`组件内实现的代码

```tsx
// Tree.tsx
onNodeExpand = (
  e: React.MouseEvent<HTMLSpanElement, MouseEvent>,
  treeNode: React.ReactElement<Props>
) => {
  let { expandedKeys } = this.state
  const { eventKey, expanded } = treeNode.props
  const targetExpanded = !expanded
  if (targetExpanded) {
    expandedKeys = arrAdd(expandedKeys, eventKey)
  } else {
    expandedKeys = arrDel(expandedKeys, eventKey)
  }
  this.setUncontrolledState({ expandedKeys })
  return null
}
```

`eventKey`是当前选中节点的 key，`expanded`是当前展开的状态，如果`expanded`为 true，则`expandedKeys`添加`eventKey`，反之删除`eventKey`，最后更新到`state`之中。

### renderCheckbox

```tsx
// TreeNode.tsx
onCheck = (e: React.MouseEvent<HTMLSpanElement, MouseEvent>) => {
  const { checked } = this.props
  const {
    rcTree: { onNodeCheck }
  } = this.context
  e.preventDefault()
  const targetChecked = !checked
  onNodeCheck(e, this, targetChecked)
}
renderCheckbox = () => {
  const { checked, halfChecked } = this.props
  return (
    <span onClick={this.onCheck}>
      {checked ? '选中|' : '未选中|'}
      {halfChecked ? '半选中|' : '未半选中|'}
    </span>
  )
}
```

与`renderSwitcher`一样，此处仅负责渲染，具体的功能由`Tree`组件中的`onNodeCheck`实现

在`onNodeCheck`中使用的`conductCheck`是一个比较重要的函数

```ts
export function conductCheck(
  keyList: string[],
  isCheck: boolean,
  keyEntities: types.traverseTreeEntityData,
  checkStatus: any = {}
) {
  const checkedKeys: { [key: string]: any } = {}
  const halfCheckedKeys: { [key: string]: any } = {} // 记录子节点被选中的key(包括子节点半选中)
  ;(checkStatus.checkedKeys || []).forEach((key: any) => {
    checkedKeys[key] = true
  })
  ;(checkStatus.halfCheckedKeys || []).forEach((key: any) => {
    halfCheckedKeys[key] = true
  })

  // Conduct up
  function conductUp(key: any) {
    if (checkedKeys[key] === isCheck) return

    const entity = keyEntities[key]
    if (!entity) return

    const { children, parent, node } = entity

    if (isCheckDisabled(node)) return

    // Check child node checked status
    let everyChildChecked = true
    let someChildChecked = false // Child checked or half checked
    ;(children || [])
      .filter((child: any) => !isCheckDisabled(child.node))
      .forEach(({ key: childKey }: any) => {
        const childChecked = checkedKeys[childKey]
        const childHalfChecked = halfCheckedKeys[childKey]

        if (childChecked || childHalfChecked) someChildChecked = true
        if (!childChecked) everyChildChecked = false
      })

    // Update checked status
    if (isCheck) {
      checkedKeys[key] = everyChildChecked
    } else {
      checkedKeys[key] = false
    }
    halfCheckedKeys[key] = someChildChecked

    if (parent) {
      conductUp(parent.key)
    }
  }

  // Conduct down
  function conductDown(key: any) {
    if (checkedKeys[key] === isCheck) return

    const entity = keyEntities[key]
    if (!entity) return

    const { children, node } = entity

    if (isCheckDisabled(node)) return

    checkedKeys[key] = isCheck
    ;(children || []).forEach((child: any) => {
      conductDown(child.key)
    })
  }

  function conduct(key: any) {
    const entity = keyEntities[key]

    if (!entity) {
      console.warn(`'${key}' does not exist in the tree.`)
      return
    }

    const { children, parent, node } = entity
    checkedKeys[key] = isCheck

    if (isCheckDisabled(node)) return // Conduct down
    ;(children || [])
      .filter((child: any) => !isCheckDisabled(child.node))
      .forEach((child: any) => {
        conductDown(child.key)
      })

    // Conduct up
    if (parent) {
      conductUp(parent.key)
    }
  }
  ;(keyList || []).forEach((key: any) => {
    conduct(key)
  })
  const checkedKeyList: any = []
  const halfCheckedKeyList: any = []
  Object.keys(checkedKeys).forEach(key => {
    if (checkedKeys[key]) {
      checkedKeyList.push(key)
    }
  })
  Object.keys(halfCheckedKeys).forEach(key => {
    if (!checkedKeys[key] && halfCheckedKeys[key]) {
      halfCheckedKeyList.push(key)
    }
  })
  return {
    checkedKeys: checkedKeyList,
    halfCheckedKeys: halfCheckedKeyList
  }
}
```

`conductCheck`接受四个参数，`keyList`选中的 key 的集合，`checked`表示执行的是勾选还是反勾选操作，`keyEntities`是以 key 为属性名的节点实体列表，`checkStatus`是当前已选择的 key 和半选择的 key 的集合。

```tsx
onNodeCheck = (
  e: React.MouseEvent<HTMLSpanElement>,
  treeNode: any,
  checked: boolean
) => {
  const {
    keyEntities,
    checkedKeys: oriCheckedKeys,
    halfCheckedKeys: oriHalfCheckedKeys
  } = this.state
  const {
    props: { eventKey }
  } = treeNode
  const eventObj: any = {
    event: 'check',
    node: treeNode,
    checked,
    nativeEvent: e.nativeEvent
  }

  const { checkedKeys, halfCheckedKeys } = conductCheck(
    [eventKey],
    checked,
    keyEntities,
    {
      checkedKeys: oriCheckedKeys,
      halfCheckedKeys: oriHalfCheckedKeys
    }
  )

  eventObj.checkedNodes = []
  eventObj.checkedNodesPositions = []
  eventObj.halfCheckedKeys = halfCheckedKeys
  checkedKeys.forEach((key: string) => {
    const entity = keyEntities[key]
    if (!entity) return
    const { node, pos } = entity
    eventObj.checkedNodes.push(node)
    eventObj.checkedNodesPositions.push({ node, pos })
  })

  this.setUncontrolledState({
    checkedKeys,
    halfCheckedKeys
  })
}
```
