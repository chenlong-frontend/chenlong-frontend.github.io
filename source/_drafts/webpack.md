---
title: webpack -ts
author: 陈龙
tags: [typescript, webpack]
categories: [webpack]
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
    // 加载文件
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
