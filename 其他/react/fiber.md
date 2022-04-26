本文首发于：https://github.com/bigo-frontend/blog/ 欢迎关注、转载。

谈谈 React Fiber 与分片
React 的理念和 Fiber 的出现
从 React 的 Doc 上可以看到 React 的理念是：

```
React is, in our opinion, the premier way to build big, fast Web apps with JavaScript.
```

但是我们有时候一个很长很深的 DOM 列表(在没有做列表优化的前提下)，setState 创建更新后，React 会进行对比创建前和创建后的节点(Reconcilation 阶段)，对比的过程是不可中断的，
由于网页的主线程不仅包含了 js 执行，样式计算, 还包含了渲染需要的重排重绘，也就是当 Reconcilation(js 执行任务)执行很久的时候，当前的任务在主线程占用时间过多，就会影响浏览器正常的重排/重绘，也会影响正常的用户交互(输入，点击，选择等等)。

举个比较极端的例子，我们有个很深的列表(1500 层)，而且变化频繁：

```
function App() {
  const [randomArray, setRandomArray] = useState(
    Array.from({ length: 1500 }, () => Math.random())
  );

  useEffect(() => {
     changeRandom()
  }, []);

  const changeRandom = () => {
    setRandomArray(randomSort(randomArray));
    cancelAnimationFrame(raf);
    raf = requestAnimationFrame(changeRandom);
  };

  const finalList = randomArray.reduce((acc, cur) => {
    acc = (
      <div key={cur} style={{ color: randomColor() }}>
        {cur} {acc}
      </div>
    );
    return acc;
  }, <div></div>);

  return (
    <div>
      <section>{finalList}</section>
    </div>
  );
}
```

从 performance 面板看，也是 changeRandom 所触发的整个 js 执行任务占用了 161ms

![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/a.png)

从事件循环看，changeRandom 函数执行 setState 进入 reconcilation 阶段，但是由于列表层次太深，整个过程又是不可中断的，所以耗时多阻碍了其他任务包括键盘输出，样式计算， 重排, 重绘等：

![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/b.png)

从浏览器的一帧来看，当上述的 reconcilationtask 阻塞了太久，导致正常刷新率情况下的每帧 16.6ms 下没有更新视图，造成掉帧的问题

![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/c.png)
所以基于 React 的理念，为了解决上述的问题，实现 reconcilation 过程可中断，当然包括其他比如 Concurrent Mode 的那些试验性的 feature, 以及优先级调度相关的东西， React 决定使用 Fiber 重写底层实现

> Fiber 的数据结构和 Fiber 树的构建

还是上述的代码，我们随便找一个渲染出来的 DOM 元素

```
右健审查
  |
在对应的DOM标签中右键store as global variable
  |
切换控制台到console
  |
输入temp1.__reactInternalInstance$mszvvg3x40p(后面会有智能提示)
```

即可看到当前节点对应的 Fiber 信息

![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/d.png)

列出主要的几个 Fiber 数据结构：

tag: 表示 Fiber 节点的类型

```
export const FunctionComponent = 0; // FC对应的Fiber节点
export const ClassComponent = 1;
export const IndeterminateComponent = 2; // Before we know whether it is function or class
export const HostRoot = 3; // 根Fiber
export const HostPortal = 4; // A subtree. Could be an entry point to a different renderer.
export const HostComponent = 5; // DOM文档的节点对应Fiber 如div,section...
export const HostText = 6; // 文本节点
export const Fragment = 7;
export const Mode = 8;
export const ContextConsumer = 9;
export const ContextProvider = 10;
export const ForwardRef = 11;
export const Profiler = 12;
export const SuspenseComponent = 13;
export const MemoComponent = 14;
export const SimpleMemoComponent = 15;
export const LazyComponent = 16;
export const IncompleteClassComponent = 17;
export const DehydratedSuspenseComponent = 18;
export const EventComponent = 19;
export const EventTarget = 20;
export const SuspenseListComponent = 21;

```

type： HostComponent 则当前 React 元素的 DOMElement 类型， 如果是组件 Fiber，则指向组件的类或者函数
stateNode: 指向创建的真实的 DOM 对象
return child 和 sibling：分别对应当前 Fiber 的父 Fiber, 第一个子 Fiber 和兄弟 Fiber 节点
alternate：双缓存使用

```
current.alternate = workInProgress
workInProgress.alternate = current
```

demo 中构建的 Fiber 树是这样的：

![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/f.png)

React 渲染的两个流程
render/reconcilation (可中断 interruptible)
beginWork 主要的作用是创建初始化和更新的当前 Fiber 节点的子 Fiber 节点，并且返回当前 Fiber 的第一个子节点去开始下一次 performUnitOfWork
completeWork 主要是创建 Fiber.stateNode 的过程，即根据 beginWork 生成的新 Fiber 调用 document.createElement 去创建 DOM 节点存储在 Fiber.stateNode 中，再在 commit 流程的时候去 append 到真实的 DOM 中
commit(不能中断，否则 DOM 可能变来变去，UI DOM 不稳定)；主要是将 render 阶段生成的 stateNode commit 到真实的 DOM 节点中
Scheduler: 调度模块，调度 render/reconcilation 阶段的任务，将任务分为 5ms 一个，可中断

