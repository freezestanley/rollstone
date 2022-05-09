# https://juejin.cn/post/7062460499576750093?utm_source=gold_browser_extension#heading-22

> useEffect 使用总结

模拟 componentDidMount - useEffect 依赖 [] ；
模拟 componentDidUpdate - useEffect 无依赖，或者依赖 [a, b] ；
模拟 componentWillUnMount - useEffect 中返回一个函数。

> useEffect 让纯函数有了副作用

默认情况下，执行纯函数的时候，只需要输入参数并返回结果即可，是没有任何副作用的；
所谓副作用，就是对函数之外造成影响，如设置全局定时任务；
而组件需要副作用，所以需要 useEffect “钩” 到纯函数中；
因此， useEffect 这个副作用并不是一件坏事。

> useEffect 中返回函数 fn

当 useEffect 依赖于 [] 的时候，在组件销毁时执行 return 的函数 fn ，等于 WillUnMount 生命周期；
当 useEffect 无依赖或依赖于 [a, b] 的时候，组件是在更新时执行 return 的函数 fn ；即在下一次执行 useEffect 之前，就会先执行 fn ，此时模拟的是 DidUpdate 生命周期。

class 组件有三种逻辑复用形式。分别是：

Mixin
高阶组件 HOC
Render Prop

下面说下它们三者各自的缺点。
（1）Mixin

变量作用域来源不清
属性重名
mixins 引入过多会导致顺序冲突

（2）高阶组件 HOC

组件层级嵌套过多，不易渲染，不易调试
HOC 会劫持 props ，必须严格规范，容易出现疏漏

（3）Render Prop

学习成本高，不易理解
只能传递纯函数，而默认情况下纯函数功能有限

了解了三种 class 组件的缺点之后，现在，我们来看下如何使用 Hooks 做组件逻辑复用。

```
import { useState, useEffect } from 'react'
import axios from 'axios'

// 封装 axios 发送网络请求的自定义 Hook
function useAxios(url, config = {}) {
    const [loading, setLoading] = useState(false)
    const [data, setData] = useState()
    const [error, setError] = useState()

    useEffect(() => {
        // 利用 axios 发送网络请求
        setLoading(true)
        axios(url, config) // 发送一个 get 请求
            .then(res => setData(res))
            .catch(err => setError(err))
            .finally(() => setLoading(false))
    }, [url, config])

    return [loading, data, error]
}

export default useAxios

```

当 config 使用的是引用数据类型时，也就是 config={} 或者 confgi=[] 时， useEffect 会出现死循环，无限的发送请求
只能传基本数据类型的

# Hooks 调用顺序必须保持一致

```
import React, { useState, useEffect } from 'react'

function Teach({ couseName }) {
    // 函数组件，纯函数，执行完即销毁
    // 所以，无论组件初始化（render）还是组件更新（re-render）
    // 都会重新执行一次这个函数，获取最新的组件
    // 这一点和 class 组件不一样

    // render: 初始化 state 的值 '张三'
    // re-render: 读取 state 的值 '张三'
    const [studentName, setStudentName] = useState('张三')

    // if (couseName) {
    //     const [studentName, setStudentName] = useState('张三')
    // }

    // render: 初始化 state 的值 '双越'
    // re-render: 读取 state 的值 '双越'
    const [teacherName, setTeacherName] = useState('星期一研究室')

    // if (couseName) {
    //     useEffect(() => {
    //         // 模拟学生签到
    //         localStorage.setItem('name', studentName)
    //     })
    // }

    // render: 添加 effect 函数
    // re-render: 替换 effect 函数（内部的函数也会重新定义）
    useEffect(() => {
        // 模拟学生签到
        localStorage.setItem('name', studentName)
    })

    // render: 添加 effect 函数
    // re-render: 替换 effect 函数（内部的函数也会重新定义）
    useEffect(() => {
        // 模拟开始上课
        console.log(`${teacherName} 开始上课，学生 ${studentName}`)
    })

    return <div>
        课程：{couseName}，
        讲师：{teacherName}，
        学生：{studentName}
    </div>
}

export default Teach
```

上面的代码演示的是一个函数组件。那对于函数组件来说，它是一个纯函数，一般执行完就会销毁。因此，无论是组件初始化（render）还是组件更新（re-render），都会重新执行一次这个函数，获取最新的组件。这一点和 class 组件不一样。
因此，假设我们把上面的 const [studentName, setStudentName] = useState('张三') 给用 if 语句包裹起来，那么它在执行时会受到阻碍。同时，沿着底下的顺序走下去，有可能就会把 张三 这个值，赋值给 teacherName 。因此，在 react-hooks 中，我们不能将 react-hooks 用在循环和判断里面，不然会很容易导致调用顺序错乱。
依据以上的解析，我们来对这个规则做个小结：

无论是 render 还是 re-render ， Hooks 调用顺序必须是一致的；
如果 Hooks 出现在循环、判断里，则无法保证顺序一致；
Hooks 严重依赖于调用顺序！重要！


