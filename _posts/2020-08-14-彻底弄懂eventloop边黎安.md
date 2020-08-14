---
layout:     post
title:      彻底弄懂eventloop
date:       2020-08-14
author:    	边黎安
header-img: img/tag-bg.jpg
catalog: true
tags:
    - node
    - eventloop
---
---


## 为什么要写这边文章？
我觉得大量的文章大家写的要凑在一起才能很好地理解 了解eventloop到底是什么机制 浏览器的nodejs 和 eventloop 又有什么区别 
### eventloop是什么?
```
eventloop 是在 h5中明确定义好的，而node 的事件循环是丢给了libuv 

```
### 微任务和宏任务的区别?
```
宏任务 macrotask  我的理解是 一些异步调用会进入 macrotask 等待 后续调用 例如（settimeout setinterval setimmediate  requestAnimation  i/o uirendering） 当前函数调用栈中执行的代码成为宏任务
微任务 microtask 例如 nexttick promise  当前（此次事件循环中）宏任务执行完，在下一个宏任务开始之前需要执行的任务,可以理解为回调事件。（promise.then，proness.nextTick等等）。 3. 宏任务中的事件放在callback queue中，由事件触发线程维护；微任务的事件放在微任务队列中，由js引擎线程维护。
讲概念 会有点蒙
```
###  浏览器的eventloop
浏览器有有个script 的标签，开始执行了里面大部分是同步的也有一些异步的比如settimeout，执行完之后清空了调用栈，紧接着微队列比如promise里面的回调凤哪敢如stack中 执行完queue -1，以此类推吧micritask 的做完 举个例子 如果promise 里面 回调了一个promise 那么 会放到这个周期的最后，做完所有的mircotask后 取出宏队列中执行 清空栈 
这就是浏览器的我跟出来的结果

```
console.log(1);

setTimeout(() => {
  console.log(2);
  Promise.resolve().then(() => {
    console.log(3)
  });
});

new Promise((resolve, reject) => {
  console.log(4)
  resolve(5)
}).then((data) => {
  console.log(data);
})

setTimeout(() => {
  console.log(6);
})

console.log(7);
```
我的模拟例子  1475236

```
分析下流程
执行全局Script代码
console.log(1)
```
Stack Queue: [console]

Macrotask Queue: []

Microtask Queue: []
```
Step 2
```
setTimeout(() => {
  // 这个回调函数叫做callback1，setTimeout属于macrotask，所以放到macrotask queue中
  console.log(2);
  Promise.resolve().then(() => {
    console.log(3)
  });
});
```
Stack Queue: [setTimeout]

Macrotask Queue: [callback1]

Microtask Queue: []


Step 3
```
new Promise((resolve, reject) => {
  // 注意，这里是同步执行的，如果不太清楚，可以去看一下我开头自己实现的promise啦~~
  console.log(4)
  resolve(5)
}).then((data) => {
  // 这个回调函数叫做callback2，promise属于microtask，所以放到microtask queue中
  console.log(data);
})
```
Stack Queue: [promise]

Macrotask Queue: [callback1]

Microtask Queue: [callback2]

打印结果：
1
4

setTimeout(() => {
  // 这个回调函数叫做callback3，setTimeout属于macrotask，所以放到macrotask queue中
  console.log(6);
})

```
Stack Queue: [setTimeout]

Macrotask Queue: [callback1, callback3]

Microtask Queue: [callback2]

打印结果：
1
4
```
Step 6
```
console.log(7)

Stack Queue: [console]

Macrotask Queue: [callback1, callback3]

Microtask Queue: [callback2]

打印结果：
1
4
7
```
```
好啦，全局Script代码执行完了，进入下一个步骤，从microtask queue中依次取出任务执行，直到microtask queue队列为空。
```
Step 7
```
console.log(data) // 这里data是Promise的决议值5
```
```
Stack Queue: [callback2]

Macrotask Queue: [callback1, callback3]

Microtask Queue: []

打印结果：
1
4
7
5

```
这里microtask queue中只有一个任务，执行完后开始从宏任务队列macrotask queue中取位于队首的任务执行

Step 8
console.log(2)
Stack Queue: [callback1]

Macrotask Queue: [callback3]

Microtask Queue: []

```
打印结果：
1
4
7
5
2
但是，执行callback1的时候又遇到了另一个Promise，Promise异步执行完后在microtask queue中又注册了一个callback4回调函数
```
Step 9

Promise.resolve().then(() => {
  // 这个回调函数叫做callback4，promise属于microtask，所以放到microtask queue中
  console.log(3)
});

```
Stack Queue: [promise]

Macrotask v: [callback3]

Microtask Queue: [callback4]

打印结果：
1
4
7
5
2
```

取出一个宏任务macrotask执行完毕，然后再去微任务队列microtask queue中依次取出执行

Step 10

```
console.log(3)
Stack Queue: [callback4]

Macrotask Queue: [callback3]

Microtask Queue: []

打印结果：
1
4
7
5
2
3
```

Step 11

```
console.log(6)
Stack Queue: [callback3]

Macrotask Queue: []

Microtask Queue: []

打印结果：
1
4
7
5
2
3
6

```
以上，全部执行完后，Stack Queue为空，Macrotask Queue为空，Micro Queue为空


换个例子再回顾下 
```
console.log(1);

setTimeout(() => {
  console.log(2);
  Promise.resolve().then(() => {
    console.log(3)
  });
});

