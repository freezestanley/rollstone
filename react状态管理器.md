# zustand

1.不需要像redux那样在最外层包裹一层高阶组件，只绑定对应关联组件即可（当在其他组件/方法修改状态后，该组件会自动更新）

2.异步处理也较为简单，与普通函数用法相同

3.支持hook组件使用、组件外使用
提供middleware拓展能力（redux、devtools、combine、persist）

4.可通过 https://github.com/mweststrate/immer 拓展能力（实现嵌套更新、日志打印）

# redux

核心原理：reducer 纯函数
使用 Context API
遵循的是函数式（如函数式编程）的风格
单一的全局存储来保存应用程序的所有状态
更改只通过动作发生
bundle size 小（redux+react-redux约为3kb）

# MobX

核心原理：ES6 proxy （可以理解为vue的双向数据绑定）
MobX是基于观察者/可观察模式的。
以真正的 "反应式 "方式管理状态，因此当你修改一个值时，任何使用该值的组件都会自动重新渲染。
不需要任何动作或者reducers，只需修改你的状态，应用程序就会反映出来。
要求使用ES6代理，意味着不支持IE11及以下版本。（或者旧版本）

# Recoil

与 React 非常相似的简单 API，它的API像React的useState和Context API的组合
通过跟踪对useRecoilState的调用，Recoil可以跟踪哪些组件使用了哪些原子。这样它就可以在数据发生变化时，只重新渲染那些 "订阅 "某项数据的组件，所以这种方法在性能方面应该可以很好地扩展。
与Redux一样需要在最外层提供类似Context Provider包裹的方式

# Dva
对Redux的包装与再次封装，核心原理依然是redux
