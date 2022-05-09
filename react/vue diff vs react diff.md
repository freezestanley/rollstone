# Diff vue
> 什么是虚拟dom
```
<ul id="list">
    <li class="item">哈哈</li>
    <li class="item">呵呵</li>
    <li class="item">嘿嘿</li>
</ul>

let oldVDOM = { // 旧虚拟DOM
        tagName: 'ul', // 标签名
        props: { // 标签属性
            id: 'list'
        },
        children: [ // 标签子节点
            {
                tagName: 'li', props: { class: 'item' }, children: ['哈哈']
            },
            {
                tagName: 'li', props: { class: 'item' }, children: ['呵呵']
            },
            {
                tagName: 'li', props: { class: 'item' }, children: ['嘿嘿']
            },
        ]
    }
```
1.虚拟dom 是在内存中的用以描述dom的数据
虚拟DOM算法 = 虚拟DOM + Diff算法
数据修改 --> 虚拟dom --> 真实dom
数据修改 --> 真实dom

虚拟DOM比真实DOM快 不严谨 中间多了一个虚拟dom的过程

![avatar](https://github.com/freezestanley/rollstone/blob/main/react/640_2.png)

Diff算法是一种对比算法。对比两者是旧虚拟DOM和新虚拟DOM，对比出是哪个虚拟节点更改了，找出这个虚拟节点，并只更新这个虚拟节点所对应的真实节点，而不用更新其他数据没发生改变的节点，实现精准地更新真实DOM，进而提高效率。

使用虚拟DOM算法的损耗计算：总损耗 = 虚拟DOM增删改+（与Diff算法效率有关）真实DOM差异增删改+（较少的节点）排版与重绘

直接操作真实DOM的损耗计算：总损耗 = 真实DOM完全增删改+（可能较多的节点）排版与重绘

# Diff算法的原理

Diff同层对比

新旧虚拟DOM对比的时候，Diff算法比较只会在同层级进行, 不会跨层级比较。所以Diff算法是:广度优先算法。 时间复杂度:O(n)

![avatar](https://github.com/freezestanley/rollstone/blob/main/react/640_3.png)

Diff对比流程
当数据改变时，会触发setter，并且通过Dep.notify去通知所有订阅者Watcher，订阅者们就会调用patch方法，给真实DOM打补丁，更新相应的视图

newVnode和oldVnode：同层的新旧虚拟节点

![avatar](https://github.com/freezestanley/rollstone/blob/main/react/640_4.png)

# patch方法

这个方法作用就是，对比当前同层的虚拟节点是否为同一种类型的标签

是：继续执行patchVnode方法进行深层比对

否：没必要比对了，直接整个节点替换成新虚拟节点

patch的核心原理代码 sameVnode 比较重要

```
function patch(oldVnode, newVnode) {
  // 比较是否为一个类型的节点
  if (sameVnode(oldVnode, newVnode)) {
    // 是：继续进行深层比较
    patchVnode(oldVnode, newVnode)
  } else {
    // 否
    const oldEl = oldVnode.el // 旧虚拟节点的真实DOM节点
    const parentEle = api.parentNode(oldEl) // 获取父节点
    createEle(newVnode) // 创建新虚拟节点对应的真实DOM节点
    if (parentEle !== null) {
      api.insertBefore(parentEle, vnode.el, api.nextSibling(oEl)) // 将新元素添加进父元素
      api.removeChild(parentEle, oldVnode.el)  // 移除以前的旧元素节点
      // 设置null，释放内存
      oldVnode = null
    }
  }

  return newVnode
}
```

# sameVnode方法

patch关键的一步就是sameVnode方法判断是否为同一类型节点

```
function sameVnode(oldVnode, newVnode) {
  return (
    oldVnode.key === newVnode.key && // key值是否一样
    oldVnode.tagName === newVnode.tagName && // 标签名是否一样
    oldVnode.isComment === newVnode.isComment && // 是否都为注释节点
    isDef(oldVnode.data) === isDef(newVnode.data) && // 是否都定义了data
    sameInputType(oldVnode, newVnode) // 当标签为input时，type必须是否相同
  )
}
```

# patchVnode方法
这个函数做了以下事情：

找到对应的真实DOM，称为el
判断newVnode和oldVnode是否指向同一个对象，如果是，那么直接return
如果他们都有文本节点并且不相等，那么将el的文本节点设置为newVnode的文本节点。
如果oldVnode有子节点而newVnode没有，则删除el的子节点
如果oldVnode没有子节点而newVnode有，则将newVnode的子节点真实化之后添加到el
如果两者都有子节点，则执行updateChildren函数比较子节点，这一步很重要

```
function patchVnode(oldVnode, newVnode) {
  const el = newVnode.el = oldVnode.el // 获取真实DOM对象
  // 获取新旧虚拟节点的子节点数组
  const oldCh = oldVnode.children, newCh = newVnode.children
  // 如果新旧虚拟节点是同一个对象，则终止
  if (oldVnode === newVnode) return
  // 如果新旧虚拟节点是文本节点，且文本不一样
  if (oldVnode.text !== null && newVnode.text !== null && oldVnode.text !== newVnode.text) {
    // 则直接将真实DOM中文本更新为新虚拟节点的文本
    api.setTextContent(el, newVnode.text)
  } else {
    // 否则

    if (oldCh && newCh && oldCh !== newCh) {
      // 新旧虚拟节点都有子节点，且子节点不一样

      // 对比子节点，并更新
      updateChildren(el, oldCh, newCh)
    } else if (newCh) {
      // 新虚拟节点有子节点，旧虚拟节点没有

      // 创建新虚拟节点的子节点，并更新到真实DOM上去
      createEle(newVnode)
    } else if (oldCh) {
      // 旧虚拟节点有子节点，新虚拟节点没有

      //直接删除真实DOM里对应的子节点
      api.removeChild(el)
    }
  }
}
```

# updateChildren方法

这是patchVnode里最重要的一个方法，新旧虚拟节点的子节点对比，就是发生在updateChildren方法中，接下来就结合一些图来讲，让大家更好理解吧

是怎么样一个对比方法呢？就是首尾指针法，新的子节点集合和旧的子节点集合，各有首尾两个指针，举个例子：
```
<ul>
    <li>a</li>
    <li>b</li>
    <li>c</li>
</ul>

修改数据后

<ul>
    <li>b</li>
    <li>c</li>
    <li>e</li>
    <li>a</li>
</ul>
```
那么新旧两个子节点集合以及其首尾指针为：
![avatar](https://github.com/freezestanley/rollstone/blob/main/react/640_5.png)

然后会进行互相进行比较，总共有五种比较情况：

1、oldS 和 newS使用sameVnode方法进行比较，sameVnode(oldS, newS)
2、oldS 和 newE使用sameVnode方法进行比较，sameVnode(oldS, newE)
3、oldE 和 newS使用sameVnode方法进行比较，sameVnode(oldE, newS)
4、oldE 和 newE使用sameVnode方法进行比较，sameVnode(oldE, newE)
5、如果以上逻辑都匹配不到，再把所有旧子节点的 key 做一个映射到旧节点下标的 key -> index 表，然后用新 vnode 的 key 去找出在旧节点中可以复用的位置。

![avatar](https://github.com/freezestanley/rollstone/blob/main/react/640_6.png)

接下来就以上面代码为例，分析一下比较的过程

分析之前，请大家记住一点，最终的渲染结果都要以newVDOM为准，这也解释了为什么之后的节点移动需要移动到newVDOM所对应的位置

![avatar](https://github.com/freezestanley/rollstone/blob/main/react/640_8.png)

> 第一步
```
oldS = a, oldE = c
newS = b, newE = a
```
比较结果：oldS 和 newE 相等，需要把节点a移动到newE所对应的位置，也就是末尾，同时oldS++，newE--
![avatar](https://github.com/freezestanley/rollstone/blob/main/react/640_9.png)

> 第二步
```
oldS = b, oldE = c
newS = b, newE = e
```
比较结果：oldS 和 newS相等，需要把节点b移动到newS所对应的位置，同时oldS++,newS++
![avatar](https://github.com/freezestanley/rollstone/blob/main/react/640_10.png)

> 第三步
```
oldS = c, oldE = c
newS = c, newE = e
```
比较结果：oldS、oldE 和 newS相等，需要把节点c移动到newS所对应的位置，同时oldS++,oldE--,newS++
![avatar](https://github.com/freezestanley/rollstone/blob/main/react/640_11.png)

> 第四步 oldS > oldE，则oldCh先遍历完成了，而newCh还没遍历完，说明newCh比oldCh多，所以需要将多出来的节点，插入到真实DOM上对应的位置上
![avatar](https://github.com/freezestanley/rollstone/blob/main/react/640_12.png)
updateChildren的核心原理代码
```
function updateChildren(parentElm, oldCh, newCh) {
  let oldStartIdx = 0, newStartIdx = 0
  let oldEndIdx = oldCh.length - 1
  let oldStartVnode = oldCh[0]
  let oldEndVnode = oldCh[oldEndIdx]
  let newEndIdx = newCh.length - 1
  let newStartVnode = newCh[0]
  let newEndVnode = newCh[newEndIdx]
  let oldKeyToIdx
  let idxInOld
  let elmToMove
  let before
  while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
    if (oldStartVnode == null) {
      oldStartVnode = oldCh[++oldStartIdx]
    } else if (oldEndVnode == null) {
      oldEndVnode = oldCh[--oldEndIdx]
    } else if (newStartVnode == null) {
      newStartVnode = newCh[++newStartIdx]
    } else if (newEndVnode == null) {
      newEndVnode = newCh[--newEndIdx]
    } else if (sameVnode(oldStartVnode, newStartVnode)) {
      patchVnode(oldStartVnode, newStartVnode)
      oldStartVnode = oldCh[++oldStartIdx]
      newStartVnode = newCh[++newStartIdx]
    } else if (sameVnode(oldEndVnode, newEndVnode)) {
      patchVnode(oldEndVnode, newEndVnode)
      oldEndVnode = oldCh[--oldEndIdx]
      newEndVnode = newCh[--newEndIdx]
    } else if (sameVnode(oldStartVnode, newEndVnode)) {
      patchVnode(oldStartVnode, newEndVnode)
      api.insertBefore(parentElm, oldStartVnode.el, api.nextSibling(oldEndVnode.el))
      oldStartVnode = oldCh[++oldStartIdx]
      newEndVnode = newCh[--newEndIdx]
    } else if (sameVnode(oldEndVnode, newStartVnode)) {
      patchVnode(oldEndVnode, newStartVnode)
      api.insertBefore(parentElm, oldEndVnode.el, oldStartVnode.el)
      oldEndVnode = oldCh[--oldEndIdx]
      newStartVnode = newCh[++newStartIdx]
    } else {
      // 使用key时的比较
      if (oldKeyToIdx === undefined) {
        oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx) // 有key生成index表
      }
      idxInOld = oldKeyToIdx[newStartVnode.key]
      if (!idxInOld) {
        api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el)
        newStartVnode = newCh[++newStartIdx]
      }
      else {
        elmToMove = oldCh[idxInOld]
        if (elmToMove.sel !== newStartVnode.sel) {
          api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el)
        } else {
          patchVnode(elmToMove, newStartVnode)
          oldCh[idxInOld] = null
          api.insertBefore(parentElm, elmToMove.el, oldStartVnode.el)
        }
        newStartVnode = newCh[++newStartIdx]
      }
    }
  }
  if (oldStartIdx > oldEndIdx) {
    before = newCh[newEndIdx + 1] == null ? null : newCh[newEndIdx + 1].el
    addVnodes(parentElm, before, newCh, newStartIdx, newEndIdx)
  } else if (newStartIdx > newEndIdx) {
    removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
  }
}
```

用index做key

平常v-for循环渲染的时候，为什么不建议用index作为循环项的key呢？

我们举个例子，左边是初始数据，然后我在数据前插入一个新数据，变成右边的列表

```
<ul>                      <ul>
    <li key="0">a</li>        <li key="0">林三心</li>
    <li key="1">b</li>        <li key="1">a</li>
    <li key="2">c</li>        <li key="2">b</li>
                              <li key="3">c</li>
</ul>                     </ul>
```

```
<ul>
   <li v-for="(item, index) in list" :key="index">{{ item.title }}</li>
</ul>
<button @click="add">增加</button>

list: [
        { title: "a", id: "100" },
        { title: "b", id: "101" },
        { title: "c", id: "102" },
      ]
      
add() {
      this.list.unshift({ title: "林三心", id: "99" });
    }
```

点击按钮我们可以看到，并不是我们预想的结果，而是所有li标签都更新了

按理说，a，b，c三个li标签都是复用之前的，因为他们三个根本没改变，改变的只是前面新增了一个林三心

![avatar](https://github.com/freezestanley/rollstone/blob/main/react/640_13.png)


在进行子节点的 diff算法 过程中，会进行 旧首节点和新首节点的sameNode对比，这一步命中了逻辑，因为现在新旧两次首部节点 的 key 都是 0了，同理，key为1和2的也是命中了逻辑，导致相同key的节点会去进行patchVnode更新文本，而原本就有的c节点，却因为之前没有key为4的节点，而被当做了新节点，所以很搞笑，使用index做key，最后新增的居然是本来就已有的c节点。所以前三个都进行patchVnode更新文本，最后一个进行了新增，那就解释了为什么所有li标签都更新了。

![avatar](https://github.com/freezestanley/rollstone/blob/main/react/640_14.png)

那我们可以怎么解决呢？其实我们只要使用一个独一无二的值来当做key就行了
```
<ul>
   <li v-for="item in list" :key="item.id">{{ item.title }}</li>
</ul>
```

为什么用了id来当做key就实现了我们的理想效果呢，因为这么做的话，a，b，c节点的key就会是永远不变的，更新前后key都是一样的，并且又由于a，b，c节点的内容本来就没变，所以就算是进行了patchVnode，也不会执行里面复杂的更新操作，节省了性能，而林三心节点，由于更新前没有他的key所对应的节点，所以他被当做新的节点，增加到真实DOM上去了

![avatar](https://github.com/freezestanley/rollstone/blob/main/react/640_15.png)

# 总结VUE Diff算法 https://mp.weixin.qq.com/s/S4S0wiRIiWO2PxGweIpCrA

1.虚拟DOM一定比dom操作快 这个是不严谨的
2.虚拟DOM其实是在内存中以数据形式描述的
3.diff原理
1）Diff算法比较只会在同层级进行, 不会跨层级比较。所以Diff算法是:广度优先算法。 时间复杂度:O(n) react 是深度优先
2）会触发setter，并且通过Dep.notify去通知所有订阅者Watcher，订阅者们就会调用patch方法，给真实DOM打补丁，更新相应的视图
3）patch方法内 newVnode 和 oldVnode对比
4）sameVnode 对比 key tagname data 进行比较
5）在updateChildren方法内 
  采用首尾指针法（S开头E结尾）
1、oldS 和 newS使用sameVnode方法进行比较，sameVnode(oldS, newS)
2、oldS 和 newE使用sameVnode方法进行比较，sameVnode(oldS, newE)
3、oldE 和 newS使用sameVnode方法进行比较，sameVnode(oldE, newS)
4、oldE 和 newE使用sameVnode方法进行比较，sameVnode(oldE, newE)
5、如果以上逻辑都匹配不到，再把所有旧子节点的 key 做一个映射到旧节点下标的 key -> index 表，然后用新 vnode 的 key 去找出在旧节点中可以复用的位置
6）vue在发生变更的时候知道所变化的组件节点 不需要遍历

# 用index做key
平常v-for循环渲染的时候，为什么不建议用index作为循环项的key呢？
进行子节点的 diff算法 过程中，会进行 旧首节点和新首节点的sameNode对比，
这一步命中了逻辑，
因为现在新旧两次首部节点 的 key 都是 0了，
同理，key为1和2的也是命中了逻辑，导致相同key的节点会去进行patchVnode更新文本，
而原本就有的c节点，却因为之前没有key为4的节点，而被当做了新节点，所以很搞笑，
使用index做key，最后新增的居然是本来就已有的c节点。
所以前三个都进行patchVnode更新文本，最后一个进行了新增，那就解释了为什么所有li标签都更新了


