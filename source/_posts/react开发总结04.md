---
title: 记如何在现有开发中“偷懒”
author: 陈龙
date: 2019-06-21 13:02:26
tags: [react]
categories: [react]
---

开发中难免有一些看似重复，又有一些区别的页面，在这里总结一下在现有脚手架下如何有效率的生成这些页面。

首先命名上，有一个父菜单名用 `$P`表示，当前页面名用`$C`表示

## 列表展示页

这种页面通常是一个标题，一个搜索，一个工具栏，一个表格

首先是 api：

```js
// src/apis/$P.js

import request from './request'

// 注释
export const xxxx = data => {
  return request({
    url: '',
    method: '',
    data
  })
}
```

redux 部分，在 constants 中定义一个常量：

```js
// src/constants/index.js

//  注释
export const SET_$C_VARIABLE = 'SET_$C_VARIABLE'
```

定义 reducer：

```js
// src/appRedux/reducers/$P/$C

import { SET_$C_VARIABLE } from 'constants/index'

const initialState = {
  list: [], // 当前表格数据
  total: 0, // 总条数
  size: 8, // 每页多少条
  current: 1, // 当前页数
  search: {}, // 搜索条件
  tableloading: false // 表格请求标识
}

const reducer = (state = initialState, action) => {
  switch (action.type) {
    case SET_$C_VARIABLE: {
      return { ...state, [action.payload.name]: action.payload.value }
    }
    default:
      return state
  }
}

export default reducer
```

定义 action：

```js
// src/appRedux/actions/$P/$C

import { message } from 'antd'
import { SET_$C_VARIABLE } from 'constants/index'
import { xxx } from 'apis/$P'

export const setReducer = ({ name, value }) => {
  return {
    type: SET_$C_VARIABLE,
    payload: { name, value }
  }
}

// 获取table列表
export function getListAction() {
  return async (dispatch, getState) => {
    const {
      $C: { size, current, search, tableloading }
    } = getState()
    if (!tableloading)
      dispatch(setReducer({ name: 'tableloading', value: true }))
    const res = await xxx({
      page: { size, current },
      ...search
    }).catch(err => {
      message.error(err.message ? err.message : '数据获取是失败')
      return err
    })
    dispatch(setReducer({ name: 'tableloading', value: false }))
    if (res && res.code) return Promise.reject(res)
    if (res.pages !== 0 && res.current > res.pages) {
      dispatch(changePageAction(res.pages))
      return Promise.reject(res.records)
    }
    dispatch(setReducer({ name: 'list', value: res.records }))
    dispatch(setReducer({ name: 'total', value: res.total }))
  }
}

// 删除
export function xxDeleteAction(id) {
  return async dispatch => {
    dispatch(setReducer({ name: 'tableloading', value: true }))
    const res = await xxDelete({ id }).catch(err => {
      message.error(err.message ? err.message : '数据删除失败')
      return err
    })
    dispatch(setReducer({ name: 'tableloading', value: false }))
    if (res && res.code) return Promise.reject(res)
    dispatch(getListAction())
  }
}

// 修改当前页码
export const changePageAction = page => {
  return async dispatch => {
    dispatch(setReducer({ name: 'current', value: page }))
    dispatch(getListAction())
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

在 store 里添加 reducer

视图层：

```js
// src/containers/$P/$C/index.js

import React, { Component } from 'react'
import { connect } from 'react-redux'
import { Row, Col, Button } from 'antd'
import Widget from 'components/Widget'
import Search from './Search'
import Table from './Table'
import { getListAction } from 'appRedux/actions/InsuranceManage/forcibleInsuranceList'
import style from './index.css'

class TablePage extends Component {
  componentDidMount() {
    this.props.getListAction()
  }

  render() {
    return (
      <Widget title="xxx">
        <Row>
          <Col className={style['col__pd0']} span={20}>
            <Search />
          </Col>
          <Col span={4} className={style['text-align__right']}>
            <Button
              type="primary"
              onClick={() => this.props.history.push(`xxx/xxx`)}
            >
              新增
            </Button>
          </Col>
        </Row>
        <Table history={this.props.history} />
      </Widget>
    )
  }
}