```
const App = () => {
  const [temp, setTemp] = React.useState(5);

  const log = () => {
    setTimeout(() => {
      console.log("3 秒前 temp = 5，现在 temp =", temp);
    }, 3000);
  };

  return (
    <div
      onClick={() => {
        log();
        setTemp(3);
        // 3 秒前 temp = 5，现在 temp = 5
      }}
    >
      xyz
    </div>
  );
};

```
temp 输出为5
在 log 函数执行的那个 Render 过程里，temp 的值可以看作常量 5，执行 setTemp(3) 时会交由一个全新的 Render 渲染，所以不会执行 log 函数。而 3 秒后执行的内容是由 temp 为 5 的那个 Render 发出的，所以结果自然为 5

可以认为 ref 在所有 Render 过程中保持着唯一引用，因此所有对 ref 的赋值或取值，拿到的都只有一个最终状态，而不会在每个 Render 间存在隔离

```
function Example() {
  const [count, setCount] = useState(0);
  const latestCount = useRef(count);

  useEffect(() => {
    // Set the mutable latest value
    latestCount.current = count;
    setTimeout(() => {
      // Read the mutable latest value
      console.log(`You clicked ${latestCount.current} times`);
    }, 3000);
  });
  // ...
}
```
也可以简洁的认为，ref 是 Mutable 的，而 state 是 Immutable 的。

虽然 React 在 DOM 渲染时会 diff 内容，只对改变部分进行修改，而不是整体替换，但却做不到对 Effect 的增量修改识别。因此需要开发者通过 useEffect 的第二个参数告诉 React 用到了哪些外部变量

```
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);

  return <h1>{count}</h1>;
}
```
“组件初始化执行一次 setInterval，销毁时执行一次 clearInterval，这样的代码符合预期。” 你心里可能这么想。
但是你错了，由于 useEffect 符合 Capture Value 的特性，拿到的 count 值永远是初始化的 0。相当于 setInterval 永远在 count 为 0 的 Scope 中执行，你后续的 setCount 操作并不会产生任何作用
```
useEffect(() => {
  const id = setInterval(() => {
    setCount(c => c + 1);
  }, 1000);
  return () => clearInterval(id);
}, []);
```
setCount 还有一种函数回调模式，你不需要关心当前值是什么，只要对 “旧的值” 进行修改即可。这样虽然代码永远运行在第一次 Render 中，但总是可以访问到最新的 state。

# 将更新与动作解耦

你可能发现了，上面投机取巧的方式并没有彻底解决所有场景的问题，比如同时依赖了两个 state 的情况：
```
useEffect(() => {
  const id = setInterval(() => {
    setCount(c => c + step);
  }, 1000);
  return () => clearInterval(id);
}, [step]);
```
你会发现不得不依赖 step 这个变量，我们又回到了 “诚实的代价” 那一章。当然 Dan 一定会给我们解法的。

利用 useEffect 的兄弟 useReducer 函数，将更新与动作解耦就可以了：

```
const [state, dispatch] = useReducer(reducer, initialState);
const { count, step } = state;

useEffect(() => {
  const id = setInterval(() => {
    dispatch({ type: "tick" }); // Instead of setCount(c => c + step);
  }, 1000);
  return () => clearInterval(id);
}, [dispatch]);
```
这就是一个局部 “Redux”，由于更新变成了 dispatch({ type: "tick" }) 所以不管更新时需要依赖多少变量，在调用更新的动作里都不需要依赖任何变量。 具体更新操作在 reducer 函数里写就可以了

```
import React, { useReducer, useEffect } from "react";
import ReactDOM from "react-dom";

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  const { count, step } = state;

  useEffect(() => {
    const id = setInterval(() => {
      dispatch({ type: 'tick' });
    }, 1000);
    return () => clearInterval(id);
  }, [dispatch]);

  return (
    <>
      <h1>{count}</h1>
      <input value={step} onChange={e => {
        dispatch({
          type: 'step',
          step: Number(e.target.value)
        });
      }} />
    </>
  );
}

const initialState = {
  count: 0,
  step: 1,
};

function reducer(state, action) {
  const { count, step } = state;
  if (action.type === 'tick') {
    return { count: count + step, step };
  } else if (action.type === 'step') {
    return { count, step: action.step };
  } else {
    throw new Error();
  }
}

const rootElement = document.getElementById("root");
ReactDOM.render(<Counter />, rootElement);
```

如果非要把 Function 写在 Effect 外面呢？

```
function Parent() {
  const [query, setQuery] = useState("react");

  // ✅ Preserves identity until query changes
  const fetchData = useCallback(() => {
    const url = "https://hn.algolia.com/api/v1/search?query=" + query;
    // ... Fetch data and return it ...
  }, [query]); // ✅ Callback deps are OK

  return <Child fetchData={fetchData} />;
}

function Child({ fetchData }) {
  let [data, setData] = useState(null);

  useEffect(() => {
    fetchData().then(setData);
  }, [fetchData]); // ✅ Effect deps are OK

  // ...
}
```
经过 useCallback 包装过的函数可以当作普通变量作为 useEffect 的依赖。useCallback 做的事情，就是在其依赖变化时，返回一个新的函数引用，触发 useEffect 的依赖变化，并激活其重新执行

