---
title: SuperTinyCompiler
author: 陈龙
date: 2019-07-04 18:40:20
tags: [compiler]
categories: [compiler]
---

[原文](https://the-super-tiny-compiler.glitch.me/intro)

这里我们打算将一些类`lisp`函数编译成类`C`函数调用。如果你对这两者都不熟悉也没关系，转换关系如下：

|         | LISP-style             | C-style              |
| ------- | ---------------------- | -------------------- |
| 2+2     | (add 2 2)              | add(2,2)             |
| 4-2     | (subtract 4 2)         | subtract(4,2)        |
| 2+(4-2) | (add 2 (subtract 4 2)) | add(2,subtract(4,2)) |

大多数编译可以拆分为三步：解析、转换和代码生成。

解析正常可以分为两步：词法分析和语法分析。

词法分析将原始代码通过分词器分解成独立的符号对象。语法分析将其格式化为抽象语法树。

以如下代码为例：

```lisp
(add 2 (subtract 4 2))
```

经过分词后得到数据如下：

```js
;[
  { type: 'paren', value: '(' },
  { type: 'name', value: 'add' },
  { type: 'number', value: '2' },
  { type: 'paren', value: '(' },
  { type: 'name', value: 'subtract' },
  { type: 'number', value: '4' },
  { type: 'number', value: '2' },
  { type: 'paren', value: ')' },
  { type: 'paren', value: ')' }
]
```

抽象语法树格式如下：

```js
{
  type: 'Program',
  body: [{
    type: 'CallExpression',
    name: 'add',
    params: [{
      type: 'NumberLiteral',
      value: '2',
    }, {
      type: 'CallExpression',
      name: 'subtract',
      params: [{
        type: 'NumberLiteral',
        value: '4',
      }, {
        type: 'NumberLiteral',
        value: '2',
      }]
    }]
  }]
}
```

接下来是转换，这一步会使用 AST 的数据来进行转换。他可以是与 AST 一样的语言结构，也可以转换成完全另一个语言。这是使用的是与 AST 一样的语言。

接着就是遍历转换得到的节点，采用深度优先原则。

以上述 AST 为例，遍历顺序如下：

1. Program
2. CallExpression (add)
3. NumberLiteral (2)
4. CallExpression (subtract)
5. NumberLiteral (4)
6. NumberLiteral (2)

最后是代码生成，一般情况下都是使用 AST 的数据结构来生成最终代码

### 总结

通过了解编译步骤可以有助我了解`babel`，另一方面，在工作中，有时候我们能遇到这么一个需求，就是将指定数据结构转换成指定 dom 树来进行渲染，往后遇到类似需求时或许可以参照参照。