export default connect(
  () => ({}),
  { getListAction }
)(TablePage)
```

对应 index.css

```css
/** src/containers/$P/$C/index.css **/

.col__pd0 {
  padding: 0;
}

.text-align__right {
  text-align: right;
}
```

对应 search.js:

```js
// src/containers/$P/$C/search.js

import React from 'react'
import { connect } from 'react-redux'
import { Form, Button, Input } from 'antd'
import { changeSearchAction } from 'appRedux/actions/$P/$C'

class Search extends React.Component {
  //  搜索
  onSearch = () => {
    const { getFieldsValue } = this.props.form
    const value = getFieldsValue()
    this.props.changeSearchAction(value)
  }
  // 重置搜索
  onResetSearch = () => {
    this.props.form.resetFields()
    this.props.changeSearchAction({})
  }

  render() {
    const { getFieldDecorator } = this.props.form
    return (
      <Form layout="inline">
        <Form.Item>
          {getFieldDecorator('XXX')(<XXXX placeholder="XXX" />)}
        </Form.Item>
        <Form.Item>
          <Button type="primary" htmlType="submit" onClick={this.onSearch}>
            搜索
          </Button>
          <Button onClick={this.onResetSearch}>重置</Button>
        </Form.Item>
      </Form>
    )
  }
}

export default connect(
  () => ({}),
  { changeSearchAction }
)(Form.create()(Search))
```

对应 table.js

```js
// src/containers/$P/$C/table.js

import React, { Fragment } from 'react'
import { connect } from 'react-redux'
import { Button, Table } from 'antd'
import { changePageAction, xDeleteAction } from 'appRedux/actions/$P/$C'
import { DropButtonGroup } from 'containers/Common'
import columns from './columns'

class TableList extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      columns: columns.concat([this.action])
    }
  }

  // 点击页码触发
  onPage = ({ current }) => this.props.changePageAction(current)
  action = {
    title: '操作',
    dataIndex: 'id',
    width: 170,
    render: (id, row) => {
      return (
        <Fragment>
          <Button
            className={'gx-mb-0'}
            size="small"
            onClick={() => this.props.history.push(`/x/x/${id}`)}
          >
            详情
          </Button>
          <DropButtonGroup
            group={[
              {
                text: ' 编辑',
                onClick: () => this.props.history.push(`/x/x/${id}`)
              },
              {
                text: '删除',
                type: 'delete',
                onClick: () => this.props.xDeleteAction(id)
              }
            ]}
          />
        </Fragment>
      )
    }
  }

  render() {
    const { tableloading, total, size, current, list } = this.props
    return (
      <Table
        loading={tableloading}
        pagination={{
          total,
          pageSize: size,
          current
        }}
        rowKey="id"
        dataSource={list}
        columns={this.state.columns}
        onChange={this.onPage}
        bordered={true}
      />
    )
  }
}

export default connect(
  ({ xxxx: { list, tableloading, total, size, current } }) => ({
    list,
    tableloading,
    total,
    size,
    current
  }),
  {
    changePageAction,
    xDeleteAction
  }
)(TableList)
```

table 对应的 columns

```js
// src/containers/$P/$C/columns.js

import React from 'react'
import moment from 'moment'
import { Tag } from 'antd'

export default [
  {
    title: '名称',
    dataIndex: 'key'
  },
  {
    title: '日期',
    dataIndex: 'key',
    render: v => (!v ? null : moment(v).format('YYYY-MM-DD'))
  },
  {
    title: 'xx',
    dataIndex: 'xx',
    render: value => {
      return (
        <Tag className="gx-mb-0" color={value ? 'green' : 'red'}>
          {value ? 'xxx' : 'xxx'}
        </Tag>
      )
    }
  }
]
```

最后路由里配一下：

```js
const $C = asyncComponent(() => import(/* webpackChunkName:"$C" */ './$C'))
;<Route path={`${match.url}/$C`} component={$C} />
```

## 增和改

上面的只是查，常有的还有增和改

首先还是 api：

```js
// src/apis/$P.js

