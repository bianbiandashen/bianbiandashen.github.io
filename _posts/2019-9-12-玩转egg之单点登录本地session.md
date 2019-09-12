---
layout:     post
title:      玩转egg之本地session
subtitle:   
date:       2019-9-11
author:    	BIANBIAN
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - Egg
    - node
    - JS
---


## 1.1 关于Session 为什么要有Session

HTTP是无状态的协议，但是在很多情况下服务端需要记录用户的状态，典型的场景比如用户登录、购物车等。所以服务端需要某种机制来识别具体的用户，这种机制就是Session。

当用户购物时，服务端并不知道是那个用户操作的，所以需要为特定的用户创建特定的Session，来标识、跟踪这个用户，才能弄清楚用户购物车内的东西并且不与其他用户的购物车混淆。

## 1.2 Session的保存

Session是保存在服务端的，有一个唯一的标识。服务端保存Session的方法很多，内存、数据库、文件都有。大型的网站一般会有专门的Session服务器集群，用来保存用户回话，这个时候Session是保存在内存的。

## 1.3 Session如何是识别用户身份

Session是使用Cookie来识别用户身份的。每次HTTP请求时，客户端会发送响应的Cookie到服务器。

第一次创建Session的时候，服务端会在HTTP响应中告诉客户端，需要在Cookie里面记录一个Session ID，以后每次请求都需要把这个ID发送到服务器，服务端就知道是哪个用户发送的请求了。

这个Session ID是维持Session回话的客户端的唯一标识，是Session实现的核心。

```javascript
如果浏览器禁用了Cookie，Session ID需要通过url来传递。
```

## 1.4 与cookie的区别
Session是存储在服务端的数据，用来跟踪是被用户的状态，可以保存在集群、数据库、内存、文件中

Cookie是客户端保存信息的一种机制，用来记录用户的一些信息，会随着网络请求发送给服务端，也是实现Session的一种方式

其他的实现方式
除了使用Session实现用户身份的识别，也可以使用JWT的机制实现，详情可以参考之前的笔记《JS42 JWT实现单点登录》

Egg中对Session的处理
Egg内置了egg-session插件，可以直接使用ctx.session访问或者修改当前用户的Session



```javascript
class HomeController extends Controller {
  async fetchPosts() {
    const ctx = this.ctx;
    
    // 获取 Session 上的内容
    const userId = ctx.session.userId;
    const posts = await ctx.service.post.fetch(userId);
    
    // 修改 Session 的值
    ctx.session.visited = ctx.session.visited ? (ctx.session.visited + 1) : 1;
    
    ctx.body = {
      success: true,
      posts,
    };
  }
}
```


## Session可以直接读取或者修改，删除的话直接置为null就可以了：
```swift
ctx.session = null;
```


## 需要特别注意的是
设置Session属性时要避免以_开头，并且不能为isNew这个属性，会造成字段丢失。

Session的实现是基于Cookie的，默认配置下，用户Session的内容加密后直接存储在Cookie的一个字段中：



## 实现一套基于jwt的sso单点登录系统
egg框架



```
├── package.json
├── app
|   ├── router.js
│   ├── controller
│   |   └── user.js
│   ├── service
│   |   └── user.js
│   ├── middleware
│   |   └── checkToken.js
│   └── model
│       └── user.js
├── config
|   ├── plugin.js
|   ├── config.default.js
```
以上就是框架约定的目录，由于我们前端分离，所以view目录也不需要了:

app/router.js 用于配置 URL 路由规则。
app/controller/** 用于解析用户的输入，处理后返回相应的结果。
app/service/** 用于编写业务逻辑层。
app/middleware/** 用于编写中间件。
config/config.{env}.js 用于编写配置文件。
config/plugin.js 用于配置需要加载的插件。


##tips
修改cors
修改plugin.js


```javascript
exports.cors = {
    enable: true,
    package: 'egg-cors',
}

```

##在egg.js中使用redis
安装请看redis在mac下的安装,其他操作系统请根据口味自行百度。

使用请参考egg-redis文档

##修改plugin.js
```javascript
exports.redis = {
    enable: true,
    package: 'egg-redis',
}

```
##修改config.default.js
```javascript
exports.redis = {
    client: {
        port: [port],          
        host: '127.0.0.1',
        password: [password],
        db: 0
    }
}

```
##注意点
一定要改默认端口!一定要改默认端口!一定要改默认端口! 请看：Redis 未授权访问缺陷可轻易导致系统被黑，请修改redis.conf: requirepass(密码) , port(端口)
可能会遇到redis快照关闭导致无法写入数据的情况，会报错，类似:MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error. 解决方案是修改redis.conf: stop-writes-on-bgsave-error改为no


##在egg.js中使用postgresql
```swift
   yarn add egg-sequelize
    yarn add pg pg-hstore
```
##修改plugin.js
```javascript

exports.sequelize = {
    enable: true,
    package: 'egg-sequelize'
}
```
##修改config.default.js
```javascript
exports.sequelize = {
    dialect: 'postgres',
    database: 'postgres',
    host: 'localhost',
    port: '8888',
    username: 'postgres',
    password: '123456'
}

```
##生成token
```javascript
yarn add jsonwebtoken
```




```javascript

var jwt = require('jsonwebtoken');
var tokenKey = 'token key'
var token = jwt.sign({ foo: 'bar' }, tokenKey);


var decoded = jwt.verify(token, tokenKey);
console.log(decoded.foo) // bar

```


##写法
app/model/checkToken.js




```javascript

const { verify }  = require('jsonwebtoken')
const moment = require('moment')

module.exports = options => {
    return async function checkToken(ctx, next) {
        const { jwtKey } = ctx.app.config.appConfig
        const {request: { path, header: {token} }} = ctx
        const {exclude=[]} = options
        let decodedJwt = {}
        try {
            if (exclude.indexOf(path.replace('/', '')) === -1) { // 需要token的接口
                decodedJwt = verify(token, jwtKey)
                // token exp 超时
                if(moment().isAfter(decodedJwt.exp)) {
                    throw {
                        code: -340,
                        msg: 'token 过期'
                    }
                }
            } else { //不需要token的接口
                decodedJwt.exp = -1
            }
        } catch (error) {
            ctx.app.logger.error('token error', error)
            if (error.code) {
                ctx.body = error
            } else {
                ctx.body = {
                    code: -360,
                    msg: 'token 错误'
                }
            }
        }
        if (decodedJwt.exp) {
            await next()
        }
    }
}
```
##配置
修改config.default.js

```javascript
    exports.middleware = ['checkToken'] // 中间件会按顺序执行
    
    // 中间件需要的配置项，可以通过app.config[${middlewareName}]访问
    exports.checkToken = {
        exclude: ['login', 'signup']
    }

```
未完待续





- 参考 node.js in Alibaba Group