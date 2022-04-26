> 什么时候会执行「render」？
> render 函数会在两种场景下被调用：

1. 状态更新时
   a. 继承自 React.Component 的 class 组件更新状态时

```
import React from 'react'
import ReactDom from 'react-dom'
class App extends React.Component{
  render(){
    return <Foo />
  }
}
class Foo extends React.Component {
  state = {count : 0 };
  increment = () => {
    const {count} = this.state;
    const newCount = count < 10 ? count +1 : count;
    this.setState({count: newCount})
  }
  render () {
    const {count} = this.state;
    console.log('Foo render');
    return (
      <div>
        <h1>{count}</h1>
        <button onClick={this.increment}>Increment</button>
      </div>
    )
  }
}
const rootElement = document.getElementById("root");
ReactDom.render(<App />, rootElement);
```

可以看到，代码中的逻辑是我们点击就会更新 count，到 10 以后，就会维持在 10。增加一个 console.log，这样我们就可以知道 render 是否被调用了。从执行结果可以知道，即使 count 到了 10 以上，render 仍然会被调用。

# 总结：继承了 React.Component 的 class 组件，即使状态没变化，只要调用了 setState 就会触发 render。

数式组件更新状态时
我们用函数实现相同的组件，当然因为要有状态，我们用上了 useState hook：

```
import React, {useState} from 'react';
import ReactDom from 'react-dom';

class App extends React.Component{
  return() {
    return <Foo />
  }
}
function Foo () {
  const [count, setCount] = useState(0)
  function increment () {
    const newCount = count < 10 ? count+1: count;
    setCount(newCount)
  }
  console.log("Foo render");
  return(
    <div>
      <h1>{count}</h1>
      <button onClick={increment}>Increment</button>
    </div>
  )
}

const rootElement = document.getElementById("root")
ReactDOM.render(<App/>, rootElement)
```

我们可以注意到，当状态值不再改变之后，render 的调用就停止了。

# 总结：对函数式组件来说，状态值改变时才会触发 render 函数的调用。

2. 父容器重新渲染时

只要点击了 App 组件内的 Change name 按钮，就会重新 render。而且可以注意到，不管 Foo 具体实现是什么，Foo 都会被重新渲染。

# 总结：无论组件是继承自 React.Component 的 class 组件还是函数式组件，一旦父容器重新 render，组件的 render 都会再次被调

在「render」过程中会发生什么？

# 只要 render 函数被调用，就会有两个步骤按顺序执行。这两个步骤非常重要，理解了它们才好知道如何去优化 React App。

# Diffing

> 在此步骤中，React 将新调用的 render 函数返回的树与旧版本的树进行比较，这一步是 React 决定如何更新 DOM 的必要步骤。虽然 React 使用高度优化的算法执行此步骤，但仍然有一定的性能开销。

# Reconciliation

> 基于 diffing 的结果，React 更新 DOM 树。这一步因为需要卸载和挂载 DOM 节点同样存在许多性能开销。

> 谨慎分配 state 以避免不必要的 render 调用

我们以下面为例，其中 App 会渲染两个组件：

CounterLabel，接收 count 值和一个 inc 父组件 App 中状态 count 的方法。
List，接收 item 的列表。

```
import React, {useState} from 'react';
import ReactDom from 'react-dom';

const ITEMS = [1,2,3,4,5,6,7,8,9,10,11,12]

function App(){
  const [count, setCount ] = useState(0)
  const [items, setItems] = useState(ITEMS)

  return (<div>
  <CounterLabel count={count} increment={() => setCount(count +1)} />
  <List items={items} />
  </div>)
}

function CounterLabel({count, increment}) {
  return (
    <>
    <h1>{count}</h1>
    <button onClick={increment}>Increment</button>
    </>
  )
}

function List({items}) {
  console.log("List render")
  return (
    <ul>
      {items.map((item,index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  )
}

const rootElement = document.getElementById("root")
ReactDOM.render(<App/>, rootElement)
```

执行上面代码可知，只要父组件 App 中的状态被更新，CounterLabel 和 List 就都会更新。

当然，CounterLabel 重新渲染是正常的，因为 count 发生了变化，自然要重新渲染；但是对于 List 而言，就完全是不必要的更新了，因为它的渲染与 count 无关。尽管 React 并不会在 reconciliation 阶段真的更新 DOM，毕竟完全没变化，但是仍然会执行 diffing 阶段来对前后的树进行对比，这仍然存在性能开销。

还记得 render 执行过程中的 diffing 和 reconciliation 阶段吗？前面讲过的东西在这里碰到了。

因此，为了避免不必要的 diffing 开销，我们应当考虑将特定的状态值放到更低的层级或组件中（与 React 中所说的「提升」概念刚好相反）。在这个例子中，我们可以通过将 count 放到 CounterLabel 组件中管理来解决这个问题。

> 合并状态更新

因为每次状态更新都会触发新的 render 调用，那么更少的状态更新也就可以更少的调用 render 了。

# 我们知道，React class 组件有 componentDidUpdate(prevProps, prevState) 的钩子，可以用来检测 props 或 state 有没有发生变化。尽管有时有必要在 props 发生变化时再触发 state 更新，但我们总可以避免在一次 state 变化后再进行一次 state 更新这种操作：

