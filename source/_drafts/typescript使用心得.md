---
title: typescript使用心得
tags:
---

## 工程创建

`yarn create react-app my-app --typescript`

compilerOptions 中设置 "declaration": true，执行 tsc 可生产 d.ts ,详细配置可见[http://www.typescriptlang.org/docs/handbook/compiler-options-in-msbuild.html](http://www.typescriptlang.org/docs/handbook/compiler-options-in-msbuild.html)

## wrbpack 打包

在使用 webpack 打包时，在使用 watch 功能时，遇到了一直编译的问题，经排查发现是"declaration": true 和 ”outDir": "./lib" 导致的。
可将这两个配置从 tsconfig 中删除，在控制台添加此两个选项。tsc --declaration true --outDir ./lib

## type 定义

type TooltipTrigger = 'hover' | 'focus' | 'click' | 'contextMenu';

当一个组件 props 中有一个值的类型为 TooltipTrigger 时，比如 trigger，
在父组件上调用此组件时打出 trigger 即可提示出上述几个
