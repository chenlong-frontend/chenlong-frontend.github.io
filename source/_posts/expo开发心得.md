---
title: expo开发心得
author: 陈龙
date: 2020-09-07 19:20:18
tags: [javascript, react-native, expo]
categories: [javascript]
---

最近自己用`uniapp`做小程序的时候，顿时就觉得刚工作是的`expo`的开发经验还是有点帮助的，这里重新温习当时做`expo`时的工作心得。

这个`expo`的小项目已上传至我的 [github](https://github.com/chenlong-frontend/expo-app)

## react native

#### 读取本地文件流

```js
const xhr = new XMLHttpRequest()
xhr.onload = function() {
  // 文件返回值
  console.log(xhr.response)
}
xhr.onerror = function() {
  console.log(new TypeError('Network request failed'))
}
xhr.responseType = 'blob'
xhr.open('GET', uri, true)
xhr.send(null)
```

#### 点击反馈

由于兼容性问题，需要判断平台使用哪种点击反馈

安卓中使用 `TouchableNativeFeedback` 组件

IOS 中使用 `TouchableOpacity` 组件

#### 关于 flatList 的和 reflash 的使用

之前在使用 flatList 时，按照文档的说明，在 flatList 中配置下拉刷新的行为，但出现一个问题，当数据为空时，将无法进行刷新，此种情况可以通过设置 flatList 的 ListEmptyComponent 来保证空白数据可刷新，或者在父级组件中使用刷新组件(前提是父级组件有高度)

#### 关于修改问题后必须清空数据重新加载问题

首页的图片并不是随着列表一并返回到前端的，而是根据 fileToken 进行二次查询的，如果在不清空数据的情况下去获取数据，会出现编辑时添加的图片无法得到及时的更新，因为编辑问题是 fileToken 不会变，列表引用的展示图片的子组件也不会发生更新，也就不会去请求新的图片，目前采用的是清空数据

## Expo

#### 使用流程

1. 注册 expo 账号，使用 expo-cli 创键项目，安装依赖，通过 `expo start` 运行项目、
2. 手机调至开发者模式，连接电脑，点击在模拟器中打开，会 cli 工作会自动安装 expo app 并打开项目
3. 开发完成时，通过 `expo build:android` 或 `expo build:ios`打包
4. 关于发布，如果使用 OTA 更新，只需打包一次，之后只需发布即可，应用会自动从云端更新
5. 打包后会生成一个二维码，可以在 expo app 中直接打开

#### OTA 配置

在 app.json 中加入以下代码即可

```json
"updates": {
    "enabled": true,
    "checkAutomatically": "ON_LOAD",
    "fallbackToCacheTimeout": 30000
}
```

#### 配置 OTA 获取资源路径

1. 通过执行例如 `expo export --public-url https://raw.githubusercontent.com/1016482011/bager-mobile/master`指令导出 js 和 html 的静态文件
2. 将导出的文件上传到你上一步所指定的服务器目录下
3. 进行 app 在线打包 `https://raw.githubusercontent.com/1016482011/bager-mobile/master/android-index.json>`，此指令为安卓打包，ios 同理

#### 隐藏

expo 项目默认是开源公开的，如果不想公开在 app.json 里设置 `"privacy": "unlisted",`

#### 实用的资源站点

1. [经过删选评分的 react native 组件](http://native.directory/)
2. [expo 中的图标库](https://expo.github.io/vector-icons/)

#### 地理位置

地理位置的接口在国内使用问题：

1. 获取的经纬度与使用的高德地图使用的不是同一套标准
2. 目前已知的此接口在华为手机中无法使用

#### 地图

expo 中只支持谷歌地图，实践证明国内无法使用，如果确实需要地址，可以内嵌 webview 来使用 web 版的地图

#### ImagePicker

此功能的 `quality` 设置仅在裁剪时生效

#### 状态栏

状态栏在双平台下表现不一致，expo 在安卓下默认是全屏，即安卓从状态栏顶部开始布局，而 ios 从底部开始布局

鉴于此种情况建议使用 react-navigation 提供的 header，或者自行根据不同平台做适配

#### 资源

在引用资源时，发现问题为 html 文件，在开发环境中可以引用，但打包之后的文件中并没有 html 文件，目前的解决方案是将 html 文件放到公网上，然后通过网络加载的方式获取

## react-navigation

子路由无法清除父路由

#### 跳转首页和登录页

首先说明一下 app 中每跳往一个新的路由，app 都会打开一个新的窗口，原先页面的组件并不会销毁，当直接已打开的页面时，不会触发原先页面的任何生命周期，但可以通过 react-navigation 的事件进行监听，如果有必要的话

在提交完表单后，通过按下安卓的物理返回键不应该可以返回到提交页面，一般可以用 replace 解决，但首页和登录页在点击返回键时，应该直接退出应用，如果可以的话应该清空整个路由栈，然后进行跳转，实践证明清空路由栈的做法在子路由中无法执行，设计时应该避免此种情况，如果避免不了，应该在登录页和首页拦截返回事件，屏蔽默认事件并退出应用

#### 使用 navigation 提供的 header

如果 header 中存在和当前页面的联动操作时，可以使用自定义 header 组件并配合 redux 使用，例子如下

```js
// 注意此处的所有变量都是redux中的状态，problemListModelAction是redux的action
<Header
    title={proName}
    leftIcon={null}
    type={!listModel ? 'FontAwesome' : 'MaterialIcons'}
    rightIcon={!listModel ? 'map' : 'format-list-bulleted'}
    onRightPress={() => problemListModelAction()}
/>

static navigationOptions = {
    headerTitle: <ProListHeaderWarp />
}
```

#### 在组件外调用路由

如果存在在组件外调用路由的需求，比如在 redux 中进行跳转，可按如下进行配置

```js
// NavigationService.js
import { NavigationActions, StackActions } from 'react-navigation'
let _navigator
function setTopLevelNavigator(navigatorRef) {
  _navigator = navigatorRef
}
function navigate(routeName, params) {
  _navigator.dispatch(
    NavigationActions.navigate({
      routeName,
      params
    })
  )
}
export default {
  navigate,
  setTopLevelNavigator
}

// APP.js

<AppNavigator
    ref={navigatorRef => {
    NavigationService.setTopLevelNavigator(navigatorRef)
    }}
/>
```

之后便可以调用 NavigationService 中的 navigate 进行路由跳转了，但此方法进行的跳转路由无法去影响其他路由，例如 replace 和清空路由栈

#### 关于不实现拦截 tab 跳转需求问题

按照文档说明，在 tab 页面中可以获取到页面跳转的事件，但并没有得到当前页面 url 和目标页面 url，虽然文档里说可以得到，之后未作深入调查，或许使用自定义 tab 栏可以更好的实现

#### 修复大屏手机下标题栏顶部不同色块方法

使用了 react-navigation 的 header，并使用自定义 header 之后，在大屏手机上会出现 head 顶部有一条约 2px 的线，需要在配置 header 组件的同时设置 header 的背景色，具体如下：

```js
static navigationOptions = {
  headerTitle: <ProListHeaderWarp />,
  headerStyle: {
    backgroundColor: '#3781C6'
  }
}
```

此处的 `#3781C6` 会同时应用到安卓手机的状态栏上

## native base

在入口页面引入 root 组件，来使用 nativebase 的全部功能

#### 定制主题

[在线定制主题](https://nativebase.io/customizer/)

注意如果使用了定制主题，需要执行 `node node_modules/native-base/ejectTheme.js`，来或者所有的样式文件，并在入口中通过以下方式引入

```jsx
<StyleProvider style={getTheme(variables)}>
  <Root>{其他代码}</Root>
</StyleProvider>
```
