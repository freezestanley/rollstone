JS可以操作DOM，GUI渲染线程与JS线程是互斥的。所以JS脚本执行和浏览器布局、绘制不能同时执行。
在每16.6ms时间内，需要完成如下工作：

JS脚本执行 -----  样式布局 ----- 样式绘制
当JS执行时间过长，超出了16.6ms，这次刷新就没有时间执行样式布局和样式绘制了。

在Demo中，由于组件数量繁多（3000个），JS脚本执行时间过长，页面掉帧，造成卡顿。

在浏览器每一帧的时间中，预留一些时间给JS线程，React利用这部分时间更新组件（可以看到，在源码 (opens new window)中，预留的初始时间是5ms）。

当预留的时间不够用时，React将线程控制权交还给浏览器使其有时间渲染UI，React则等待下一帧时间到来继续被中断的工作。

这种将长任务分拆到每一帧中，像蚂蚁搬家一样一次执行一小段任务的操作，被称为时间切片（time slice）

开启Concurrent Mode
我们的长任务被拆分到每一帧不同的task中，JS脚本执行时间大体在5ms左右，这样浏览器就有剩余时间执行样式布局和样式绘制，减少掉帧的可能性。

https://react.iamkasong.com/preparation/idea.html#cpu%E7%9A%84%E7%93%B6%E9%A2%88


React15架构可以分为两层：

Reconciler（协调器）—— 负责找出变化的组件
Renderer（渲染器）—— 负责将变化的组件渲染到页面上

Reconciler会做如下工作：

调用函数组件、或class组件的render方法，将返回的JSX转化为虚拟DOM
将虚拟DOM和上次更新时的虚拟DOM对比
通过对比找出本次更新中变化的虚拟DOM
通知Renderer将变化的虚拟DOM渲染到页面上

Reconciler发现1需要变为2，通知Renderer
Renderer 更新Dom， 1变2
Reconciler发现2需要变为4，通知Renderer
Renderer 更新Dom， 2变4
我们可以看到，Reconciler和Renderer是交替工作的，当第一个li在页面上已经变化后，第二个li再进入Reconciler。

由于整个过程都是同步的，所以在用户看来所有DOM是同时更新的

由于递归执行，所以更新一旦开始，中途就无法中断。当层级很深时，递归更新时间超过了16ms，用户交互就会卡顿


# React16架构
React16架构可以分为三层：

Scheduler（调度器）—— 调度任务的优先级，高优任务优先进入Reconciler
Reconciler（协调器）—— 负责找出变化的组件
Renderer（渲染器）—— 负责将变化的组件渲染到页面上
可以看到，相较于React15，React16中新增了Scheduler（调度器），让我们来了解下他。

我们可以看见，更新工作从递归变成了可以中断的循环过程。每次循环都会调用shouldYield判断当前是否有剩余时间
```
/** @noinline */
function workLoopConcurrent() {
  // Perform work until Scheduler asks us to yield
  while (workInProgress !== null && !shouldYield()) {
    workInProgress = performUnitOfWork(workInProgress);
  }
}
```

# 代数效应

代数效应是函数式编程中的一个概念，用于将副作用从函数调用中分离。典型代表HOOKS

自我理解就是。当在函数内执行方法（异步）直接跳到调用方法的逻辑体系内 而函数的当前状态被暂存 当执行完毕后 回复函数状态继续后面的逻辑。 async generator等具有传染性 会导致 函数的改变
并且Generator执行的中间状态是上下文关联的，所以计算y时无法复用之前已经计算出的x，需要重新计算。
如果通过全局变量保存之前执行的中间状态，又会引入新的复杂度
所有放弃


# fiber 原理
stack Reconciler。React16的Reconciler基于Fiber节点实现，被称为Fiber Reconciler。

作为静态的数据结构来说，每个Fiber节点对应一个React element，保存了该组件的类型（函数组件/类组件/原生组件...）、对应的DOM节点等信息。

作为动态的工作单元来说，每个Fiber节点保存了本次更新中该组件改变的状态、要执行的工作（需要被删除/被插入页面中/被更新...

# 双缓存Fiber树

在React中最多会同时存在两棵Fiber树。当前屏幕上显示内容对应的Fiber树称为current Fiber树，正在内存中构建的Fiber树称为workInProgress Fiber树。

current Fiber树中的Fiber节点被称为current fiber，workInProgress Fiber树中的Fiber节点被称为workInProgress fiber，他们通过alternate属性连接

```
currentFiber.alternate === workInProgressFiber;
workInProgressFiber.alternate === currentFiber;
```
即当workInProgress Fiber树构建完成交给Renderer渲染在页面上后，应用根节点的current指针指向workInProgress Fiber树，此时workInProgress Fiber树就变为current Fiber树。

每次状态更新都会产生新的workInProgress Fiber树，通过current与workInProgress的替换，完成DOM更新。



mount

首次执行ReactDOM.render会创建fiberRootNode（源码中叫fiberRoot）和rootFiber。其中fiberRootNode是整个应用的根节点，rootFiber是<App/>所在组件树的根节点。
分fiberRootNode与rootFiber，是因为在应用中我们可以多次调用ReactDOM.render渲染不同的组件树，他们会拥有不同的rootFiber。但是整个应用的根节点只有一个，那就是fiberRootNode。

fiberRootNode的current会指向当前页面上已渲染内容对应Fiber树，即current Fiber树

```
fiberRootNode.current = rootFiber;
```
由于是首屏渲染，页面中还没有挂载任何DOM，所以fiberRootNode.current指向的rootFiber没有任何子Fiber节点（即current Fiber树为空）

接下来进入render阶段，根据组件返回的JSX在内存中依次创建Fiber节点并连接在一起构建Fiber树，被称为workInProgress Fiber树。（下图中右侧为内存中构建的树，左侧为页面显示的树）
在构建workInProgress Fiber树时会尝试复用current Fiber树中已有的Fiber节点内的属性，在首屏渲染时只有rootFiber存在对应的current fiber（即rootFiber.alternate）


update
接下来我们点击p节点触发状态改变，这会开启一次新的render阶段并构建一棵新的workInProgress Fiber 树

workInProgress Fiber 树在render阶段完成构建后进入commit阶段渲染到页面上。渲染完毕后，workInProgress Fiber 树变为current Fiber 树

Reconciler工作的阶段被称为render阶段。因为在该阶段会调用组件的render方法。
Renderer工作的阶段被称为commit阶段。就像你完成一个需求的编码后执行git commit提交代码。commit阶段会把render阶段提交的信息渲染在页面上。
render与commit阶段统称为work，即React在工作中。相对应的，如果任务正在Scheduler内调度，就不属于work


# 创建并构建Fiber树的。

render阶段开始于performSyncWorkOnRoot或performConcurrentWorkOnRoot方法的调用。这取决于本次更新是同步更新还是异步更新。

```
// performSyncWorkOnRoot会调用该方法
function workLoopSync() {
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}

// performConcurrentWorkOnRoot会调用该方法
function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```

