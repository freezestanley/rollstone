> 使用 shouldComponentUpdate 对组件性能进行优化

shouldComponentUpdate 非常关键。
这个函数是 re-render 是 render()函数调用前被调用的，他的两个参数 nextProps 和 nextState，分别表示下一个 props 和下一个 state 的值。我们重写这个钩子，当函数返回 false 时，阻止接下来的 render()调用以及组件重新渲染，反之，返回 true 时，组件向下走 render 重新渲染。

> 父组件

```
shouldComponentUpdate(nextProps, nextState) {
  if(nextState.name !== this.state.name) {
    return true
  }
  return false
}
```

> 子组件

```
shouldComponentUpdate(nextProps, nextState) {
  if(nextState.son !== this.state.son) {
    return true
  }
  return false
}

```

（1）使用 setState 改变数据之前，先采用 es6 中 assgin 进行拷贝，但是 assgin 只深拷贝的数据的第一层，所以说不是最完美的解决办法。

```
const o2 = Object.assgin({}, this.state.obj)
o2.student.count ='000000'
this.setState({obj: o2})
```

（2）使用 JSON.parse(JSON.stringfy())进行深拷贝，但是遇到数据为 undefined 和函数时就会错。

```
const o2 =JSON.parse(JSON.stringify(this.state.obj))
o2.student.count ='000000'
this.setState({
  obj:o2
})
```

（3）使用 immutable.js 进行项目的搭建。immutable 中讲究数据的不可变性，每次对数据进行操作前，都会自动的对数据进行深拷贝，项目中数据采用 immutable 的方式，可以轻松解决问题，但是又多了一套 API 去学习。
