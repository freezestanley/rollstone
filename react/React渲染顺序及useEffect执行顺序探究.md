```
import React,{useEffect} from 'react'
export function SubApp({name}) {
  console.log(`${name} mount`)
  useEffect(() => {
    conosle.log(`${name} effect`)
    return () =>{
      console.log(`${name} clean`)
    }
  }, [])
  return <div>{name}</div>
}
```

在 App 组件下渲染两个连续的 SubApp 组件，一个叫A，一个叫B，来看执行结果
App mount --> A mount --> B mount --> A effect --> B effect --> App effect
在 React16 中 （重要！）
mount（挂载）的顺序是 父组件 -> 子组件
effect（副作用）的顺序是 子组件 -> 父组件，而在兄弟（同级）组件中，先渲染的会先执行
18的渲染模式和旧版本是没有区别的，下面，我们需要修改 index.js，采用并发模式下的渲染方法
 App 将会使用并发模式（Concurrent Mode）

 ```
 App mount ✖️2
A mount ✖️2
B mount ✖️2
A effect
B effect
App effect
A clean
B clean
App clean
A effect
B effect
App effect
```

每一个组件都被 mount 了两次
子组件的 effet 首先执行，然后执行父组件的，这里顺序和之前没有不同
子组件的 clean 执行，然后父组件的 clean 也执行，顺序和 effect 执行顺序一致
子组件和父组件的 effect 再次执行，顺序保持不变

为什么会有 clean 执行了，这是因为 React 18 并发模式下会强制让组件更新一次，也就是 clean->effect ，并且会先整体把所有组件清理一遍，再执行所有组件的副作用，而不是穿插着一个个组件交替执行，这是十分有趣的


