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
