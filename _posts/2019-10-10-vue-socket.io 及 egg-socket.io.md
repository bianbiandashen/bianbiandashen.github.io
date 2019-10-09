---
layout:     post
title:      vue-socket.io 及 egg-socket.io
subtitle:   
date:       2019-10-10
author:    	BIANBIAN
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - egg
---


## 1.1 egg-socket.io 的使用
![](https://img2018.cnblogs.com/blog/1479789/201901/1479789-20190102164453381-1492535421.png)




## 2 serve

#开启插件以及插件配置
```javascript
cnpm install --save egg-socket.io

exports.io = {
  enable: true,
  package: 'egg-socket.io'
};


```
```javascript
// app/config/config.default.js
// 这里的 auth 以及 filter 是待会会编写的两个中间件，用于不用依据自己的情况选择即可
exports.io = {
   namespace: {
       '/': {
           connectionMiddleware: [ 'auth' ],
           packetMiddleware: [ 'filter' ],
       }
   }
};
```


编写中间件
```javascript
// app/io/middlewware/auth.js
// 这个中间件的作用是提示用户连接与断开的，连接成功的消息发送到客户端，断开连接的消息在服务端打印
module.exports = app => {
    return function* (next) {
        this.socket.emit('res', 'connected!');
        yield* next;
        console.log('disconnection!');
    };
};

// app/io/middleware/filter.js
// 这个中间件的作用是将接收到的数据再发送给客户端
module.exports = app => {
    return function* (next) {
        this.socket.emit('res', 'packet received!');
        console.log('packet:', this.packet);
        yield* next;
    };
};

```
编写控制器

```javascript
// app/io/controller/chat.js
// 将收到的消息发送给客户端
module.exports = app => {
  return function* () {
    const self = this;
    const message = this.args[0];
    console.log('chat 控制器打印', message);
    this.socket.emit('res', `Hi! I've got your message: ${message}`);
  };
};

```
编写路由

```javascript
// app/router.js
// 这里表示对于监听到的 chat 事件，将由 app/io/controller/chat.js 处理
module.exports = app => {
  app.io.of('/').route('chat', app.io.controllers.chat);
};

```

```javascript
// app/controller 中
    async send() {
      const { ctx, app } = this;
      const nsp = app.io.of('/');

      let msg = '{"id":2, "message":666}'

      let data = await JSON.parse(msg)

      // app.io.controllers.chat(msg)
      nsp.emit('chat', data);
      return ctx.body = 'hi, egg';
    }

```







## 3 vue-socket.io 的使用


usage： 
```javascript
cnpm install --save vue-socket.io
cnpm install --save socket.io-client

```

如果出现页面显示不出来，或者出现  TypeError: Cannot call a class as a function

可以尝试把依赖 替换成   "vue-socket.io": "^2.1.1-a"
id 对应上面的 _id 后面的id



## 3.2  连接服务器，以及接收服务端消息 
```javascript
// src/main.js
import VueSocketio from 'vue-socket.io';
import socketio from 'socket.io-client';

Vue.use(VueSocketio, socketio('http://127.0.0.1:7001/'));

new Vue({
  el: '#app',
  router,
  template: '<App/>',
  components: { App },
  sockets: {
    connect: function () {
      console.log('socket connected');
    },
    res: function (val) {
      console.log('接收到服务端消息', val);
    }
  }
});
```
Vue.use()里面的 url 是你服务器地址。

connect 是 http://socket.io 默认的事件，看这名字就知道是干啥的了，另外一个 res 是自定义的监听事件，表示监听服务端发送的名为 res 的事件。




## 3.3 这里我们使用的事件名为 chat，所以服务端会将这条消息交给 chat.js（就是上面服务器端项目里面的文件啦） 这个控制器处理。


```javascript

<script>
// ...
methods: {
  sendMessageToServer: function() {
    this.$socket.emit('chat', '111111111111');
  }
}
</script>
```



## 3.4 

项目部署到生产环境总是会出现各种各样的错误

nodejs+socket.io用nginx反向代理提示400 Bad Request-nginx反向代理nodejs

 

问题描述：在项目中引用Socket.Io，在项目部署后报错，本地运行不报错

错误原因：需要在配置文件nginx.conf中配置相关信息

解决方案：

       在nginx文件的location中添加

```javascript


       proxy_http_version 1.1;    
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

```
例如：
```javascript
server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  localhost;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
             proxy_pass  http://localhost:3100;
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection "upgrade";
             proxy_set_header Host $host;
        }
}

```


- 参考 https://zhuanlan.zhihu.com/p/28112873