import request from './request'

// 注释
export const xxxx = data => {
  return request({
    url: '',
    method: '',
    data
  })
}
```

redux 部分，在 constants 中定义一个常量：

```js
// src/constants/index.js

//  注释
export const SET_$C_VARIABLE = 'SET_$C_VARIABLE'
```

定义 reducer：

```js
// src/appRedux/reducers/$P/$C

import { SET_$C_VARIABLE } from 'constants/index'

const initialState = {
  details: null
}

const reducer = (state = initialState, action) => {
  switch (action.type) {
    case SET_$C_VARIABLE: {
      return { ...state, [action.payload.name]: action.payload.value }
    }
    default:
      return state
  }
}

export default reducer
```

定义 action：

```js
// src/appRedux/actions/$P/$C

import { message } from 'antd'
import { SET_$C_VARIABLE } from 'constants/index'
import { isUndefined } from 'lodash'
import { xxx } from 'apis/$P'

export const setReducer = ({ name, value }) => {
  return {
    type: SET_$C_VARIABLE,
    payload: { name, value }
  }
}

// 车辆信息新增
export function submitAction(param) {
  return async () => {
    const res = await xxx(param).catch(err => {
      message.error(err.message ? err.message : '提交失败')
      return err
    })
    if (res && res.code) return Promise.reject(res)
    message.success('提交成功')
  }
}

// 车辆信息更新
export function updateAction(param) {
  return async dispatch => {
    const res = await xxx(param).catch(err => {
      message.error(err.message ? err.message : '更新失败')
      return err
    })
    if (res && res.code) return Promise.reject(res)
    dispatch(push('/xxx/xx'))
  }
}

// 根据ID获取主体信息管理
export function getDataByIdAction(id) {
  return async dispatch => {
    dispatch(setVariable({ name: 'details', value: null }))
    const res = await xxx({ id }).catch(err => {
      message.error(err.message ? err.message : '数据获取失败')
      return err
    })
    if (isUndefined(res) || (res && res.code)) return Promise.reject(res)
    dispatch(setVariable({ name: 'details', value: res }))
  }
}
```

在 store 里添加 reducer

视图部分：

```js
// src/containers/$P/$C/index.js

import React from 'react'
import { Button, Empty } from 'antd'
import { isUndefined } from 'lodash'
import { connect } from 'react-redux'
import { SimpleForm } from 'components/Form'
import { mapFileListToUrl, mapUrlToFileList } from 'components/Upload'
import PageBackContent from 'components/PageBackContent'
import { formItem } from './formItem'
import { onRouterChange } from '../../../navigation'
import {
  getDataByIdAction,
  submitAction,
  updateAction
} from 'appRedux/actions/$P/$C'
import style from './index.css'

// 时间字段
const timeFields = ['']

class AddEditForm extends React.Component {
  constructor() {
    super()
    this.state = {
      isAdd: true,
      id: null,
      getDataFail: false,
      hide: true,
      isSubmit: false
    }
  }

  componentDidMount() {
    const {
      match: {
        params: { id }
      },
      getDataByIdAction
    } = this.props

    onRouterChange('/xxx/xxx', () => {
      this.setState({ isAdd: !this.state.isAdd })
      this.resetFields()
    })

    // 如果是编辑页面，取得页面初始值
    if (isUndefined(id)) {
      this.setState({ hide: false })
    } else {
      this.setState({ isAdd: false, id })
      getDataByIdAction(id)
        .catch(() => this.setState({ getDataFail: true }))
        .finally(() => {
          this.setEdit()
          this.setState({ hide: false })
        })
    }
  }

