---
title: react之redux
author: 陈龙
date: 2019-05-23 13:02:26
tags: [react, redux]
categories: [react, redux]
---

在学习 redux 之初，我们应该都看过官网的例子，其中会提到容器的概念，当时我并不理解何为容器以及为何需要容器，在经历一段时间的开发后对容器的概念有了自己的理解。

以一个常见的列表搜索分页的功能来说，要实现一个如图所示的页面：![react01](/img/react01.png)

可按如下步骤进行：

### 接口

如果后台接口以及到位或者有 mock 接口，可事先在 api 文件里把接口先定义好，如果接口尚未完成，在确认字段后在 api 里返回假数据，例子如下：

```js
// api.js
// 接口已存在
export const apis = data => {
  return request({
    url: '/api/xxx',
    method: 'POST',
    data
  })
}
// 接口尚未定义
export const apis = data => {
  return Promise.resolve({xx: 'xx'})
}
```

### reducer 定义

接口完成之后，定义页面中我们需要的用到的一些状态，目前我们需要的暂定为以下这些：

```js
const initialState = {
  list: [],                // 当前表格数据
  total: 0,                // 总条数
  size: 8,                 // 每页多少条
  current: 1,              // 当前页数
  search: {}               // 搜索条件
  tableloading: false      // 表格请求标识
}
```

### action 定义

状态定义完成之后就是 action 了，首先我个人也是比较讨厌在 reducer 中写一堆 case 条件判断的，所以在每个模块下的会有一个用来设置的 action：

```js
// action.js
export const setReducer = ({ name, value }) => {
  return {
    type: SET_XXX_REDUCER,
    payload: { name, value }
  }
}
// reducer.js
...
case SET_XXX_REDUCER: {
  return { ...state, [action.payload.name]: action.payload.value }
}
```

有了这个用来设值得 action 之后，接着就是把所有的接口请求分别写成一个 action，就以获取列表的例子来说：

```js
export function getList() {
  return async (dispatch, getState) => {
    try {
      // 获取请求参数
      const {
        xxx: { size, current, search }
      } = getState()
      dispatch(setReducer({ name: 'tableloading', value: true }))
      const res = await roles({
        page: { size, current },
        ...search
      })
      dispatch(setReducer({ name: 'list', value: res.list }))
      dispatch(setReducer({ name: 'total', value: res.total }))
      return Promise.resolve(res)
    } catch (error) {
      message.error(err.message ? err.message : '数据获取失败')
      return Promise.reject(res)
    } finally {
      dispatch(setReducer({ name: 'tableloading', value: false }))
    }
  }
}
```

写完 api 对应的 action 之后，在分析一下页面的功能，我们还需要一个改变页码和搜索条件的 action：

```js
// 修改当前页码
export const changePageAction = page => {
  return async dispatch => {
    dispatch(setReducer({ name: 'current', value: page }))
    dispatch(getList())
  }
}

// 修改查询条件
export const changeSearchAction = search => {
  return async dispatch => {
    dispatch(setReducer({ name: 'search', value: search }))
    dispatch(changePageAction(1))
  }
}
```

只要保证 reducer 里的值均有输入的 action 此阶段工作就结束了，当然如果像 pageSize 这种不需要改变的可暂且不管。

### 组件

状态都已经准备完毕，下面着手组件的编码了，首先按照功能和布局我将页面分为如下几块：![react02](/img/react02.png)

这里其实还有一个新增和编辑的模态框，这个后面会提及到。

首先创建四个文件：`index.js`、`Search.js`、`Tool.js`、`Table.js`，其中`index.js`是一个总的页面用来组装容器组件和总体布局，剩下的三个容器组件分别对应图片中的 1，2，3

容器组件初始化的时候先将此组件需要使用的 state 和 action 注入进来，然后再进行具体功能的编码，大致如下：

```jsx
import React, { Component } from 'react'
import { connect } from 'react-redux'
class MyTable extends Component {
  render() {
    return <div />
  }
}
export default connect(
  ({ xxx: { current, size, list, total, tableloading } }) => ({
    current,
    size,
    list,
    total,
    tableloading
  }),
  {
    changePageAction
  }
)(MyTable)
```