开启 Concurrent Mode 以及分片
启动 Fiber 时间分片功能需要开启 Concurrent Mode 模式，也就是说我们平时开发中默认用的 ReactDOM.render，虽然用了 Fiber，但其实没有用到时间分片。

开启 Concurrent Mode 只需要两个步骤：

装上试验性的 react 和 react-dom 包

```
npm install react@experimental react-dom@experimental
```

使用 ReactDOM.createRoot 创建一个 FiberRoot 再 render 替代 ReactDOM.render

```
ReactDOM.createRoot(rootNode).render(<App />)
```

开启完毕。

当然除了我们平常用的 ReactDOM.render 的 Legacy Mode 以及 Concurrent Mode, React 还出了一个 Blocking Mode,其实就是拥有部分 Concurrent Mode 功能的一个中间版本，创建方式是:ReactDOM.createBlockingRoot(rootNode).render(<App />)
三种模式的对比：

![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/g.png)

可以看到在 ConcurrentMode 下，开始有了 SuspenseList,可以控制 Suspense 组件的一个顺序，也支持了优先级渲染，中断预渲染等，还有一些新的 hook, 比如用 useTransition 搭配 Suspense 可以用来做加载的优化，useDefferredValue 来做一些 state 值的缓存，对于某些优先级不是很高但是又很耗时间的更新，可以不用立即更新，而是获取 deffer 延迟的 state 等等，但是这些还是还是在试用包里面，可能随时会改，所以就不细说，感兴趣的可以看下：
Suspense for Data Fetching

开启分片后性能和用户体验对比(concurrent mode vs legacy mode)
我们回到最开始的 demo, 我们对比下开启 Concurrent Mode（也就是开启分片）前后的 performance 面板对比：
开启分片前：

![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/h.png)

可以看到主线程上的每次更新都是由 changeRandom 发起然后再进行 reconcilation 阶段和 commit 阶段,整个方法包含在一个 Task 里面：
![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/i.png)
开启分片后：
开启分片后,可以看到 render/reconcilation 阶段分成了很多个任务，有很多个 Task 都是 5ms 的任务
![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/j.png)

这时候我们加一个输入框，来测试是否用户体验提升很多，是否 reconcilation 分片真的这么强。

于是加个输入框，并且不要让输入框影响随机的 div 深列表，所以我们把 List 单独抽出来，并且用 React.memo 包起来，这样输入框的 setState 并不会引发 List 的重渲染：
外层：

```
function App() {
  const [filterText, setFilterText] = useState('');
  return (
    <div>
      <input
        value={filterText}
        onChange={(e) => setFilterText(e.target.value)}
      />
      <button>按钮</button>
      <section>
        <List />
      </section>
    </div>
  );
}
```

随机 List:

```
const List = React.memo(function List (props) {
  const [randomArray, setRandomArray] = useState(
    Array.from({ length: 1500 }, () => Math.random())
  );

  useEffect(() => {
    changeRandom()
 }, []);

 const changeRandom = () => {
   setRandomArray(randomSort(randomArray));
   cancelAnimationFrame(raf);
   raf = requestAnimationFrame(changeRandom);
 };

 const finalList = randomArray.reduce((acc, cur) => {
   acc = (
     <span key={Math.random()} style={{ color: randomColor() }}>
       {cur} {acc}
     </span>
   );
   return acc;
 }, <span></span>);
 return <div>{finalList}</div>
})

```

在 Legacy 模式下，可以看到输入框的输入有点卡顿，主要是整个 render/reconcilation 任务占用了太多时间导致
![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/k.png)
然后我们开启 Concurrent Mode,发现还是被阻塞了，还是会有一点卡，看 performance 发现主要是被 commit 和 layout/paint 两个流程卡住了，
![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/l.png)

当然当输入也没有刚好卡在 render/reconcilation 的分片当中，是不会被 render/reconcilation 本身阻塞的，所以可以总结: render/reconcilation 的分片以及达到了效果，在分片的间隔时间已经可以去插入执行其他优先级更高的用户相应了。
![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/m.png)

但是，为了更好的演示分片带来的效果，我决定排除 commit 流程和 Layout/Paint 重排重绘带来的影响。

抛开 Layout/Paint 流程和 commit 流程来看分片带来的 performance 优化
抛开重排重绘带来的影响: 直接给 List 组件外层套一个 style={{ display: 'none' }}, 这样 render/reconcilation 阶段完成之后就不会进行重排重绘了，但是生成的 Fiber 还是会生成，只不过最后 commit 到 DOM 上也不会渲染

```
<section style={{ display: 'none' }}>
  <List />
</section>
```

为了效果更明显，我直接拷贝多了两个 List, 并且每个 List 增加到 3000 条,

