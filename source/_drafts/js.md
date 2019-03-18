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
  Roam
} from './tools'

class ModelTool extends EventDispatcher {
  static ColorTool = ColorTool
  static ClipTool = ClipTool
  static DragTool = DragTool
  static HideTool = HideTool
  static MarkTool = MarkTool
  static Measure = Measure
  static RotateTool = RotateTool
  static Roam = Roam
  static PickTool = PickTool
  static SaveView = SaveView
  static Translate = Translate
  static WireframeTool = WireframeTool
}

import { ModelTool as Bustard } from './modelTool'
export { Bustard }
这样外部可以这么使用

import { Bustard } from 'bustard'

const t = new Bustard()
t.use(new Bustard.ColorTool())
```