  setEdit = () => {
    let { datas } = this.props

    // 转换时间格式
    datas = SimpleForm.timeToMoment(datas, timeFields)
    datas.uploadPhotos = mapUrlToFileList(datas.uploadPhotos)
    SimpleForm.setFieldsValue(datas, this.formRef)
  }

  submitDataFormat = values => {
    values = SimpleForm.momentFormat(values, timeFields)
    values = SimpleForm.undefinedToNull(values)
    values.uploadPhotos = mapFileListToUrl(values.uploadPhotos, 1)
    return values
  }

  // 表单提交
  onClickSubmit = () => {
    const { submitAction, updateAction } = this.props
    const { id, isAdd, isSubmit } = this.state
    const { validateFieldsAndScroll } = this.formRef
    if (isSubmit) return
    validateFieldsAndScroll((err, values) => {
      if (err) return

      this.setState({ isSubmit: true }, () => {
        values = this.submitDataFormat(values)
        if (isAdd) {
          // 新增提交
          submitAction(values)
            .then(() => this.resetFields())
            .finally(() => this.setState({ isSubmit: false }))
        } else {
          // 编辑提交
          values.id = id
          updateAction(values).catch(() => {
            this.setState({ isSubmit: false })
          })
        }
      })
    })
  }
  resetFields = () => {
    const { resetFields } = this.formRef
    resetFields()
  }

  render() {
    const { token } = this.props
    const { getDataFail, hide, isAdd, isSubmit } = this.state
    return (
      <div>
        <div
          style={{ display: hide ? 'none' : 'block' }}
          className={style.container}
        >
          <PageBackContent title={`xxx${isAdd ? '新增' : '编辑'}`}>
            {getDataFail && (
              <Empty style={{ padding: '50px 0' }} description="数据异常" />
            )}
            <div style={{ display: getDataFail ? 'none' : 'block' }}>
              <SimpleForm
                model="divide"
                itemCol={3}
                wrappedComponentRef={formRef =>
                  formRef &&
                  !this.formRef &&
                  (this.formRef = formRef.props.form)
                }
                items={formItem({ token })}
              />
              <div className={style.submit}>
                <Button
                  loading={isSubmit}
                  className={style.submitBtn}
                  type="primary"
                  onClick={this.onClickSubmit}
                >
                  {isAdd ? '提交' : '确认修改'}
                </Button>
              </div>
            </div>
          </PageBackContent>
        </div>
      </div>
    )
  }
}

const mapStateToProps = ({ runtime: { token }, xxx: { details } }) => ({
  token,
  datas: details
})
export default connect(
  mapStateToProps,
  {
    getDataByIdAction,
    submitAction,
    updateAction
  }
)(AddEditForm)
```

对应 index.css

```css
/** src/containers/$P/$C/index.js **/

.container {
  height: 100%;

  .formItemWidth {
    width: 100%;
  }

  .uploadLabel {
    line-height: 40px;
    text-align: right;
  }

  .submit {
    margin-top: 15px;
    text-align: center;
    .submitBtn {
      width: 150px;
    }
  }
}
```

对应 formItem：

一级 formItem：

```js
import React from 'react'
import { DatePicker } from 'antd'
import { EWPicturesWall } from 'components/Upload'
// import { xx } from 'containers/Common/'
import style from './index.css'

const formItem = function({ token }) {
  return [
    {
      label: '名称',
      required: true,
      key: 'xx'
    },
    {
      label: '日期',
      key: 'xx',
      component: (
        <DatePicker placeholder="日期" className={style.formItemWidth} />
      )
    },
    {
      label: '上传照片',
      key: 'xxx',
      col: 24,
      layout: {
        labelCol: { span: 3 },
        wrapperCol: { span: 9 }
      },
      component: (
        <EWPicturesWall token={token} data={{ model: 'xxx' }} limit={1} />
      )
    }
  ]
}
export { formItem }
```

二级 formItem：

```js
const formItem = function({ token }) {
  return [
    {
      title: '必填信息',
      children: []
    }
  ]
}
```
