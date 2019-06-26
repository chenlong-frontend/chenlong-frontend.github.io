---
title: prisma、jwt入门笔记
author: 陈龙
date: 2019-06-26 14:40:20
tags: [prisma]
categories: [prisma]
---

最近利用空闲时间学习了一下 prisma 简单使用，对于我这种前端开发者确实相当方便。

## prisma

在 docker 中使用 prisma 相当方便。安装流程可以看[这里](https://prisma.1wire.com/blog/newdatabase)。

这里就我当初的初识 prisma 时的疑惑谈一谈。

### 启动失败

一开始因为数据库配置一直无法启动服务。在执行`prisma deploy`报如下错：

```shell
could not connect to server at http://localhost:4466.Please check if your server is running.
```

只是看这个我当时也不知道错在哪里，后来通过查看日志找到了问题所在。

首先执行`docker-compose ps`查看容器名称，再执行`docker logs --tail 50 --follow --timestamps prismademo_prisma_1`查看容器的报错，其中`prismademo_prisma_1`是你要查看的容器名称。

解决了数据库问题，接着启动又遇到一个报错：

```shell
bad indentation of a mapping entry at line 3, column 14:
- generator: javascript-client
```

因为`prisma.yml`中的`- generator: javascript-client`少了个缩进。。。，加上一个缩进空格即可。

### prisma-client

关于这个生成的客户端的方式，我尝试过几种，最终选择了 rest 风格方式。具体使用方式可见我写的一个简单的[例子](https://github.com/1016482011/prisma-rest)。

其中`src/prisma-client`下的两个文件便是由`prisma generate`生成，复制到你的 node 中间层即可。

其中有一点需要注意，如果你需要远程调用 prisma 的接口的话，需要修改`prisma-client`下的`index.js`文件：

```js
// ...
exports.Prisma = prisma_lib_1.makePrismaClientClass({
  typeDefs,
  models,
  // 此处将localhost改为你的服务器地址
  endpoint: `http://xxxxxxx:4466`
})
// ...
```

### 列表的联表查询

默认查询只能查出自身的字段，如果需要他表关联数据或者需要简化查询返回字段，可以通过如下方式:

```js
const query = `
  query {
    webAddBalances{
      id,
      Amount,
      Createtime,
      User{realname}
    }
  }
  `
const result = await prisma.$graphql(query)
```

## jwt

jwt 使用相对来说简单一些，具体可见[这里](https://github.com/auth0/node-jsonwebtoken#readme)查看具体使用方式：

使用的话也相当简单

```js
const jwt = require('jsonwebtoken')
class Jwt {
  //生成token
  generateToken(data) {
    let created = Math.floor(Date.now() / 1000)
    let token = jwt.sign(
      {
        data,
        exp: created + 60 * 300
      },
      'secret'
    )
    return token
  }

  // 校验token
  verifyToken(t) {
    let res
    try {
      const token = t.replace('Bearer ', '')
      let result = jwt.verify(token, 'secret') || {}
      let { exp = 0 } = result,
        current = Math.floor(Date.now() / 1000)
      if (current <= exp) {
        res = result.data || {}
      }
    } catch (e) {
      res = e
    }
    return res
  }
}

module.exports = Jwt
```

总的来说，通过这次学习，让我有了快速搭建小型项目的能力，但这些基础还远远不够，后续我也会尽量寻找一些练手的小玩意来深入学习。
