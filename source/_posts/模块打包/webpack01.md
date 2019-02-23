---
title: webpack -ts
author: 陈龙
tags: [webpack, rollup]
categories: [webpack]
date: 2019-02-22 8:43:00
---

最近遇到一个需要将 ts 文件打包成 js 的小问题，打包工具有很多，就我所知道的就有`gulp`、`rollup`、`webpack`，下面就大致说说我是如何通过阅读文档从零搭建一个简单的打包配置的。

## 安装 npm 包

按照模块功能才划分，可以分为下面几组：

1. webpack：`webpack`、`webpack-cli`、`@types/webpack`
2. typescript：`typescript`、`ts-node`、`ts-loader`、`@types/node`
3. babel：`babel-core`、`babel-loader`、`babel-preset-env`
4. plugin：`clean-webpack-plugin`、`uglifyjs-webpack-plugin`

1 是 webpck 的核心文件和 cli 工具以及类型声明，2 是 ts 编译器和加载器，3 是 babel 核心文件，4 是 webpack 清理打包文件夹以及代码压缩插件

## webpack.config.js

下面是具体配置代码，这里是只定义了生产环境。

```ts
const path = require('path')
const CleanWebpackPlugin = require('clean-webpack-plugin')
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')

const config = {
  // 标识是生产环境还是开发环境，或者都不是
  mode: 'production',
  // 入口文件设置
  entry: {
    modelTool: './src/modelTool.ts'
  },
  // 控制是否生成source map来增强调试
  devtool: 'none',
  // 是否失败之后中断并抛出错误
  bail: true,
  // 打包输出设置
  output: {
    filename: '[name].min.js',
    path: path.resolve(__dirname, 'dist'),
    // var 变量名
    library: 'ModelTool'
  },
  resolve: {
    // 自动解析确定的扩展
    extensions: ['.ts', '.js']
  },
  module: {
    // ts配置
    rules: [
      {
        test: /\.ts?$/,
        use: 'ts-loader',
        exclude: /node_modules/
      },
      // babel配置
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
  // 清理dist文件夹
  plugins: [new CleanWebpackPlugin('dist/*')],
  optimization: {
    // 使用uglifyjs压缩js体积
    minimizer: [new UglifyJsPlugin()]
  }
}

module.exports = config
```

## package.json

在`package.json`中加入如下快捷启动方式，`watch`用来监听文件变化，build 用来打包

```json
{
  "scripts": {
    "watch": "webpack --watch --progress",
    "build": "webpack --config webpack.config.js"
  }
}
```

## rollup

另外我也用尝试着使用 rollup 进行对比，rollup 比起 webpack 较为简单，打包速度也比较快，大致配置如下：

```js
// rollup.config.js
import babel from 'rollup-plugin-babel'
import resolve from 'rollup-plugin-node-resolve'
import typescript from 'rollup-plugin-typescript'

const license = `/*!
 * Released under the MIT License.
 */`

export default {
  input: 'src/modelTool.ts',
  output: [
    {
      // 输出js格式
      format: 'umd',
      // 全局变量名
      name: 'ModelTool',
      // 输出文件地址
      file: 'dist/xx.js',
      // 输出文件首行显示文字
      banner: license,
      // 全局变量与external映射
      globals: {
        lodash: '_'
      }
    },
    {
      format: 'es',
      file: 'dist/xx.module.js',
      banner: license,
      globals: {
        lodash: '_'
      }
    }
  ],
  plugins: [
    resolve({
      // 将自定义选项传递给解析插件
      customResolveOptions: {
        moduleDirectory: 'node_modules'
      }
    }),
    // babel插件
    babel(),
    // ts插件
    typescript({ lib: ['es5', 'es6', 'dom'], target: 'es5' })
  ],
  // 外部依赖的名称
  external: ['lodash']
}
```

babel 在 `package.json` 中的配置

```json
// package.json
{
  "babel": {
    "presets": [
      [
        "@babel/preset-env",
        {
          "useBuiltIns": false
        }
      ]
    ]
  }
}
```

当前官方提供的 ts 插件`rollup-plugin-typescript`不会做类型检查以及`threejs`的一些插件引用问题，故而没有采用 rollup

rollup 打包出来的文件默认是未压缩的，这里采用的是 `terser`进行的压缩，只需在 `package.josn` 中进行如下设置即可：

```json
{
  "scripts": {
    "build": "rollup --config && terser dist/xxx.js -o dist/xxx.min.js"
  }
}
```