new Promise((resolve, reject) => {
  console.log(4)
  resolve(5)
}).then((data) => {
  console.log(data);
  
  Promise.resolve().then(() => {
    console.log(6)
  }).then(() => {
    console.log(7)
    
    setTimeout(() => {
      console.log(8)
    }, 0);
  });
})
```

```
// 正确答案
1
4
10
5
6
7
2
3
9
8
```

在执行微队列microtask queue中任务的时候，如果又产生了microtask，那么会继续添加到队列的末尾，也会在这个周期执行，直到microtask queue为空停止。

```
注：当然如果你在microtask中不断的产生microtask，那么其他宏任务macrotask就无法执行了，但是这个操作也不是无限的，拿NodeJS中的微任务process.nextTick()来说，它的上限是1000个，后面我们会讲到。

浏览器的Event Loop就说到这里，下面我们看一下NodeJS中的Event Loop，它更复杂一些，机制也不太一样
```

####  NodeJS中的Event Loop
```
timers 这个阶段 执行setTimeout 和 setInterval 预订的callback
I/O callback阶段：执行除了close事件的callbacks、被timers设定的callbacks、setImmediate()设定的callbacks这些之外的callbacks
idle, prepare阶段：仅node内部使用
poll阶段：获取新的I/O事件，适当的条件下node将阻塞在这里
check阶段：执行setImmediate()设定的callbacks
close callbacks阶段：执行socket.on('close', ....)这些callbacks

```
大体解释一下NodeJS的Event Loop过程：

执行全局Script的同步代码
执行microtask微任务，先执行所有Next Tick Queue中的所有任务，再执行Other Microtask Queue中的所有任务
开始执行macrotask宏任务，共6个阶段，从第1个阶段开始执行相应每一个阶段macrotask中的所有任务，注意，这里是所有每个阶段宏任务队列的所有任务，在浏览器的Event Loop中是只取宏队列的第一个任务出来执行，每一个阶段的macrotask任务执行完毕后，开始执行微任务，也就是步骤2
Timers Queue -> 步骤2 -> I/O Queue -> 步骤2 -> Check Queue -> 步骤2 -> Close Callback Queue -> 步骤2 -> Timers Queue ......
这就是Node的Event Loop

```


第一个例子

```
console.log('start');

setTimeout(() => {          // callback1
  console.log(111);
  setTimeout(() => {        // callback2
    console.log(222);
  }, 0);
  setImmediate(() => {      // callback3
    console.log(333);
  })
  process.nextTick(() => {  // callback4
    console.log(444);  
  })
}, 0);

setImmediate(() => {        // callback5
  console.log(555);
  process.nextTick(() => {  // callback6
    console.log(666);  
  })
})

setTimeout(() => {          // callback7              
  console.log(777);
  process.nextTick(() => {  // callback8
    console.log(888);   
  })
}, 0);

process.nextTick(() => {    // callback9
  console.log(999);  
})

console.log('end');
```
setImmediate 在  setTimeout之后 同为 宏观任务
start
end
999
111
777
444
888
555
333
666
222

### setTimeout 对比 setImmediate
setTimeout(fn, 0)在Timers阶段执行，并且是在poll阶段进行判断是否达到指定的timer时间才会执行
setImmediate(fn)在Check阶段执行
两者的执行顺序要根据当前的执行环境才能确定：

如果两者都在主模块(main module)调用，那么执行先后取决于进程性能，顺序随机
如果两者都不在主模块调用，即在一个I/O Circle中调用，那么setImmediate的回调永远先执行，因为会先到Check阶段



### setImmediate 对比 process.nextTick
setImmediate(fn)的回调任务会插入到宏队列Check Queue中
process.nextTick(fn)的回调任务会插入到微队列Next Tick Queue中
process.nextTick(fn)调用深度有限制，上限是1000，而setImmedaite则没有



#### 总结
浏览器的Event Loop和NodeJS的Event Loop是不同的，实现机制也不一样，不要混为一谈。
浏览器可以理解成只有1个宏任务队列和1个微任务队列，先执行全局Script代码，执行完同步代码调用栈清空后，从微任务队列中依次取出所有的任务放入调用栈执行，微任务队列清空后，从宏任务队列中只取位于队首的任务放入调用栈执行，注意这里和Node的区别，只取一个，然后继续执行微队列中的所有任务，再去宏队列取一个，以此构成事件循环。
NodeJS可以理解成有4个宏任务队列和2个微任务队列，但是执行宏任务时有6个阶段。先执行全局Script代码，执行完同步代码调用栈清空后，先从微任务队列Next Tick Queue中依次取出所有的任务放入调用栈中执行，再从微任务队列Other Microtask Queue中依次取出所有的任务放入调用栈中执行。然后开始宏任务的6个阶段，每个阶段都将该宏任务队列中的所有任务都取出来执行（注意，这里和浏览器不一样，浏览器只取一个），每个宏任务阶段执行完毕后，开始执行微任务，再开始执行下一阶段宏任务，以此构成事件循环。
MacroTask包括： setTimeout、setInterval、 setImmediate(Node)、requestAnimation(浏览器)、IO、UI rendering
Microtask包括： process.nextTick(Node)、Promise、Object.observe、MutationObserver
Node 在新版本中，也是每个 Macrotask 执行完后，就去执行 Microtask 了，和浏览器的模型一致。




### 持续关注　https://bianbiandashen.github.io/　个人博客

 