```
import React, {useState} from 'react';
import ReactDom from 'react-dom';

function getRange(limit){
  let range = [];
  for (let i =0;i<limit;i++) {
    range.push(i)
  }
  return range;
}

class App extends React.Component {
  state = {
    numbers: getRange(7),
    limit: 7
  }
  handleLimitChange = e=> {
    const limit = e.target.value;
    const limitChanged = limit !== this.state.limit;
    if (limitChanged) {
      this.setState({limit})
    }
  }
  componentDidUpdate(prevProps, prevState) {
    const limitChanged = prevState.limit !== this.state.limit;
    if (limitChanged) {
      this.setState({numbers: getRange(this.state.limit)})
    }
  }
  render(){
    return (
      <div>
        <input
          onChange={this.handleLimitChange}
          placeholder="limit"
          value={this.state.limit}
        />
        {this.state.number.map((number,idx) => {
          return (<p key={idx}>{number}</p>)
        })}
      </div>
    )
  }
}


const rootElement = document.getElementById("root")
ReactDOM.render(<App/>, rootElement)
```

这里渲染了一个范围数字序列，即范围为 0 到 limit。只要用户改变了 limit 值，我们就会在 componentDidUpdate 中进行检测，并设定新的数字列表。

毫无疑问，上面的代码是可以满足需求的，但是，我们仍然可以进行优化。

上面的代码中，每次 limit 发生改变，我们都会触发两次状态更新：第一次是为了修改 limit，第二次是为了修改展示的数字列表。这样一来，每次 limit 的变化会带来两次 render 开销：

```
// 初始状态
{ limit: 7, number: [0, 1, 2, 3, 4, 5, 6]}
// 更新 limit -> 4
render 1: {limit: 4, number: [0, 1, 2, 3, 4, 5, 6]}
rendre 2: {limit: 2, number: [0, 2, 3]}
```

我们的代码逻辑带来了下面的问题：

我们触发了比实际需要更多的状态更新；
我们出现了「不连续」的渲染结果，即数字列表与 limit 不匹配。
为了改进，我们应避免在不同的状态更新中改变数字列表。事实上，我们可以在一次状态更新中搞定：

```
import React, {useState} from 'react';
import ReactDom from 'react-dom';

function getRange(limit){
  let range = [];
  for (let i =0;i<limit;i++) {
    range.push(i)
  }
  return range;
}

class App extends React.Component {
  state = {
    numbers: [1,2,3,4,5,6],
    limit: 7
  }
  handleLimitChange = e=> {
    const limit = e.target.value;
    const limitChanged = limit !== this.state.limit;
    if (limitChanged) {
      this.setState({limit,numbers: getRange(limit)})
    }
  }

  render(){
    return (
      <div>
        <input
          onChange={this.handleLimitChange}
          placeholder="limit"
          value={this.state.limit}
        />
        {this.state.number.map((number,idx) => {
          return (<p key={idx}>{number}</p>)
        })}
      </div>
    )
  }
}

const rootElement = document.getElementById("root")
ReactDOM.render(<App/>, rootElement)
```

# 使用 PureComponent 和 React.memo 以避免不必要的 render 调用

我们在之前的例子中看到将特定状态值放到更低的层级来避免不必要渲染的方法，不过这并不总是有用。

我们在之前的例子中看到将特定状态值放到更低的层级来避免不必要渲染的方法，不过这并不总是有用。

```
import React, {useState} from 'react';
import ReactDom from 'react-dom';

function App () {
  const [isFooVisible, setFooVisibility] = useState(false);

  return (
    <div className="App">
    {
      isFooVisible ? (
        <Foo hidefoo={() => setFooVisibility(false)} />
      ) : (
        <button onClick={() => setFooVisibility(true)}> Show Foo </button>
      )
    }
    <Bar name="Bar">
    </div>
  )
}

function Foo ({hideFoo}){
  return (
    <>
      <h1>Foo</h1>
      <button onClick={hideFoo}>Hide Foo</button>
    </>
  )
}

function Bar({name}) {
  return <h1>{name}</h1>;
}
const rootElement = document.getElementById("root")
ReactDOM.render(<App/>, rootElement)

```

# 更好的 props 写法

不传递对象作为 props，而是将对象拆分成基本类型：
对于传递箭头函数的场景，我们可以代以只唯一声明过一次的函数，从而总可以拿到相同的引用

```

```

# 控制更新

是那句话，任何方法总有其适用范围。

第三条建议虽然处理了不必要的更新问题，但我们也不总能使用它。

而第四条，在某些情况下我们并不能拆分对象，如果我们传递了某种嵌套确实复杂的数据结构，那我们也很难将其拆分开来。

不仅如此，我们也不总能传递只声明了一次的函数。比如在我们的例子中，如果 App 是个函数式组件，恐怕就不能做到这一点了（在 class 组件中，我们可以用 bind 或者类内箭头函数来保证 this 的指向及唯一声明，而在函数式组件中则可能会有些问题）。

幸运的是，无论是 class 组件还是函数式组件，我们都有办法控制浅比较的逻辑。

在 class 组件中，我们可以使用生命周期钩子 shouldComponentUpdate(prevProps, prevState) 来返回一个布尔值，当返回值为 true 时才会触发 render。

而如果我们使用 React.memo，我们可以传递一个比较函数作为第二个参数。

```
React.memo 的第二参数（比较函数）和 shouldComponentUpdate 的逻辑是相反的，只有当返回值为 false 的时候才会触发 render。参考文档。
```
