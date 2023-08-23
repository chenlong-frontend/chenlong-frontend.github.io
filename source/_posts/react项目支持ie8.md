---
title: react支持ie8
author: 陈龙
date: 2023-03-05 13:02:26
tags: [react, ie]
categories: [react]
---

## 需求背景

公司的链接入会页面原来使用 `jquery` 开发的，产品有一个比较大的需求变动，便趁着这个机会对原来的页面进行了`react + ts` 的改造
这个页面主要是用来使用浏览器调起客户端用的，有一些用户群体来自医院，他们使用的机器系统比较老旧，可能存在`ie`低版本浏览器，所以需要支持到 `ie8`

## 代码实现

首先是主要的依赖库和 `babel` 设置，需要注意的是 `react` 版本不能升高，高版本 `react`无法兼容 `ie` 低版本浏览器
另外 `babel` 的 `@babel/preset-env` 设置项也需要注意

```json
{
  "scripts": {
    "start": "webpack serve --config ./config/webpack.dev.config.js --progress",
    "build": "webpack --config ./config/webpack.pro.config.js --progress"
  },
  "dependencies": {
    "@types/react": "^18.0.27",
    "@types/react-dom": "^16.4.0",
    "copy-webpack-plugin": "^11.0.0",
    "react": "^16.4.0",
    "react-dom": "^16.4.0",
    "typescript": "^4.9.5"
  },
  "devDependencies": {
    "@babel/core": "^7.20.12",
    "@babel/preset-env": "^7.20.2",
    "@babel/preset-react": "^7.18.6",
    "@babel/preset-typescript": "^7.18.6",
    "babel-loader": "^9.1.2",
    "babel-polyfill": "^6.26.0",
    "css-loader": "^6.7.3",
    "css-minimizer-webpack-plugin": "^4.2.2",
    "html-webpack-plugin": "^5.5.0",
    "mini-css-extract-plugin": "^2.7.2",
    "style-loader": "^3.3.1",
    "terser-webpack-plugin": "^5.3.6",
    "ts-loader": "^9.4.2",
    "webpack": "^5.75.0",
    "webpack-cli": "^5.0.1",
    "webpack-dev-server": "^4.11.1",
    "webpack-merge": "^5.8.0"
  },
  "babel": {
    "presets": [
      [
        "@babel/preset-env",
        {
          "useBuiltIns": "usage",
          "corejs": "3",
          "targets": {
            "browsers": ["last 5 versions", "ie >= 9"]
          }
        }
      ],
      "@babel/preset-typescript",
      "@babel/preset-react"
    ]
  }
}
```

`ts`设置项 `tsconfig.json`

```json
{
  "compilerOptions": {
    "outDir": "./dist/",
    "sourceMap": true,
    "noImplicitAny": false,
    "module": "es2020",
    "target": "es5",
    "jsx": "react-jsx",
    "allowJs": true,
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "suppressImplicitAnyIndexErrors": true
  }
}
```

`webpack` 的核心设置项：

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: {
    pc: ['babel-polyfill', path.resolve(__dirname, '../src/')], // 添加babel 垫片
  },
  output: {
    path: path.resolve(__dirname, '../build'),
    publicPath: '/',
  },
  target: ['web', 'es5'], // 编译结果支持到 es5
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: ['babel-loader'],
      },
      {
        test: /\.css$/i,
        use: ['style-loader', 'css-loader'],
      },
      {
        test: /\.(png|svg|jpg|jpeg|gif)$/i,
        type: 'asset',
      },
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '*', '.js', '.jsx'],
  },
  plugins: [
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: path.resolve(__dirname, '../public/index.html'),
      chunks: ['pc'],
    }),
  ],
};
```