```
const List = React.memo(function List (props) {
  const [randomArray, setRandomArray] = useState(
    Array.from({ length: 3000 }, () => Math.random())
  );

  useEffect(() => {
    changeRandom()
 }, []);

 const changeRandom = () => {
   setRandomArray(randomSort(randomArray));
   cancelAnimationFrame(raf);
   raf = requestAnimationFrame(changeRandom);
 };

 const finalList = randomArray.reduce((acc, cur) => {
   acc = (
     <span key={Math.random()} style={{ color: randomColor() }}>
       {cur} {acc}
     </span>
   );
   return acc;
 }, <span></span>);

 return <div>{finalList}{finalList1}{finalList2}</div>

})
```

这样， 在 Legacy 模式下， 我们看到 performance 里面，已经没有 Layout/Paint 这样的任务来阻碍我们的输入了，只剩下 commit，现在加大 List 数量的情况下，render/reconcilation 大概阻塞了 500 多 ms, 很卡。

![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/n.png)

这时候再看 Concurrent Mode，很流畅，commitRoot 直接没有了(也算是 Concurrent Mode 一个优化吧，对于 display:none 来说，本身 commit 就没有必要)

![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/o.png)

总结及 Concurrent Mode 的其他 Features
当然我们举了很夸张的例子(深节点，移除重排重绘)来单独看 Concurrent Mode 模式下对于 render/reconcilation 带来的优化效果。当然分片只是 Fiber 的一小部分功能，Fiber 架构解锁了很多 Concurrent Mode 的新功能： <SuspenseList>, useTransition, useDeferredValue 等等，当然这些暂时是试用性的。 在我们的 demo 中，可以使用 useDeferredValue 来做 state 的延迟，比如我们轮训获取到了实时的长列表，但是又不想阻塞输入框等用户的操作，我们可以将旧的 state 用 useDeferredValue 暂时存起来，然后将旧版的 state 传给带 memo 的组件，这时候我们通过降低了列表的时效性来换取了用户交互体验的提升，而且我们原 state 永远是最新的，所以跟增大轮询时间又不太一样。
总之，Concurrent Mode 解锁了很多新的功能，当然有些是试用性的，但是可以期待当 Concurrent Mode 正式使用的时候，新特性给性能和用户体验带来的提升。

---

Fiber 的前任： Stack Reconciler
Stack Reconciler（下文将简称为 Stack）作为 Fiber 的前任调度器，就像它的名字一样，通过栈的方式实现任务的调度：将不同任务（渲染变动）压入栈中，浏览器每次绘制的时候，将执行这个栈中已经存在的任务。

说到这，Stack 的问题已经很明显的暴露出来了。我们知道设备刷新频率通常为 60Hz，如今支持高刷（120Hz+）的设备也在不断增加，页面每一帧所消耗掉的时间也在不断减少 1s/60↑ ≈ 16ms↓，在这段时间内，浏览器需要执行如下任务

可用户并不关心上面的大部分流程，只需要页面可以及时的展示就足够了。如果我们在一次渲染时，向栈中推入了过多的任务，从而导致其执行时间超过浏览器的一帧，就会使这一帧没能及时响应渲染页面，也是就我们常说的掉帧。

而 Stack 这种架构的特点就是，所有任务都按顺序的压入了栈中，而执行的时候无法确认当前的任务是否会耗去过长的脚本运行时间，使得这一帧时间内里浏览器能做的事不可控。

所以可控便成了 React 团队的优化方向，Fiber Reconciler 应运而生。

requestIdleCallback 方法插入一个函数，这个函数将在浏览器空闲时期被调用。这使开发者能够在主事件循环上执行后台和低优先级工作，而不会影响延迟关键事件。
requestAnimationFrame
requestAnimationFrame 比起 setTimeout、setInterval 的优势主要有两点：
1、requestAnimationFrame 会把每一帧中的所有 DOM 操作集中起来，在一次重绘或回流中就完成，并且重绘或回流的时间间隔紧紧跟随浏览器的刷新频率，一般来说，这个频率为每秒 60 帧。
2、在隐藏或不可见的元素中，requestAnimationFrame 将不会进行重绘或回流，这当然就意味着更少的的 cpu，gpu 和内存使用量。

```
rafId = requestAnimationFrame(animloop)
cancelAnimationFrame(rafId)
```

![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/react/q.png)

在 19 年的一次更新中，React 团队推翻之前的设计，使用了 MessageChannel 来实现了对于线程控制。

MessageChannel 允许我们创建一个新的消息通道，并通过它的两个 MessagePort 属性发送数据。此特性在 Web Worker 中可用。其使用方式如下：

```
const channel = new MessageChannel()

channel.port1.onmessage = function(msgEvent) {
  console.log('recieve message!')
}

channel.port2.postMessage(null)
```

// output: recieve message!
React 开发成员对这次更新这样说道：requestAnimationFrame 过于依赖硬件设备，无法在其之上进一步减少任务调度频率，以获得更大的优化空间。使用高频（5ms）少量的消息事件进行任务调度，虽然会加剧主线程与其他浏览器任务的争用，但却值得一试。
