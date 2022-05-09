最核心的内容就是同构
node server 接收客户端请求，得到当前的req url path,然后在已有的路由表内查找到对应的组件，拿到需要请求的数据，将数据作为 props
、context或者store 形式传入组件，然后基于 react 内置的服务端渲染api renderToString() or renderToNodeStream() 把组件渲染为 html字符串或者 stream 流, 在把最终的 html 进行输出前需要将数据注入到浏览器端(注水)，server 输出(response)后浏览器端可以得到数据(脱水)，浏览器开始进行渲染和节点对比，然后执行组件的componentDidMount 完成组件内事件绑定和一些交互，浏览器重用了服务端输出的 html 节点，整个流程结束
![avatar](https://github.com/freezestanley/rollstone/blob/main/react/aa.png)