此处需要注意的是此处除去`index.js`之外的组件均是有状态的容器组件，如果我们需要可以直接复用组件的组件大部分情况下最好不要使用容器组件。

### 页面

组件全都完成之后，在`index.js`进行简单的拼装布局此页面的分页条件查询功能也就完成。

### 复用

此种容器组件没法像通用组件直接复用，但只要组件分解得当，遇到同样功能的时候照样可以做到复制粘贴稍加修改即可使用，这里就以之前留下的新增编辑的模态框来举例。我要实现的功能大致如下，需要同时实现新增和编辑功能：![react03](/img/react03.png)

啥都不说，先上代码：

```jsx
import React, { Component } from 'react'
import { connect } from 'react-redux'
import { Input, Modal } from 'antd'
import { isNull } from 'lodash'
import { EwForm } from 'components/Form'
import { addAction, editAction,hideModalAction} from 'appRedux/actions/UserManage/roleGrid'

const { TextArea } = Input
const formItem = [
  {
    label: '角色名称',
    key: 'name',
    required: true
  },
  {
    label: '备注',
    key: 'description',
    component: <TextArea placeholder="备注" />
  }
]

class MyMoal extends Component {
  constructor() {
    this.formRef = React.createRef()
  }

  componentDidUpdate(preProps, preState) {
    const { fields } = this.props
    // 每次进模态框，给表单设置初始值
    if (this.formRef.current && preProps.fields !== fields) {
      const { setFieldsValue } = this.formRef.current
      isNull(fields) ? this.reset() : setFieldsValue(fields)
    }
  }

  // 提交表单
  onSubmit = () => {
    const { isSubmit } = this.state
    const { user, addAction, onOk, fields, editAction } = this.props
    const { validateFields } = this.formRef.current
    const isEdit = !!fields
    validateFields((err, values) => {
      if (err) return

      this.setState({ isSubmit: true }, () => {
        if (isEdit) {
          values.id = fields.id
          editAction(values)
            .then(() => this.reset())
        } else {
          values.createBy = user.id
          addAction(values)
            .then(() => {this.reset())
        }
      })
    })
  }
  // 重置表单
  reset = () => this.formRef.current.resetFields()

  render() {
    const { visible, fields,isSubmit,hideModalAction } = this.props

    return (
      <Modal
        title={`${fields ? '编辑' : '新增'}`}
        visible={visible}
        onOk={this.onSubmit}
        onCancel={hideModalAction}
        confirmLoading={isSubmit}
      >
        <EwForm ref={this.formRef} items={formItem} itemCol={1} />
      </Modal>
    )
  }
}

const mapStateToProps = ({ xx: { fields, visible,isSubmit } }) => ({ fields, visible,isSubmit })

export default connect(
  mapStateToProps,
  { addAction, editAction,hideModalAction }
)(MyMoal)
```

现在看下如何将这个组件放到页面中，首先看状态，需要在 reducer 中添加`fields`、`visible`和`isSubmit`三个，分别用来控制表单初始值和模态框的显示以及提交时的等待标识。然后是五个 action，`addAction`、`editAction`、`editShowModalAction`、`addShowModalAction`、`hideModalAction`，分别用来新增请求、编辑请求、编辑显示、新增显示和隐藏，最后在`Tool.js`中添加上`showModalAction`，在`Table.js`中添加上`editShowModalAction`，别看前面这这么多 action，state，我们要做的就是复制粘贴即可，不过只是粘贴过来肯定是没法用的，有以下几点需要我们做些修改，当然这里我只谈比较通用的，具体的可以结合自己各自的页面做调整，首先是`addAction`，`editAction`中将 api 函数替换掉，然后是组件中的`formItem`替换为自己的表单。

接下来就是测试看功能是否完整了=。=

### 总结

只要我们将页面都拆分为一个个有状态的容器，随着开发的模块越来越多，我们将发现，越来越多的页面可以使用容器进行拼装，效率也会越来越高。

拆分容器的时候一定要秉持一个理念：容器组件不需要大而全，但一定是可拔插的。也就是这个容器组件一定要方便复制粘贴，但没必要覆盖所有需求。
