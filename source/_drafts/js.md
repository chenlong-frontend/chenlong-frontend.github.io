---
title: 流式加载数据
tags: [react]
categories: [react]
---

// 监听复制
document.addEventListener('copy', function(e){
e.clipboardData.setData('text/plain', 'Hello, world!');
e.clipboardData.setData('text/html', '<b>Hello, world!</b>');
e.preventDefault(); // We want our data, not data from any selection, to be written to the clipboard
});

// 手动触发复制
document.execCommand('copy')

// 代码注释中添加 TODO 来标识需改进片段

在写 sdk 工程中，为了不污染全局变量，在暴露的节点中使用 static 来写入一些 tool 工具

```ts
import {
  Tool,
  RotateTool,
  HideTool,
  ColorTool,
  ClipTool,
  DragTool,
  WireframeTool,
  PickTool,
  SaveView,
  Translate,
  MarkTool,
  Measure,
  Roam,
} from './tools';

class ModelTool extends EventDispatcher {
  static ColorTool = ColorTool;
  static ClipTool = ClipTool;
  static DragTool = DragTool;
  static HideTool = HideTool;
  static MarkTool = MarkTool;
  static Measure = Measure;
  static RotateTool = RotateTool;
  static Roam = Roam;
  static PickTool = PickTool;
  static SaveView = SaveView;
  static Translate = Translate;
  static WireframeTool = WireframeTool;
}

import { ModelTool as Bustard } from './modelTool';
export { Bustard };
这样外部可以这么使用;

import { Bustard } from 'bustard';

const t = new Bustard();
t.use(new Bustard.ColorTool());
```

防抖动（debounce ）和节流阀（throttle ） loadsh

// 事件多重触发时使用
e.stopPropagation()

## NodeJS 回调的错误处理方式及其优点

优点包括如下：

如果不需要引用数据，则无需对数据进行处理
API 保持高度的一致性可以带来更多的便捷
能够轻松适配回调模式，从而实现更易于维护的代码

加分回答
虽然这只是一种约定，但是我们应该按照约定的去做。
