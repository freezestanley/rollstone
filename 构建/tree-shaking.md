# Tree shaking
sideEffects 和 usedExports（更多被认为是 tree shaking）是两种不同的优化方式
sideEffects 更为有效 是因为它允许跳过整个模块/文件和整个文件子树

usedExports 依赖于 terser 去检测语句中的副作用。它是一个 JavaScript 任务而且没有像 sideEffects 一样简单直接。而且它不能跳转子树/依赖由于细则中说副作用需要被评估

可以通过 /*#__PURE__*/ 注释来帮忙 terser。它给一个语句标记为没有副作用。就这样一个简单的改变就能够使下面的代码被 tree-shake:

```
var Button$1 = /*#__PURE__*/ withAppProvider()(Button);
```

可以告诉 webpack 一个函数调用是无副作用的，只要通过 /*#__PURE__*/ 注释。它可以被放到函数调用之前，用来标记它们是无副作用的(pure)

package.json 的 "sideEffects" 属性
```
{
  "name": "your-project",
  "sideEffects": false,
  "sideEffects": ["./src/some-side-effectful-file.js"]
}
```


“Tree shaking 是一种通过清除多余代码方式来优化项目打包体积的技术，专业术语叫 Dead code elimination

在ES6以前，我们可以使用CommonJS引入模块：require()，这种引入是动态的，也意味着我们可以基于条件来导入需要的代码：

let dynamicModule;// 动态导入
if (condition) { 
  myDynamicModule = require("foo");
} else { 
  myDynamicModule = require("bar");
}
但是CommonJS规范无法确定在实际运行前需要或者不需要某些模块，所以CommonJS不适合tree-shaking机制。
在 ES6 中，引入了完全静态的导入语法：import。
这也意味着下面的导入是不可行的

// 不可行，ES6 的import是完全静态的
```
if (condition) { 
  myDynamicModule = require("foo");
} else { 
  myDynamicModule = require("bar");
}
```
我们只能通过导入所有的包后再进行条件获取。如下：
```
import foo from"foo";
import bar from"bar";
if (condition) {
  // foo.xxxx
} else {
  // bar.xxx
}
```
ES6的import语法可以完美使用tree shaking，因为可以在代码不运行的情况下就能分析出不需要的代码。

看完上面的分析，你可能还是有点懵，
这里我简单做下总结：
因为tree shaking只能在静态modules下工作。
ECMAScript 6 模块加载是静态的,因此整个依赖树可以被静态地推导出解析语法树。
所以在 ES6 中使用 tree shaking 是非常容易的。

# tree shaking的原理是什么?
1.ES6 Module引入进行静态分析，故而编译的时候正确判断到底加载了那些模块
2.静态分析程序流，判断那些模块和变量未被使用或者引用，进而删除对应代码

# common.js 和 es6 中模块引入的区别？

CommonJS 是一种模块规范，最初被应用于 Nodejs，成为 Nodejs 的模块规范。运行在浏览器端的 JavaScript 由于也缺少类似的规范，在 ES6 出来之前，前端也实现了一套相同的模块规范 (例如: AMD)，用来对前端模块进行管理。自 ES6 起，引入了一套新的 ES6 Module 规范，在语言标准的层面上实现了模块功能，而且实现得相当简单，有望成为浏览器和服务器通用的模块解决方案。但目前浏览器对 ES6 Module 兼容还不太好，我们平时在 Webpack 中使用的 export 和 import，会经过 Babel 转换为 CommonJS 规范。在使用上的差别主要有：

1、CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。

2、CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。

3、CommonJs 是单个值导出，ES6 Module可以导出多个

4、CommonJs 是动态语法可以写在判断里，ES6 Module 静态语法只能写在顶层

5、CommonJs 的 this 是当前模块，ES6 Module的 this 是 undefined

# es module
出现之前还有社区推出amd和cmd的规范

在浏览器端使用 es module ，首先在 html 当中引入 js 文件的时候，就需要将script标签中的type设置为module

```
index.html  
<script src="b.js" type="module"></script>
```
> 第一种，直接导出定义的变量
```
a.js
export const name = 'alice'
export const age = 16
 
b.js
import { name, age } from "./a.js";
```

> 第二种，先定义变量，再使用 export 一起导出、通过一个 * 来将所有的导出内容定义为一个对象，从对象中再去取值
```
a.js
const name = 'alice'
const age = 16
export { name, age }
 
b.js
import * as obj from "./a.js";
console.log(obj.name, obj.age) // alice 16

```

