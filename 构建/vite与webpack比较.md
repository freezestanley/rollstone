记录一下 vite 构建工具与 webpack 的区别

1.打包原理比较 2.原理图示
3.vite 原理简述
4.vite 的改进点
5.vite 缺点

|  name   |                           打包过程                            |                                        原理                                         |
| :-----: | :-----------------------------------------------------------: | :---------------------------------------------------------------------------------: |
| webpack | 识别入口->逐层识别依赖->分析/转换/编译/输出代码->打包后的代码 | 逐级递归识别依赖，构建依赖图谱->转化 AST 语法树->处理代码->转换为浏览器可识别的代码 |
|  vite   |                               -                               |       基于浏览器原生 ES module，利用浏览器解析 imports，服务器端按需编译返回        |

> 原理图示
> Vite

![avatar](https://github.com/freezestanley/rollstone/blob/main/%E6%9E%84%E5%BB%BA/a.png)

> webpack

![avatar](https://github.com/freezestanley/rollstone/blob/main/%E6%9E%84%E5%BB%BA/b.png)

> vite 原理简述

声明 script 标签类型为 module

```
<script type="module" src="/src/main.js"></script>
```

浏览器向服务器发起 GET

```

// 请求main.js文件：
http://localhost:3000/src/main.js

// /src/main.js:
import { createApp } from 'vue'
import App from './App.vue'
createApp(App).mount('#app')
```

请求到 main.js 文件，检测内部含有 import 引入的包，会对内部的 import 引用发起 HTTP 请求获取模块的内容文件
劫持浏览器的 http 请求，在后端进行相应的处理将项目中使用的文件通过简单的分解与整合，然后再返回给浏览器(整个过程没有对文件进行打包编译)

> vite 的改进点

|                              webpack 缺点                              |                                         vite 改进点                                          |
| :--------------------------------------------------------------------: | :------------------------------------------------------------------------------------------: |
|                             服务器启动缓慢                             | 将应用模块区分为依赖 和 源码 两类;使用 esbuild 构建;在浏览器请求源码时进行转换并按需提供源码 |
|                              基于 nodejs                               |                      esbuild(Go 编写) 预构建依赖，比 node 快 10-100 倍                       |
| 热更新效率低下;编辑单个文件会重新构建整个包;HMR 更新速度随规模增大下降 |            HMR 基于原生 ESM 上，更新速度与应用规模无关;利用 http2 的缓存+压缩优势            |

> vite 缺点
> 生态不及 webpack，加载器、插件不够丰富
> 生产环境 esbuild 构建对于 css 和代码分割不够友好
> 没被大规模重度使用，会隐藏一些问题

库用 rollup,项目用 webpack
之前 tree-shaking 热更新

跟 webpack 有些区别，rollup 不分 plugin 和 loader，只通过 plugin 来实现插件。主要流程如下：
1、创建 rollup 插件;
2、通过 rollup 的 acorn 插件将 code 转为 ast 语法树;
3、通过 stack-utils 插件将当前文件/文件名行数等信息添加到 err 里;
3、通过 estree-walker 遍历语法树，将相关的语句(函数)增加 wrap 节点；
4、通过 escodegen 插件将语法树转为 code;
