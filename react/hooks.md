# 基础版 useState
1.新建 states 和 setter 2个数组，cursor用于记录指针
2.当使用useState()时候 将默认值 放入states 将cursor设置为数组对应索引

此处使用闭包
```
const [book, getBook] = useState('chinese book')
states.push('chinese book')
setter.push(function (cursor){
  return function (value){
    book = value
  }
})
cursor = states的索引
```

```
let states = []
let setters = []
let firstRun = true
let cursor = 0

//  使用工厂模式生成一个 createSetter，通过 cursor 指定指向的是哪个 state
function createSetter(cursor) {
  return function(newVal) { // 闭包
    states[cursor] = newVal
  }
}

function useState(initVal) {
  // 首次
  if(firstRun) {
    states.push(initVal)
    setters.push(createSetter(cursor))
    firstRun = false
  }
  let state = states[cursor]
  let setter = setters[cursor]
  // 光标移动到下一个位置
  cursor++
  // 返回
  return [state, setter]
}

function App() {
  // 每次重置 cursor
  cursor = 0
  return <RenderFunctionComponent />
}
function RenderFunctionComponent() {
  const [firstName, setFirstName] = useState("Rudi");
  const [lastName, setLastName] = useState("Yardley");

  return (
    <Button onClick={() => setFirstName("Fred")}>Fred</Button>
  );
}
```

# 进阶版 useState

简单实现：只是链表

屏幕上显示内容对应的 Fiber 树称为 current Fiber树, 内存中构建的 Fiber 树称为 workInProgress Fiber树
current Fiber 树中的 Fiber 节点被称为 current fiber ，workInProgress Fiber 树中的 Fiber 节点被称为 workInProgress fiber ，他们通过 alternate 属性连接

```
// workInProgressHook 指针，指向当前 hook 对象
let workInProgressHook = null
// workInProgressHook fiber，这里指的是 App 组件
let fiber = {
  stateNode: App, // App 组件
  memoizedState: null // hooks 链表，初始为 null
}
// 是否是首次渲染
let isMount = true
```
初始化阶段，react hooks 做的事情，在一个函数组件第一次渲染执行上下文过程中，
> 每个 react hooks执行，都会产生一个 hook 对象，并形成链表结构，绑定在 workInProgress 的 memoizedState 属性上
```
// workInProgressHook 指针，指向当前 hook 对象
let workInProgressHook = null
// workInProgressHook fiber，这里指的是 App 组件
let fiber = {
  stateNode: App, // App 组件
  memoizedState: null // hooks 链表，初始为 null
}
// 是否是首次渲染
let isMount = true
```

然后 react hooks 上的状态，绑定在当前 hooks 对象的 memoizedState 属性上

```
// 每个 hook 对象，例如 state hook、memo hook、ref hook 等
let hook = {
  memoizedState: initVal, // 当前state的值，例如 useState(initVal)
  action: null, // update 函数
  next: null // 因为是采用链表的形式连接起来，next指向下一个 hook
}
```
```
function useState(initVal) {
  let hook
  // 首次会生成 hook 对象，并形成链表结构，绑定在 workInProgress 的 memoizedState 属性上
  if(isMount) {
    // 生成当前 hook 对象
    hook = {
      memoizedState: initVal, // 当前state的值，例如 useState(initVal)
      action: null, // update 函数
      next: null // 因为是采用链表的形式连接起来，next指向下一个 hook
    }
    
    // 绑定在 workInProgress 的 memoizedState 属性上
    if(!fiber.memoizedState) {
      // 如果是第一个 hook 对象
      fiber.memoizedState = hook
      // 指针指向当前 hook
      // workInProgressHook = hook
    } else {
      // 如果不是, 将 hook 追加到链尾
      workInProgressHook.next = hook
      // workInProgressHook 指向下一个，鸡 hook
      // workInProgressHook = workInProgressHook.next
    }
    workInProgressHook = hook
  }
}
```
对于一次函数组件更新，当再次执行 hooks 函数的时候，比如 useState(0) ，首先要从 current 的 hooks 中找到与当前 workInProgressHook ，对应的 current hook

```
function useState(initVal) {
  let hook
  // 首次会生成 hook 对象，并形成链表结构，绑定在 workInProgress 的 memoizedState 属性上
  if(isMount) {
    // ...
  } else {
    // 拿到当前的 hook
    hook = workInProgressHook
    // workInProgressHook 指向链表的下一个 hook
    workInProgressHook = workInProgressHook.next
  }
}
```
接下来 hooks 函数执行的时候，把最新的状态更新到 workInProgressHook ，保证 hooks 状态不丢失

