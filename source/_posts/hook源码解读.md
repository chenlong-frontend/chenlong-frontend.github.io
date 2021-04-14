---
title: hook
author: 陈龙
date: 2021-04-14 17:41:58
tags: [react, hook]
categories: [react]
---

本篇源码解读基于 [preact](https://github.com/preactjs/preact)，版本号为 10.5.13

主要涉及的`API`为: `useState`, `useReducer`, `useEffect`

## useState

[useState](https://github.com/preactjs/preact/blob/master/hooks/src/index.js#L125) 声明如下：

```js
export function useState(initialState) {
	currentHook = 1;
	return useReducer(invokeOrReturn, initialState);
}
```

接着看 [useReducer](https://github.com/preactjs/preact/blob/master/hooks/src/index.js#L136):

根据官方文档说明，这是一个 `useState` 的替代方案，第一个参数`reducer`格式为 `(state, action) => newState`，并且返回当前的 `state` 以及与其配套的 `dispatch` 方法。类似于 `redux`

```js
export function useReducer(reducer, initialState, init) {
  // 获取当前hook状态值
	const hookState = getHookState(currentIndex++, 2);

  /**
  * reducer 函数如下
  * function invokeOrReturn(arg, f) {
  *  return typeof f == 'function' ? f(arg) : f;
  * }
  **/
	hookState._reducer = reducer;
  // 如果hook是第一次创建
	if (!hookState._component) {
		hookState._value = [
      // 初始值
			!init ? invokeOrReturn(undefined, initialState) : init(initialState),

      // dispatch
			action => {
        // 相当于redux里的reducer，得出计算之后的新值，如果是useState的情况，直接返回新值
				const nextValue = hookState._reducer(hookState._value[0], action);
        // 对比新旧值是否发生了变化，如果发生了变化会重新赋值
				if (hookState._value[0] !== nextValue) {
					hookState._value = [nextValue, hookState._value[1]];
					hookState._component.setState({});
				}
			}
		];
    // hookState的_component设置为 当前组件 currentComponent
		hookState._component = currentComponent;
	}
  // 返回当前的 state 以及与其配套的 dispatch 方法
	return hookState._value;
}

function getHookState(index, type) {

	if (options._hook) {
		options._hook(currentComponent, index, currentHook || type);
	}

	currentHook = 0;

  // 如果当前组件存在hooks，返回组件当前hooks值
  // 如果不存在，则创建一个包含 _list 和 _pendingEffects属性的对象，并赋值给 currentComponent的 __hooks属性
	const hooks =
		currentComponent.__hooks ||
		(currentComponent.__hooks = {
			_list: [],
			_pendingEffects: []
		});
  // 创建一个新hook状态对象，压入hooks的数组，并且将其返回
	if (index >= hooks._list.length) {
		hooks._list.push({});
	}
  // 否则返回原有的hook状态值
	return hooks._list[index];
}
```

## useEffect

[useEffect](https://github.com/preactjs/preact/blob/master/hooks/src/index.js#L163) 声明如下：

```js
export function useEffect(callback, args) {

	const state = getHookState(currentIndex++, 3);
  // 对比指定值是否发生变化
	if (!options._skipEffects && argsChanged(state._args, args)) {
    // 将回调函数赋给 _value
		state._value = callback;
    // 如果传入了要监听的值，此处便会赋给 _args，下次数据发生变化后会执行
		state._args = args;
    // 加入待处理任务
		currentComponent.__hooks._pendingEffects.push(state);
	}
}
```