# React Fiber 的思想和协程的概念是契合的
React 渲染的过程可以被中断，可以将控制权交回浏览器，让位给高优先级的任务，浏览器空闲后再恢复渲染。

# requestIdleCallback

window.requestIdleCallback() 方法插入一个函数，这个函数将在浏览器空闲时期被调用。这使开发者能够在主事件循环上执行后台和低优先级工作，而不会影响延迟关键事件，如动画和输入响应

```
var handle = window.requestIdleCallback(callback[, options])
```
下面是该方法参数的介绍：

# callback


一个在事件循环空闲时即将被调用的函数的引用。函数会接收到一个名为IdleDeadline的参数，这个参数可以获取当前空闲时间以及回调是否在超时时间前已经执行的状态。


# options 可选

options.timeout：如果指定了timeout，并且有一个正值，而回调在timeout毫秒过后还没有被调用，那么回调任务将放入事件循环中排队，即使这样做有可能对性能产生负面影响。

callback 方法中的 IdleDeadline 参数为：

# didTimeout


一个Boolean类型当它的值为true的时候说明callback正在被执行(并且上一次执行回调函数执行的时候由于时间超时回调函数得不到执行)，因为在执行requestIdleCallback回调的时候指定了超时时间并且时间已经超时。


 # timeRemaining()


返回一个时间DOMHighResTimeStamp, 并且是浮点类型的数值，它用来表示当前闲置周期的预估剩余毫秒数。如果idle period已经结束，则它的值是0。你的回调函数(传给requestIdleCallback的函数)可以重复的访问这个属性用来判断当前线程的闲置时间是否可以在结束前执行更多的任务


# requestAnimationFrame

浏览器在一帧中会做什么事情
<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0bfb8c90bfc24e5b9d3a602aa3501f33~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?"/>

callback 参数：

下一次重绘之前更新动画帧所调用的函数(即上面所说的回调函数)。该回调函数会被传入DOMHighResTimeStamp参数，该参数与performance.now()的返回值相同，它表示requestAnimationFrame() 开始去执行回调函数的时刻

如果我在requestAnimationFrame()的回调中开启一个宏任务，那这个宏任务会在本帧的浏览器渲染后执行，这时我获取该回调方法的执行时机并和上文中的deadline做比较

```
if(now - deadline < 0) {
    // 本帧有空闲时间
}
```

> setTimeout
放弃了setTimeout，因为浏览器在执行setTimeout()和setInterval()时，会设定一个最小的时间阈值，一般是 4ms
> MessageChannel
MessageChannel来执行宏任务由于没有时延效果更加的出色

基于MessageChannel 实现requestIdleCallback
1.setTimeout setInterval 4ms延时
2.messagechannel 无延时
