## 事件

```js
import { Editor, getDefaultKeyBinding, KeyBindingUtil } from 'draft-js'
const { hasCommandModifier } = KeyBindingUtil

function myKeyBindingFn(e) {
  // 如果是ctrl+s，则返回'myeditor-save'
  if (e.keyCode === 83 /* `S` key */ && hasCommandModifier(e)) {
    return 'myeditor-save'
  }
  return getDefaultKeyBinding(e)
}

handleKeyCommand = command => {
  console.log(command)
  if (command === 'myeditor-save') {
    // 这里可以保存你的内容，设置状态等等
    return 'handled'
  }
  return 'not-handled'
}
;<Editor
  editorState={this.state.editorState}
  // 处理指令
  handleKeyCommand={this.handleKeyCommand}
  ref="editor"
  // 处理按键
  keyBindingFn={myKeyBindingFn}
/>
```

```js
editorState.getCurrentContent()就是contentState
```
