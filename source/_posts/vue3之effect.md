---
title: vue3 响应原理
author: 陈龙
date: 2021-03-09 18:35:35
tags: [vue3]
categories: [vue3]
---

注：以下说明基于 vue3 v3.0.7

`vue3` 的响应式模块位于 `reactivity` 目录下，我们以以下代码为例：

```ts
let test = reactive({
  count: 0
})
```

### track

首先找到 [reactive](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/reactive.ts#L85) 方法

```ts
export function  (target: object) {
  // 如果尝试观察一个只读的 proxy，返回 target 本身
  if (target && (target as Target)[ReactiveFlags.IS_READONLY]) {
    return target
  }
  return createReactiveObject(
    target,
    false,
    mutableHandlers,
    mutableCollectionHandlers
  )
}
```

接着移步到 `166` 行，找到 `createReactiveObject`

```ts
function createReactiveObject(
  target: Target,
  isReadonly: boolean,
  baseHandlers: ProxyHandler<any>,
  collectionHandlers: ProxyHandler<any>
) {
  // 如果被观察对象已经被代理了，直接返回
  // 例外: 调用 readonly()
  if (
    target[ReactiveFlags.RAW] &&
    !(isReadonly && target[ReactiveFlags.IS_REACTIVE])
  ) {
    return target
  }
  // 如果已经存在了，直接返回已存在的值
  const proxyMap = isReadonly ? readonlyMap : reactiveMap
  const existingProxy = proxyMap.get(target)
  if (existingProxy) {
    return existingProxy
  }
  // 只有白名单内的值可以被观察
  const targetType = getTargetType(target)
  if (targetType === TargetType.INVALID) {
    return target
  }
  // 创建代理
  // collectionHandlers 引用类型的代理
  // baseHandlers 基础类型的代理
  const proxy = new Proxy(
    target,
    targetType === TargetType.COLLECTION ? collectionHandlers : baseHandlers
  )
  // 缓存到全局
  proxyMap.set(target, proxy)
  // 返回代理
  return proxy
}
```

此处使用的 `baseHandlers` 由 `reactive` 可知，调用的是[createGetter](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/baseHandlers.ts#L75)

```ts
export const mutableHandlers: ProxyHandler<object> = {
  get,
  set,
  deleteProperty,
  has,
  ownKeys
}

const get = /*#__PURE__*/ createGetter()

function createGetter(isReadonly = false, shallow = false) {
  return function get(target: Target, key: string | symbol, receiver: object) {
    // ...

    // 数组操作
    const targetIsArray = isArray(target)

    if (!isReadonly && targetIsArray && hasOwn(arrayInstrumentations, key)) {
      return Reflect.get(arrayInstrumentations, key, receiver)
    }

    // 非数组操作
    const res = Reflect.get(target, key, receiver)

    // ...

    // 如果不是只读的，调用 track
    if (!isReadonly) {
      track(target, TrackOpTypes.GET, key)
    }

    // ...

    // ref 处理
    if (isRef(res)) {
      // ref unwrapping - does not apply for Array + integer key.
      const shouldUnwrap = !targetIsArray || !isIntegerKey(key)
      return shouldUnwrap ? res.value : res
    }

    // 对象处理，
    if (isObject(res)) {
      // 为了避免无效值的警告，以及环形引用，这里将返回值也通过 代理返回
      return isReadonly ? readonly(res) : reactive(res)
    }

    return res
  }
}

```

在处理数组的时候，调用了 `arrayInstrumentations`

```js
const arrayInstrumentations: Record<string, Function> = {}
// 拦截了 'includes', 'indexOf', 'lastIndexOf' 方法
;(['includes', 'indexOf', 'lastIndexOf'] as const).forEach(key => {
  const method = Array.prototype[key] as any
  arrayInstrumentations[key] = function(this: unknown[], ...args: unknown[]) {
    const arr = toRaw(this)
    for (let i = 0, l = this.length; i < l; i++) {
      track(arr, TrackOpTypes.GET, i + '')
    }
    // 先用初始参数运行
    const res = method.apply(arr, args)
    if (res === -1 || res === false) {
      // 如果不起作用，使用初始值再跑一遍
      return method.apply(arr, args.map(toRaw))
    } else {
      return res
    }
  }
})
// 避免数组长度变化导致的无限死循环 (#2137)
;(['push', 'pop', 'shift', 'unshift', 'splice'] as const).forEach(key => {
  const method = Array.prototype[key] as any
  arrayInstrumentations[key] = function(this: unknown[], ...args: unknown[]) {
    pauseTracking()
    const res = method.apply(this, args)
    resetTracking()
    return res
  }
})
```

下面着一下关键的 [track](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/effect.ts#L141)

```ts
export function track(target: object, type: TrackOpTypes, key: unknown) {
  // 调用时需要 shouldTrack 和 activeEffect 为真
  if (!shouldTrack || activeEffect === undefined) {
    return
  }
  // 从缓存中获取，如果不存在，则创建一个新map，并以target为key，空map为 value 赋值
  let depsMap = targetMap.get(target)
  if (!depsMap) {
    targetMap.set(target, (depsMap = new Map()))
  }
  // 从depsMap中获取指定key的值，这里的值一般存的副作用方法
  let dep = depsMap.get(key)
  // 如果不存在，创建一个新set，并赋值
  if (!dep) {
    depsMap.set(key, (dep = new Set()))
  }
  // 如果dep中没有当前的副作用方法，则添加
  if (!dep.has(activeEffect)) {
    dep.add(activeEffect)
    activeEffect.deps.push(dep)
  }
}
```

### effect

在 `track` 里提到了 `effect`，这里也来看一下，首先看一下器入口函数 [effect](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/effect.ts#L55)

```ts
export function effect<T = any>(
  fn: () => T,
  options: ReactiveEffectOptions = EMPTY_OBJ
): ReactiveEffect<T> {
  // 返回原函数
  if (isEffect(fn)) {
    fn = fn.raw
  }
  // 创建副作用函数
  const effect = createReactiveEffect(fn, options)
  if (!options.lazy) {
    effect()
  }
  return effect
}

let uid = 0

function createReactiveEffect<T = any>(
  fn: () => T,
  options: ReactiveEffectOptions
): ReactiveEffect<T> {
  const effect = function reactiveEffect(): unknown {
    if (!effect.active) {
      return options.scheduler ? undefined : fn()
    }
    // 如果当前effect没被
    if (!effectStack.includes(effect)) {
      // 清空 effect上的deps数据
      cleanup(effect)
      try {
        enableTracking()
        // 加入当前调用栈
        effectStack.push(effect)
        // 设置当前调用effect
        activeEffect = effect
        // 调用外部传入的方法，并返回其结果
        return fn()
      } finally {
        // 调用完弹出
        effectStack.pop()
        resetTracking()
        // 将当前调用栈最后一个设置为当前调用 effect
        activeEffect = effectStack[effectStack.length - 1]
      }
    }
  } as ReactiveEffect
  effect.id = uid++
  effect.allowRecurse = !!options.allowRecurse
  effect._isEffect = true
  effect.active = true
  effect.raw = fn
  effect.deps = []
  effect.options = options
  return effect
}

function cleanup(effect: ReactiveEffect) {
  const { deps } = effect
  if (deps.length) {
    for (let i = 0; i < deps.length; i++) {
      deps[i].delete(effect)
    }
    deps.length = 0
  }
}
```

### trigger

现在还剩最后一步，就是如何去触发，`track` 中存起来的 `effect`

`track` 是在 `get` 拦截中被调用的，相对的，`trigger` 则是在 `set` 拦截中被调用的。

找到 [createSetter](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/baseHandlers.ts#L132)

```ts
function createSetter(shallow = false) {
  return function set(
    target: object,
    key: string | symbol,
    value: unknown,
    receiver: object
  ): boolean {
    const oldValue = (target as any)[key]
    if (!shallow) {
      // 获取初始值
      value = toRaw(value)
      if (!isArray(target) && isRef(oldValue) && !isRef(value)) {
        oldValue.value = value
        return true
      }
    }

    const hadKey =
      isArray(target) && isIntegerKey(key)
        ? Number(key) < target.length
        : hasOwn(target, key)
    const result = Reflect.set(target, key, value, receiver)
    // don't trigger if target is something up in the prototype chain of original
    if (target === toRaw(receiver)) {
      if (!hadKey) {
        trigger(target, TriggerOpTypes.ADD, key, value)
      } else if (hasChanged(value, oldValue)) {
        trigger(target, TriggerOpTypes.SET, key, value, oldValue)
      }
    }
    return result
  }
}
```

这里在最后调用了关键的方法 [trigger](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/effect.ts#L167)

```ts
export function trigger(
  target: object,
  type: TriggerOpTypes,
  key?: unknown,
  newValue?: unknown,
  oldValue?: unknown,
  oldTarget?: Map<unknown, unknown> | Set<unknown>
) {
  const depsMap = targetMap.get(target)
  if (!depsMap) {
    // 没有被追踪
    return
  }

  const effects = new Set<ReactiveEffect>()
  const add = (effectsToAdd: Set<ReactiveEffect> | undefined) => {
    if (effectsToAdd) {
      effectsToAdd.forEach(effect => {
        if (effect !== activeEffect || effect.allowRecurse) {
          effects.add(effect)
        }
      })
    }
  }

  // 这里通过调用 add 方法，将 effect 加入到 effects中
  if (type === TriggerOpTypes.CLEAR) {
    depsMap.forEach(add)
  } else if (key === 'length' && isArray(target)) {
    depsMap.forEach((dep, key) => {
      if (key === 'length' || key >= (newValue as number)) {
        add(dep)
      }
    })
  } else {
    // ...
  }

  const run = (effect: ReactiveEffect) => {
    if (effect.options.scheduler) {
      effect.options.scheduler(effect)
    } else {
      effect()
    }
  }

  // 触发副作用
  effects.forEach(run)
}

```

`vue` 中的使用示例 [setupRenderEffect](https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/renderer.ts#L1389)

```ts
  const setupRenderEffect: SetupRenderEffectFn = (
    instance,
    initialVNode,
    container,
    anchor,
    parentSuspense,
    isSVG,
    optimized
  ) => {
    // 为渲染创建响应式 effect
    instance.update = effect(function componentEffect() {
      // ...
    }
    // ...
  }
```

总结：`vue` 在`update，watch，reactive`中调用 `effect`，通过 `Proxy` 劫持 `get` 通过 `track` 来收集 `effect`，然后在 `trigger` 中触发目标收集的所有 `effect`，在`effect` 中调用 `patch` 来触发渲染。


