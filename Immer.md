# Immutable数据流React性能优化

Redux中的reducer是基于纯函数设计的，要求返回的状态数据(对象或数组)，需要先深拷贝一份（目的是：防止影响老状态）

immutable数据一种利用结构共享形成的持久化数据结构，一旦有部分被修改，那么将会返回一个全新的对象，并且原来相同的节点会直接共享。
每次修改一个 immutable 对象时都会创建一个新的不可变的对象，在新对象上操作并 不会影响到原对象的数据。
具体点来说，immutable 对象数据内部采用是多叉树的结构，凡是有节点被改变，那么它和与它相关的所有上级节点都更新

采用 immutable 既能够最大效率地更新数据结构，又能够和现有的 React中的 PureComponent (memo) 顺利对接，感知到状态的变化，是提高 React 渲染性能的极佳方案

与React中的 PureComponent(memo) 相结合，我们知道PureComponent能够在内部帮我们比较新props跟旧props，新state和旧state，如果值相等或者对象含有的相同的属性、且属性值相等，便确定shouldComponentUpdate返回true或者false，从而判断是否再次渲染render函数。
看上述代码，我们可以看出来，当代码中使用immutable第三库的时候，可以精确地深拷贝 a 对象，改a对象中的select属性赋值给b之后，并不会影响原对象a，而b的select属性变为了新值。
如果上述select属性给一个组件用，因为其值被改变了，导致shouldComponentUpdate应该返回true，而filter属性给另一个组件用，通过判断，并无变化，导致shouldComponentUpdate应该返回false，故此组件就避免了重复的diff算法对比，大大提高了React中的性能优化。


# immutable优化性能的方式

immutable实现的原理是：持久化数据结构，也就是使用旧数据创建新数据时，要保证旧数据同时可用且不变。同时为了避免deepCopy 把所有节点都复制一遍带来的性能损耗。
immutable使用了结构共享，即如果对象树中一个节点发生变化，只修改这个节点和受它影响的父节点，其它节点则进行共享。






----------------------

Immer 包暴露了一个完成所有工作的默认函数

produce(currentState, recipe: (draftState) => void): nextState

1.(base)state， 传递给 produce 的不可变状态
2.recipe: produce 的第二个参数，它捕获了 base state 应该如何 mutated。
3.draft: 任何 recipe 的第一个参数，它是可以安全 mutate 的原始状态的代理。
4.producer. 一个使用 produce 的函数，通常形式为 (baseState, ...arguments) => resultState

```
produce(state, draft => {
        const todo = draft.find(todo => todo.id === id)
        todo.done = !todo.done
    })

import produce from "immer"

function toggleTodo(state, id) {
    return produce(state, draft => {
        const todo = draft.find(todo => todo.id === id)
        todo.done = !todo.done
    })
}

const baseState = [
    {
        id: "JavaScript",
        title: "Learn TypeScript",
        done: true
    },
    {
        id: "Immer",
        title: "Try Immer",
        done: false
    }
]

const nextState = toggleTodo(baseState, "Immer")

```

将函数作为第一个参数传递给 produce 会创建一个函数，该函数尚未将 produce 应用于特定 state，而是创建一个函数，该函数将应用于将来传递给它的任何 state。这通常称为柯里化


current 提供 draft 当前状态的快照

current 工作生成的对象类似于 produced 本身创建的对象。

未修改的对象将在结构上与原始对象共享。
如果未对 draft 进行任何更改，通常它会保留 original(draft) === current(draft)，但这并不能保证。
未来对 draft 的更改不会反映在 current 生成的对象中（不可被 draft 对象的引用除外）
与 produce 创建的对象不同，current 创建的对象不会被冻结

```
const base = {
    x: 0
}

const next = produce(base, draft => {
    draft.x++
    const orig = original(draft)
    const copy = current(draft)
    console.log(orig.x)
    console.log(copy.x)

    setTimeout(() => {
        // 将在 produce 完成后执行
        console.log(orig.x)
        console.log(copy.x)
    }, 100)

    draft.x++
    console.log(draft.x)
})
console.log(next.x)

// 将会打印
// 0 (orig.x)
// 1 (copy.x)
// 2 (draft.x)
// 2 (next.x)
// 0 (after timeout, orig.x)
// 1 (after timeout, copy.x)
```

# original 从 produce 内部的代理实例获取原始对象（对于未代理值返回 undefined

