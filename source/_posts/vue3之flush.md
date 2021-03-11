---
title: vue3之flush
author: 陈龙
date: 2021-03-10 17:38:46
tags: [vue3]
categories: [vue3]
---

注：以下说明基于 `vue3 v3.0.7`

以下代码，来自于 [scheduler](https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/scheduler.ts)

```js
// 是否正在刷新
let isFlushing = false
// 是否等待刷新，true表示正在等待刷新
let isFlushPending = false
// 当前刷新队列指针位置
let flushIndex = 0

// 刷新队列
const queue: SchedulerJob[] = []
```

外部调用一般为 [queueJob](https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/scheduler.ts#L79)

```js
export function queueJob(job: SchedulerJob) {
  if (
    (!queue.length ||
      !queue.includes(
        job,
        isFlushing && job.allowRecurse ? flushIndex + 1 : flushIndex
      )) &&
    job !== currentPreFlushParentJob
  ) {

    // 查看这个任务是否
    const pos = findInsertionIndex(job)
    if (pos > -1) {
      // 如果队列中已经存在此任务，则插在此任务后面
      queue.splice(pos, 0, job)
    } else {
      // 压入队列
      queue.push(job)
    }
    queueFlush()
  }
}
```

`queueJob` 的主要作用就是将传入的任务，压入队列。任务压入完毕之后调用了 `queueFlush`。

```js
function queueFlush() {
  if (!isFlushing && !isFlushPending) {
    // 标识为正在等待刷新
    isFlushPending = true
    // 设置当前执行的任务
    currentFlushPromise = resolvedPromise.then(flushJobs)
  }
}
```

`queueFlush` 设置了当前等待刷新状态，表明当前有任务正在等待刷新。其后调用了 `flushJobs`

```js
function flushJobs(seen?: CountMap) {
  // 转换状态，设置正在刷新状态
  isFlushPending = false
  isFlushing = true

  flushPreFlushCbs(seen)

  // 再刷新前给队列排序
  // 确保：
  // 1. 组件是按照父节点到子节点顺序更新
  //（因为父节点总是在子节点之前被创建，这样父节点的 effect 的优先权会更小）
  // 2. 如果一个组件在父组件更新的时候被卸载了，这个组件的更新操作将会被跳过
  queue.sort((a, b) => getId(a) - getId(b))

  try {
    // 依次执行队列里的job
    for (flushIndex = 0; flushIndex < queue.length; flushIndex++) {
      const job = queue[flushIndex]
      if (job) {
        callWithErrorHandling(job, null, ErrorCodes.SCHEDULER)
      }
    }
  } finally {
    // 指正归零
    flushIndex = 0
    // 队列清零
    queue.length = 0

    flushPostFlushCbs(seen)

    // 结束刷新
    isFlushing = false
    currentFlushPromise = null
    // 在队列清空前持续刷新
    if (queue.length || pendingPostFlushCbs.length) {
      flushJobs(seen)
    }
  }
}
```

总结：`vue` 异步刷新的原理在于，外部的 `job` 持续进来之后，如果会持续压入队列

如果当前没有任务在等待或者没有正在任务在刷新，就会发起一个微任务，将任务设置为等待状态

如果当前有刷新正在进行，这个`job`也会顺带着一起处理

由于任务是放在微任务队列里的，所以是异步的


