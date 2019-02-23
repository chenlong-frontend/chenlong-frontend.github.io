---
title: webpack -git
author: 陈龙
date: 2019-02-23 16:58:03
tags: [webpack, git]
categories: [webpack]
---

继上次 webpack 打包配置完成之后，后续针对使用过程中遇到的问题又进行了些许修改

因为此次工作的目标是在 git 上发布，所以一个工具使用的例子就必不可少了，经过几次修改后，文件夹结构如下

```js
project
├── dist
|   ├── xx.js
|   └── xx.min.js
├── example
|   ├── css
|   ├── js
|   └── *.html
├── src
|   └── *.ts
├── package.json
├── tsconfig.json
├── webpack.dev.js
└── webpack.prod.js
```

`dist`用来存放打包之后的文件，`example`是使用样例，`src`是工具源码，`webpack.dev.js`开发配置，`webpack.prod.js`生成配置

首先是`example`文件夹下都是一些简单的`html`文件，其会直接引用`dist`文件夹生成的代码

```html
<script src="../dist/xx.js"></script>
```

两个 webpack 配置基本一致，下面是`webpack.dev.js`的配置：

```js
const path = require('path')
const CleanWebpackPlugin = require('clean-webpack-plugin')
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')

const config = {
  mode: 'development',
  entry: {
    modelTool: './src/index.ts'
  },
  devtool: 'source-map',
  bail: true,
  output: {
    filename: '[name].js',
    path: path.resolve(__dirname, 'dist'),
    libraryTarget: 'this'
  },
  resolve: {
    extensions: ['.ts', '.js']
  },
  module: {
    rules: [
      {
        test: /\.ts?$/,
        use: 'ts-loader',
        exclude: /node_modules/
      },
      {
        test: /\.js$/,
        exclude: /(node_modules|bower_components)/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env'],
            plugins: ['@babel/transform-runtime']
          }
        }
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(['dist/modelTool.js', 'dist/modelTool.js.map'])
  ],
  optimization: {
    minimizer: [new UglifyJsPlugin()]
  }
}

module.exports = config
```

`webpack.prod.js` 不同点在于 `mode`为`production`，`devtool`为`none`，`output`中的`filename`为`[name].min.js`，其他的都是一致的。

添加 example 的目的一是可以在开发的时候测一下功能是否正常，二是传到 git 上后可以作为使用示例。
