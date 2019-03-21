---
title: EventDispatcher
author: 陈龙
date: 2019-03-20 22:11:50
tags: [javascript]
categories: [javascript]
---

最近在[git](https://github.com/mrdoob/eventdispatcher.js/)上看到一个事件分发的小插件，个人觉得非常有用。

源代码如下：

```ts
export class EventDispatcher {
  constructor() {
    this._listeners = {}
  }
  private _listeners: { [type: string]: Array<(event: Event) => void> }
  addEventListener(type, listener) {
    const listeners = this._listeners
    if (listeners[type] === undefined) {
      listeners[type] = []
    }

    if (listeners[type].indexOf(listener) === -1) {
      listeners[type].push(listener)
    }
  }

  hasEventListener(type, listener) {
    const listeners = this._listeners

    return (
      listeners[type] !== undefined && listeners[type].indexOf(listener) !== -1
    )
  }

  removeEventListener(type, listener) {
    const listeners = this._listeners
    const listenerArray = listeners[type]

    if (listenerArray !== undefined) {
      const index = listenerArray.indexOf(listener)

      if (index !== -1) {
        listenerArray.splice(index, 1)
      }
    }
  }

  dispatchEvent(event) {
    const listeners = this._listeners
    const listenerArray = listeners[event.type]

    if (listenerArray !== undefined) {
      event.target = this

      const array = listenerArray.slice(0)

      for (let i = 0, l = array.length; i < l; i++) {
        array[i].call(this, event)
      }
    }
  }
}
```

源代码实现的很精简

- `addEventListener`：绑定事件，`type`为事件类型，`listener`为事件回调函数。
- `hasEventListener`：检查指定类型的回调函数是否存在。
- `removeEventListener`：删除指定事件类型下的函数。
- `dispatchEvent`：触发指定类型的事件，`event`为传给事件回调函数的参数。

其用法也比较简单，如下是实际项目中使用代码：

```ts
import { Measure } from './tools'
class ModelTool extends EventDispatcher {
  static Measure = Measure

  constructor(el: HTMLDivElement | null) {
    super()
    this.core = new Core(el)
    this.eventBind()
  }

  public eventBind() {
    this.core.el.addEventListener(
      'click',
      e => {
        const intersection = this.core.getFirstIntersection(
          e.clientX,
          e.clientY
        )
        this.dispatchEvent({ type: 'click', event: e, intersection })
      },
      false
    )
  }
  private core: Core

  public use = <T extends Tool>(tool: T): T => {
    tool.core = this.core
    if (!_.isNull(tool.action)) this.addEventListener('click', tool.action)

    return tool
  }

  public remove = (tool: Tool) => {
    if (!_.isNull(tool.action)) this.removeEventListener('click', tool.action)
    tool.core = null
  }
}
```

在`eventBind`函数中，将`dom(el)`的事件进行了绑定，并在点击时分发类型为`click`的函数。

`use`函数是一个用来初始化插件的函数，并且会自动将插件内的函数绑定到点击事件之下。

`remove`是一个清除插件的函数，将插件的点击事件注销。

假设`Measure`函数如下：

```ts
export class Measure {
  public core: Core | null = null

  public action: (e: ClickEvent) => void = e => {
    console.log(e.intersection)
  }
}
```

使用如下：

```js
const modeltool = new ModelTool(document.getElementById('container'))
modelTool.use(new ModelTool.Measure())
```

点击 id 为`container`的元素是会打印出`getFirstIntersection`方法返回的结果。

当然这只是`EventDispatcher`一种使用情景，你也可以使用它来为一个`class`或者函数制定自定义函数等等。在封装一些组件中非常好用。