> 第三种，给导出的变量取别名
```
a.js
const name = 'alice'
const age = 16
 
export { name as myName, age as myAge }
 
b.js
import { myName as aliceName, myAge } from "./a.js";
 
console.log(aliceName, myAge)  // alice 16
```

> 第四种，默认导出
```
a.js
export default function(){
    return 'hello world'
}
 
b.js
import foo from './a.js'
 
console.log(foo()) // hello world
```

> 第五种，合并导出
```
a.js
export const name = 'alice'
 
b.js
export { name } from './a.js'
 
index.js
import { name } from './b.js'
 
console.log(name)
```

# 特点
1、异步加载，当script标签中定义 type="module"之后，相当于给js标签加上了 async 的标识，代表异步加载资源，不会阻塞其它内容的执行，按照如下代码，打印的hi有可能是在引入的index.js文件之前，要根据 index.js 的执行速度来判断。

2、编译时解析，简单来说javascript的执行过程需要将原代码编译成抽象语法树，运行的时候再转成机器可识别的语言，在编译阶段解析数据，并不知道该不该加载此js文件，只有等到文件运行时，才知道文件里具体逻辑的执行过程，所以不能够在编译时解析的模块化方式出现类似条件判断，动态引入等代码
```
const flag = true
 
if(flag){
    import xxx from './a.js'
}
```

3、export 关键字后面跟的大括号并不是代表对象，在对象中也没有通过 as 取别名这样的方式，如果我们尝试以下把它当成对象来导出，一定是会报错的

```
let name = 'alice'
export {
    name: name
}
```
import 导入的内容是一个通过 const 定义的常量，常量是不可以被修改的，以下操作是不可行的

```
import { name } from './a.js'
 
name = '哈哈哈哈'
```

# 在浏览器端使用 es module，如果想要在 node 端编写es module代码，可以有两种方式，
1）一种是在 package.json 中配置 type：module
2）一种是直接把js文件的后缀名为改为 .mjs

# nodejs端模块化方式comomjs详解
通过 module.exports 或者 exports 导出，通过 require函数来导入
```
// a.js 导出内容
const name = 'alice'
const age = 16
module.exports = {
  name,
  age
}
 
// b.js 导入
const obj = require('./a.js')
console.log(obj) // { name: 'alice', age: 16}
```
module.exports和exports 导出不同的书写方式

```
// a.js
const name = 'alice'
const age = 16
exports.age = age
module.exports.name = name
 
// b.js
const obj = require('./a.js')
console.log(obj) // { name: 'kiki', age: 16 }
```
两者的不同之处在于，即使通过 exports.xxx 导出，其实最终也是通过 module.exports 这样的方式进行导出的，因为在node内部，初始化时先将一个空对象赋值给 exports，然后再将exports赋值给module.exports，所以通过 exports.xxx 这样的形式就相当于在 module.exports 这个对象里面追加子元素

```

// a.js
const name = 'alice'
const age = 16
const hobby = 'singing'
 
exports.name = name
exports.age = age
 
module.exports = {
   hobby
}
 
// b.js
const obj = require('./a.js')
console.log(obj) // { hobby: 'singing' }
```
重新将一个对象赋值给了 module.exports，module.exports 指向的内存空间已经和 exports 不是同一个指向，所以最终的导出只有 module.exports 部分



# 有react fiber，为什么不需要 vue fiber呢、之前递归遍历虚拟dom树被打断就得从头开始，为什么有了react fiber就能断点恢复呢

react调用setState去修改数据，之后页面便会重新渲染
vue直接this.state=xxx修改，页面就会展示最新的数据

# react、vue的响应式原理

react需要调用setState方法，而vue直接修改变量就行 响应式原理正在于此

在react中，组件的状态是不能被修改的，setState没有修改原来那块内存中的变量，而是去新开辟一块内存；而vue则是直接修改保存状态的那块原始内存。

所以经常能看到react相关的文章里经常会出现一个词"immutable"，翻译过来就是不可变的

数据修改了，接下来要解决视图的更新：react中，调用setState方法后，会自顶向下重新渲染组件，自顶向下的含义是，该组件以及它的子组件全部需要渲染；而vue使用Object.defineProperty（vue@3迁移到了Proxy）对数据的设置（setter）和获取（getter）做了劫持，也就是说，vue能准确知道视图模版中哪一块用到了这个数据，并且在这个数据修改时，告诉这个视图，你需要重新渲染了