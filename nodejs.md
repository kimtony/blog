## nodejs
```
js ==> v8 ==> 中间层(libuv) ==> 操作系统
1、事件驱动
2、异步IO
3、epoll (IO多路复用)
```

* io
```
网络IO
文件IO
内存IO
标准IO
```
* 非io
```
定时器（setTimeout，setInterval）
microtask（promise）
process.nextTick
setImmediate
DNS.lookup
```


Nodejs事件循环可以划分为两种，微任务和宏任务
* 宏任务
```
timers 执行setTimeout和setInterval的回调
pending callbacks 执行推迟的回调如IO，计时器
idle，prepare 空闲状态 nodejs内部使用无需关心
poll 执行与I/O相关的回调（除了关闭回调、计时器调度的回调和setImmediate之外，几乎所有回调都执行） 例如 fs的回调 http回调
check 执行setImmediate的回调
close callback 执行例如socket.on('close', ...) 关闭的回调
```
* 微任务
```
process.nextTick
promise
```


## 事件循环
```
1.概述
和浏览器中一样NodeJS中也有事件环(Event Loop),
但是由于执行代码的宿主环境和应用场景不同,
所以两者的事件环也有所不同.

>扩展阅读: 在NodeJS中使用libuv实现了Event Loop.
>源码地址: https://github.com/libuv/libuv

2.NodeJS事件环和浏览器事件环区别
2.1任务队列个数不同
浏览器事件环有2个事件队列(宏任务队列和微任务队列)
NodeJS事件环有6个事件队列
2.2微任务队列不同
浏览器事件环中有专门存储微任务的队列
NodeJS事件环中没有专门存储微任务的队列
2.3微任务执行时机不同
浏览器事件环中每执行完一个宏任务都会去清空微任务队列
NodeJS事件环中只有同步代码执行完毕和其它队列之间切换的时候回去清空微任务队列
2.4微任务优先级不同
浏览器事件环中如果多个微任务同时满足执行条件, 采用先进先出
NodeJS事件环中如果多个微任务同时满足执行条件, 会按照优先级执行

2.NodeJS中的任务队列
    ┌───────────────────────┐
┌> │timers          │执行setTimeout() 和 setInterval()中到期的callback
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │pending callbacks│执行系统操作的回调, 如:tcp, udp通信的错误callback
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │idle, prepare   │只在内部使用
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │poll            │执行与I/O相关的回调
    │                  (除了close回调、定时器回调和setImmediate()之外，几乎所有回调都执行);
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │check           │执行setImmediate的callback
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└─┤close callbacks │执行close事件的callback，例如socket.on("close",func)
    └───────────────────────┘


    ┌───────────────────────┐
┌> │timers          │执行setTimeout() 和 setInterval()中到期的callback
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │poll            │执行与I/O相关的回调
    │                  (除了close回调、定时器回调和setImmediate()之外，几乎所有回调都执行);
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└─┤check           │执行setImmediate的callback
    └───────────────────────┘

1.注意点:
和浏览器不同的是没有宏任务队列和微任务队列的概念
宏任务被放到了不同的队列中, 但是没有队列是存放微任务的队列
微任务会在执行完同步代码和队列切换的时候执行

什么时候切换队列?
当队列为空(已经执行完毕或者没有满足条件回到)
或者执行的回调函数数量到达系统设定的阈值时任务队列就会切换

2.注意点:
在NodeJS中process.nextTick微任务的优先级高于Promise.resolve微任务
-->
<!--
            ┌───────────────────────┐
            │                同步代码
            └──────────┬────────────┘
                                  │
                                  │ <---- 满足条件微任务代码
                                  │
            ┌──────────┴────────────┐
        ┌> │timers          │执行setTimeout() 和 setInterval()中到期的callback
        │  └──────────┬────────────┘
        │                        │
        │                        │ <---- 满足条件微任务代码
        │                        │
        │  ┌──────────┴────────────┐
        │  │poll            │执行与I/O相关的回调
        │  │                  (除了close回调、定时器回调和setImmediate()之外，几乎所有回调都执行);
        │  └──────────┬────────────┘
        │                        │
        │                        │ <---- 满足条件微任务代码
        │                        │
        │  ┌──────────┴────────────┐
        └─┤check           │执行setImmediate的callback
            └───────────────────────┘

注意点:
执行完poll, 会查看check队列是否有内容, 有就切换到check
如果check队列没有内容, 就会查看timers是否有内容, 有就切换到timers
如果check队列和timers队列都没有内容, 为了避免资源浪费就会阻塞在poll
```
