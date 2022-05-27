# Worker

| 类型                          | Worker        | SharedWorker        | ServiceWorker        |
| ----------------------------- | ----------- | ----------- |----------- |
| 通信方式                     | postMessage | port.postMessage | 单向通信，通过addEventListener 监听serviceWorker 的状态 |
| 使用场景                     | 适合大量计算的场景适合跨 tab、iframes之间共享数据    | 适合跨 tab、iframes之间共享数据 | 缓存资源、网络优化 |
| 兼容性                       | >= IE 10>= Chrome 4   | 不支持 IE、Safari、Android、iOS、>= Chrome 4 | 不支持 IE、>= Chrome 40 |


1、 普通的 Worker 可以在需要大量计算的时候使用，创建新的线程可以降低主线程的计算压力，不会导致 UI 卡顿。

2、SharedWorker 主要是为不同的 window、iframes 之间共享数据提供了另外一个解决方案。

3、ServiceWorker 可以缓存资源，提供离线服务或者是网络优化，加快 Web 应用的开启速度，更多是优化体验方面的

https://juejin.cn/post/7091068088975622175