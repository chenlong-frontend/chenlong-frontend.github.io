---
title: react工作的一些总结
author: 陈龙
date: 2019-04-22 9:08:16
tags: [react]
categories: [react]
---

## Modal

根据[文档描述](https://react.docschina.org/docs/portals.html)，portals 可以将子节点渲染到父节点之外的 DOM 当中，这在子组件需要“跳出”容器时非常有用，例如模态框。

其实现核心代码简单如下：

```tsx
import React from 'react';
import ReactDOM from 'react-dom';
import { Modal as ModalContainer, Props as ModalProps } from './Modal';

interface Props extends ModalProps {}

interface State {}

class Modal extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
  }

  private el = document.createElement('div');

  componentDidMount() {
    document.body.appendChild(this.el);
  }

  render() {
    return ReactDOM.createPortal(<ModalContainer {...this.props} />, this.el);
  }
}

export { Modal };
```

## 将 src 加入引入路径

当文件嵌套过深时，可能会存在这样的路径引入-`../../../`，这种相对路径的引入方式可能会对代码阅读造成困扰，一时间不知道文件具体路径在哪(如果编辑器没有`ctrl+左键`自动寻址的话)，另外可能有些人也会觉得不太美观，因此可采取的将`src`将入 node 的引入路径下，具体配置如下：

```js
const path = require('path');
const { override } = require('customize-cra');

const overrideProcessEnv = () => config => {
  config.resolve.modules = [path.join(__dirname, 'src')].concat(
    config.resolve.modules
  );
  return config;
};

module.exports = override(overrideProcessEnv());
```

## autocompelete

在开发 react 表单的时候经常会遇到一个很麻烦的事情，那就是在我不需要的地方，浏览器也给我自动填充了用户名和密码，并且设置`autoComplete="off"`无效。

可采用如下方式进行最简单的处理：

```jsx
<Input autoComplete="new-password" type="password" placeholder="Password" />
```

## 路由用法

match 即为上级路由，在使用此路由时传入，此种路由不同于 hash 路由，不存在`#`。

```jsx
const App = ({ match }) => (
  <div>
    <Switch>
      <Route
        path={`${match.url}ChatRoom`}
        component={asyncComponent(() => import('./Chat'))}
      />
      <Route
        path={match.url}
        render={props => (
          <Redirect
            to={{
              pathname: '/ChatRoom',
              state: { from: props.location }
            }}
          />
        )}
      />
    </Switch>
  </div>
);
```

## 动态加载路由组件

在上面路由用法中使用的`asyncComponent`为动态的加载组件，避免首页加载时间过长，可用此种方式进行异步加载。

实现代码如下：

```jsx
import React, { Component } from 'react';
import Nprogress from 'nprogress';
import ReactPlaceholder from 'react-placeholder';
import 'nprogress/nprogress.css';

import 'react-placeholder/lib/reactPlaceholder.css';
import CircularProgress from 'components/CircularProgress';

export default function asyncComponent(importComponent) {
  class AsyncFunc extends Component {
    constructor(props) {
      super(props);
      this.state = {
        component: null
      };
    }

    componentWillMount() {
      Nprogress.start();
    }

    componentWillUnmount() {
      this.mounted = false;
    }

    async componentDidMount() {
      this.mounted = true;
      const { default: Component } = await importComponent();
      Nprogress.done();
      if (this.mounted) {
        this.setState({
          component: <Component {...this.props} />
        });
      }
    }

    render() {
      const Component = this.state.component || <CircularProgress />;
      return (
        <ReactPlaceholder type="text" rows={7} ready={Component !== null}>
          {Component}
        </ReactPlaceholder>
      );
    }
  }

  return AsyncFunc;
}
```

## 登录认证

react 中实现根据 token 进行登录认证，也就是简单的判断 token 是否存在，如果不存在则跳转登录页。

```jsx
// 路由重定向
const RestrictedRoute = ({ component: Component, token, ...rest }) => (
  <Route
    {...rest}
    render={props =>
      token ? (
        <Component {...props} />
      ) : (
        <Redirect
          to={{
            pathname: '/signin',
            state: { from: props.location }
          }}
        />
      )
    }
  />
);

class App extends Component {
  render() {
    const { match, token } = this.props;
    return (
      <LocaleProvider locale={antdCN}>
        <Switch>
          <Route exact path="/signin" component={() => <div>登录</div>} />
          <RestrictedRoute
            path={`${match.url}`}
            token={token}
            component={() => <div>sd</div>}
          />
        </Switch>
      </LocaleProvider>
    );
  }
}
```
