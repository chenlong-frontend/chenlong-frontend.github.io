title: 在 react 中实现 tree
tags: []
categories: []
date: 2019-02-13 11:43:00

---

工作中接到一个需求是从模型文件 glb 文件中提取其属性，在页面中通过树形结构来做展示，数据格式大致如下：

```js
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

# 数据精简

这是通过 threejs 解析之后数据，虽然得到的已经是一个树形结构的数据，但在数据中有许多与展示无关的数据，拿到数据后的第一反应是将数据做一下精简，只留下页面展示需要的数据。

可通过如下函数对数据进行精简：

```js
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
