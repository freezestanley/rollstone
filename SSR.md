# SSR 

node server 接收客户端请求，得到当前的req url path,然后在已有的路由表内查找到对应的组件，拿到需要请求的数据，将数据作为 props
、context或者store 形式传入组件，然后基于 react 内置的服务端渲染api renderToString() or renderToNodeStream() 把组件渲染为 html字符串或者 stream 流, 在把最终的 html 进行输出前需要将数据注入到浏览器端(注水)，server 输出(response)后浏览器端可以得到数据(脱水)，浏览器开始进行渲染和节点对比，然后执行组件的componentDidMount 完成组件内事件绑定和一些交互，浏览器重用了服务端输出的 html 节点，整个流程结束

![avatar](https://github.com/freezestanley/rollstone/blob/main/ssr.png)

> ejs 模版

```
// index.html
<!DOCTYPE html>
<html lang="en">
<head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <meta http-equiv="X-UA-Compatible" content="ie=edge">
   <title>react ssr <%= title %></title>
</head>
<body>
   <%=  data %>
</body>
</html>
```
```
 //node ssr
 const ejs = require('ejs');
 const http = require('http');

http.createServer((req, res) => {
    if (req.url === '/') {
        res.writeHead(200, {
            'Content-Type': 'text/html' 
        });
        // 渲染文件 index.ejs
        ejs.renderFile('./views/index.ejs', {
            title: 'react ssr', 
            data: '首页'}, 
            (err, data) => {
            if (err ) {
                console.log(err);
            } else {
                res.end(data);
            }
        })
    }
}).listen(8080);
```

# jsx
```
const  React  = require('react');

const { renderToString}  = require( 'react-dom/server');

const http = require('http');

//组件
class Index extends React.Component{
    constructor(props){
        super(props);
    }

    render(){
        return <h1>{this.props.data.title}</h1>
    }
}
 
//模拟数据的获取
const fetch = function () {
    return {
        title:'react ssr',
        data:[]
    }
}

//服务
http.createServer((req, res) => {
    if (req.url === '/') {
        res.writeHead(200, {
            'Content-Type': 'text/html'
        });

        const data = fetch();

        const html = renderToString(<Index data={data}/>);
        res.end(html);
    }
}).listen(8080);
```

# 路由同构
```
// routes-config.js
class Detail extends React.Component{

    render(){
        return <div>detail</div>
    }
}

class Index extends React.Component {

    render() {
        return <div>index</div>
    }
}


const routes = [
  
            {
                path: "/",
                exact: true,
                component: Home
            },
            {
                path: '/detail', exact: true,
                component:Detail,
            },
            {
                path: '/detail/:a/:b', exact: true,
                component: Detail
            }
         
];
```
//客户端 路由组件
```
import routes from './routes-config.js';

function App(){
    return (
        <Layout>
            <Switch>

                        {
                            routes.map((item,index)=>{
                                return <Route path={item.path} key={index} exact={item.exact} render={item.component}></Route>
                            })
                        }
            </Switch>
        </Layout>
    );
}

export default App;
```
# node server 进行组件查找

```
//引入官方库
// matchRoutes(routes, pathname)
import { matchRoutes } from "react-router-config";
import routes from './routes-config.js';

const path = req.path;

const branch = matchRoutes(routes, path);

//得到要渲染的组件
const Component = branch[0].route.component;
 

//node server 
http.createServer((req, res) => {
    
        const url = req.url;
        //简单容错，排除图片等资源文件的请求
        if(url.indexOf('.')>-1) { res.end(''); return false;}

        res.writeHead(200, {
            'Content-Type': 'text/html'
        });
        const data = fetch();

        //查找组件
        const branch =  matchRoutes(routes,url);
        
        //得到组件
        const Component = branch[0].route.component;

        //将组件渲染为 html 字符串
        const html = renderToString(<Component data={data}/>);

        res.end(html);
        
 }).listen(8080);
```
matchRoutes方法的返回值,其中route.component 就是 要渲染的组件
```
[
    { 
    
    route:
        { path: '/detail', exact: true, component: [Function: Detail] },
    match:
        { path: '/detail', url: '/detail', isExact: true, params: {} } 
        
    }
   ]

```

# 解决我们最开始发现的第二个问题 - 【获取数据的方法和逻辑写在哪里？】
```
static async  getInitialProps(opt) {
        const fetch1 =await fetch('/xxx.com/a');
        const fetch2 = await fetch('/xxx.com/b');

        return {
            res:[fetch1,fetch2]
        }
    }
```
static 数据方法

```
//组件
class Index extends React.Component{
    constructor(props){
        super(props);
    }

    //数据预取方法  静态 异步 方法
    static async  getInitialProps(opt) {
        const fetch1 =await fetch('/xxx.com/a');
        const fetch2 = await fetch('/xxx.com/b');

        return {
            res:[fetch1,fetch2]
        }
    }

    render(){
        return <h1>{this.props.data.title}</h1>
    }
}


//node server 
http.createServer((req, res) => {
    
        const url = req.url;
        if(url.indexOf('.')>-1) { res.end(''); return false;}

        res.writeHead(200, {
            'Content-Type': 'text/html'
        });
        
        //组件查找
        const branch =  matchRoutes(routes,url);
        
        //得到组件
        const Component = branch[0].route.component;
    
        //数据预取
        const data = Component.getInitialProps(branch[0].match.params);
      
        //传入数据，渲染组件为 html 字符串
        const html = renderToString(<Component data={data}/>);

        res.end(html);

 }).listen(8080);
```
