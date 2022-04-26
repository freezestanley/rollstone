# commonjs

nodejs 里的规范，环境变量：

module
exports
require
global

每一个文件是一个模块，有自己的作用域。在文件内定义的变量、函数、类都是私有的，对其他文件不可见。
global 是全局变量，多个文件内可以共同分享变量。

commonjs 规定：
每个模块内部，module 变量代表当前模块，该变量是一个对象。他有一个 exports 属性，这个属性是对外的接口。加载某一个模块，其实就是加载该模块的 module.exports 属性。
commonjs 模块的特点：

所有代码运行在模块作用域内，不会污染全局变量
模块可以加载多次，但是只有第一次加载时运行一次。然后运行结果就被缓存下来，以后再加载，就是直接读取缓存的结果。

# Module 对象

> node 内部提供一个 Module 构建函数。每一个模块内部都有一个 module 对象，代表当前模块。它有以下属性：

id 模块标识符，通常是带有绝对路径的模块文件名
path 模块的文件名，绝对路径
exports 模块对外输出的值
parent 调用该模块的模块
children 该模块用到的其他模块
loaded 该模块是否已经加载完（在父模块中 require 一个子模块之后，子模块的 loaded 才变为 true）

module.exports
表示对外输出的接口，其他文件加载该模块其实就是读取 module.exports 变量
exports
为了方便，Node 为每一个模块提供了 exports 变量，指向 module.exports。等同于

```
var exports = module.exports
```

所以切记，不可以直接给 exports 赋值，这样就是切断了 exports 和 module.exports 的联系。

```
exports = () => {}
```

只可以在 exports 上添加属性。
如果该模块对外输出的是一个简单类型的值。那么不可以用 exports 去输出了，只能用 module.exports = 'xx'
加载模式 - 同步/运行时加载
首先，commonjs 加载模块的方式是同步的。在输入时先加载整个模块，生成一个对象。再从这个对象上读取方法，这种加载被成为运行时加载。只有对应子模块加载完成，才能执行后面的操作。
为什么是同步的？因为 Nodejs 是用于服务端编程，模块文件存在于硬盘中，读取非常快。

> 加载时机
> 输入的值是被输出的值的拷贝。
> 父模块引入了一个子模块，其实引入的是这个子模块输出的值的拷贝，一旦输出了这个值，模块内部的变化就影响不到这个值了。

# AMD

非同步加载模块。浏览器环境要加载资源，需要从服务器端加载模块，依靠网络下载，时间比较长。所以需要采用非同步的模块。
AMD 实现者 require.js
AMD 相关的 api
define，用于定义模块，如果我们定义的模块本身也依赖其他模块，那么就需要把它放在数组中，作为第一个参数

```
define(['xx/lib.js'], (lib) => {
  function foo(){
    lib.log('hello world!');
  }
  return {
    foo
  };
})
```

CMD
sea.js
都是非同步加载，
AMD 推崇的是前置依赖，先第一时间将前面数组内的模块都加载
CMD 推崇的是就近依赖，延迟加载

# UMD

是一种思想，兼容 commonjs、AMD、CMD。
先判断是否支持 Nodejs 模块(exports 是否存在)，如果存在就使用 Nodehs 模块。不支持的话，再判断是否支持 AMD/CMD(判断 define 是否存在)。都不行就挂载在 window 全局对象上

```
(function(t, e) {
  if (typeof module === 'object' && module.exports) { // Nodejs 环境
    module.exports = e(require('react'))
  } else if (typeof define === 'function' && define.amd) { // 浏览器环境
    define('react', e)
  } else { // 其他运行环境，比如小程序
    t.xx = e(t.React)
  }
)(window, function () {})
```

# ES6 Module

ES6 在语言标准层面上，实现了模块功能，而且实现的非常简单，宗旨是在浏览器和服务器通用的模块解决方案。其模块功能由两个命令组成：export 和 import。
ES6 模块的特征：

import 是只读属性，不能赋值。相当于 const
export/import 提升，import/export 必须位于模块的顶级，不可以位于作用域内，其次对于模块内的 import/export 都会提升到模块的顶部。

ES6 Module 加载时机
import 是静态命令的方式，js 引擎对脚本静态分析时，遇到模块加载命令 import，会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被记载的那么模块中去取值。模块内部引用的变化会反应在外部。
在 import 时可以指定加载某个输出值，而不是加载整个模块，这种加载称为编译时加载。在编译时就引入模块代码，而不是在代码运行时加载，所以无法实现条件加载。也正因为这个，使得静态分析成为可能。