```
import {original, produce} from "immer"

const baseState = {users: [{name: "Richie"}]}
const nextState = produce(baseState, draftState => {
    original(draftState.users) // is === baseState.users
})
```

# Patches

在 producer 运行期间，Immer 可以记录所有的补丁来回溯 reducer造成的更改 。这是一个非常强大的工具，如果您想暂时 fork 您的状态并回溯对原始状态的更改。

Patches 在下面场景很有用:

与其他方交换增量更新，例如通过
对于调试/跟踪，准确查看状态如何随时间变化
作为撤消/重做的基础或作为在稍微不同的状态树上回溯更改的方法
```
import produce, {applyPatches} from "immer"

// 版本 6
import {enablePatches} from "immer"
enablePatches()

let state = {
    name: "Micheal",
    age: 32
}

// 假设用户在向导中
// 他的更改 应该以最终是否为基本状态结束...

his changes should end up in the base state ultimately or not...
let fork = state
// 用户在向导中所作的所有更改
let changes = []
// 与向导中所做的所有更改相反
let inverseChanges = []

fork = produce(
    fork,
    draft => {
        draft.age = 33
    },
    // 产生的第三个参数是一个回调，patches 将从这里产生
    (patches, inversePatches) => {
        changes.push(...patches)
        inverseChanges.push(...inversePatches)
    }
)

// 同时，我们的原始状态被替换，例如
// 从服务器收到了一些更改
state = produce(state, draft => {
    draft.name = "Michel"
})

// 当向导完成（成功）后，我们可以将 fork 中的更改重播到新的状态！
state = applyPatches(state, changes)

// state 现在包含来自两个代码路径的更改！
expect(state).toEqual({
    name: "Michel", // 服务器更改
    age: 33 // 向导更改
})

// 最后，即使在完成向导之后，用户也可能会改变主意并撤消他的更改......
state = applyPatches(state, inverseChanges)
expect(state).toEqual({
    name: "Michel", // 没有还原
    age: 32 // 还原了
})
```

# produceWithPatches

produceWithPatches，它与 produce 具有相同的签名，不过它 不止返回 next state，而是一个包含 [nextState, patches, inversePatches] 的元组

```
import {produceWithPatches} from "immer"

const [nextState, patches, inversePatches] = produceWithPatches(
    {
        age: 33
    },
    draft => {
        draft.age++
    }
)
```

# 自动冻结
Immer 自动冻结所有使用 produce 修改的任何 state。这可以防止在 producer 之外意外修改 state。在大多数情况下，这是最佳实践，但你可以通过 setAutoFreeze(true / false) 显式打开或关闭此功能。

Immer 永远不会冻结不可枚举、非自己或符号属性的（内容），除非它们的内容是可以被 draft 的

# 使用 nothing 产生 undefined
```
produce({}, draft => {
    // 什么也不干
})

produce({}, draft => {
    // 尝试从 producer 中返回 undefined
    return undefined
})
```
让 Immer 清楚您有意生成 undefined 值，您可以返回内置标记 nothing：

```
import produce, {nothing} from "immer"

const state = {
    hello: "world"
}

produce(state, draft => {})
produce(state, draft => undefined)
// 都会返回最初的状态: { hello: "world"} 

produce(state, draft => nothing)
// 产生一个新的状态, 'undefined'
```

# 使用 void 的内联快捷方式

```
// 单次修改
produce(draft => void (draft.user.age += 1))

// 多次修改
produce(draft => void ((draft.user.age += 1), (draft.user.height = 186)))
```

# 异步 producers & createDraft / finishDraft

允许从 recipe 返回 Promise 对象。或者，换句话说，使用 async / await。这对于长时间运行的进程非常有用，只有在 Promise 链解析后才生成新对象。请注意，如果 producer 是异步的，produce 本身（即使是柯里化形式）也会返回一个 promise。例子:
```
import produce from "immer"

const user = {
    name: "michel",
    todos: []
}

const loadedUser = await produce(user, async function(draft) {
    draft.todos = await (await window.fetch("http://host/" + draft.name)).json()
})
```

# createDraft and finishDraft
```
import {createDraft, finishDraft} from "immer"

const user = {
    name: "michel",
    todos: []
}

const draft = createDraft(user)
draft.todos = await (await window.fetch("http://host/" + draft.name)).json()
const loadedUser = finishDraft(draft)
```
警告：一般情况下，我们建议使用 producer 而不是 createDraft / finishDraft 组合，produce 在使用中不易出错，并且在代码中更清楚地区分了可变性和不变性的概念