```
function useState(initVal) {
  let hook
  // 首次会生成 hook 对象，并形成链表结构，绑定在 workInProgress 的 memoizedState 属性上
  if(isMount) {
    // ...
  } else {
    // ...
  }
  // 状态更新，拿到 current hook，调用 action 函数，更新到最新 state
  let baseState = hook.memoizedState 
  // 执行 update 函数
  if(hook.action) {
    // 更新最新值
    let action = hook.action
    // 如果是 setNum(num=>num+1) 形式
    if(typeof action === 'function') {
      baseState = action(baseState)
    } else {
      baseState = action
    }
    // 清空 action
    hook.action = null
  }
  // 更新最新值
  hook.memoizedState = baseState
  // 返回最新值 baseState、dispatchAction
  return [baseState, dispatchAction(hook)]
}

// action 函数
function dispatchAction(hook) {
  return function (action) {
    hook.action = action
  }
}
```

# 完整实现
```
// workInProgressHook 指针，指向当前 hook 对象
let workInProgressHook = null
// workInProgressHook fiber，这里指的是 App 组件
let fiber = {
  stateNode: App, // App 组件
  memoizedState: null // hooks 链表，初始为 null
}
// 是否是首次渲染
let isMount = true

function schedule() {
  workInProgressHook = fiber.memoizedState
  const app = fiber.stateNode()
  isMount = false
  return app
}

function useState(initVal) {
  let hook
  // 首次会生成 hook 对象，并形成链表结构，绑定在 workInProgress 的 memoizedState 属性上
  if(isMount) {
    // 每个 hook 对象，例如 state hook、memo hook、ref hook 等
    hook = {
      memoizedState: initVal, // 当前state的值，例如 useState(initVal)
      action: null, // update 函数
      next: null // 因为是采用链表的形式连接起来，next指向下一个 hook
    }
    // 绑定在 workInProgress 的 memoizedState 属性上
    if(!fiber.memoizedState) {
      // 如果是第一个 hook 对象
      fiber.memoizedState = hook
    } else {
      // 如果不是, 将 hook 追加到链尾
      workInProgressHook.next = hook
    }
    // 指针指向当前 hook，链表尾部，最新 hook
    workInProgressHook = hook
  } else {
    // 拿到当前的 hook
    hook = workInProgressHook
    // workInProgressHook 指向链表的下一个 hook
    workInProgressHook = workInProgressHook.next
  }
  // 状态更新，拿到 current hook，调用 action 函数，更新到最新 state
  let baseState = hook.memoizedState
  // 执行 update
  if(hook.action) {
    // 更新最新值
    let action = hook.action
    // 如果是 setNum(num=>num+1) 形式
    if(typeof action === 'function') {
      baseState = action(baseState)
    } else {
      baseState = action
    }
    // 清空 action
    hook.action = null
  } 
  // 更新最新值
  hook.memoizedState = baseState
  // 返回最新值 baseState、dispatchAction
  return [baseState, dispatchAction(hook)]
}

// action 函数
function dispatchAction(hook) {
  return function (action) {
    hook.action = action
  }
}
```
测试：
```
// 调度函数，模拟 react scheduler
function schedule() {
  workInProgressHook = fiber.memoizedState
  const app = fiber.stateNode()
  isMount = false
  return app
}

function App() {
  const [num, setNum] = useState(0)
  return {
    onClick() {
      console.log('num: ', num)
      setNum(num+1)
    }
  }
}

// 测试结果
schedule().onClick(); // 'num: ' 0
schedule().onClick(); // 'num: ' 1
schedule().onClick(); // 'num: ' 2
```

# 优化版：useState 是如何更新的

更近一步，其实 useState 有两个阶段，负责初始化的 mountState 与负责更新的 updateState，在 mountState 阶段会创建一个 state hook.queue 对象，保存负责更新的信息（包含 pending，待更新队列），以及一个负责更新的函数 dispatchAction （就是 setNum ，第三个参数就是 queue）

```
/ 因此，实际的 hook 是这样的
// 每个 hook 对象，例如 state hook、memo hook、ref hook 等
let hook = {
  memoizedState: initVal, // 当前state的值，例如 useState(initVal)
  queue: {
    pending: null
  }, // update 待更新队列, 链表的形式存储
  next: null // 因为是采用链表的形式连接起来，next指向下一个 hook
}

// 调用updateNum实际上调用这个,queue就是当前hooks对应的queue。
function dispatchAction(queue, action) {
 // 每一个任务对应一个update
  const update = {
    // 更新执行的函数
    action,
    // 与同一个Hook的其他更新形成链表
    next: null,
  };
  // ...
}
```

每次更新的时候（updateState）都会创建一个 update 对象，里面记录了此次更新的信息，然后将此update 放入待更新的 pending 队列中，最后，dispatchAction 判断当前 fiber 没有处于更新阶段

如果处于渲染阶段，那么不需要我们在更新当前函数组件，只需要更新一下当前 update 的expirationTime 即可

没有处于更新阶段，获取最新的 state ,和上一次的 currentState ，进行浅比较


如果相等，那么就退出
不相等，那么调用 scheduleUpdateOnFiber 调度渲染当前 fiber


>>> https://mp.weixin.qq.com/s/DhnZSmoVQuHlhb3wD4frCg
