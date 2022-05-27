https://juejin.cn/post/7096070620105932813

第一步返回宿主页我需要以下操作步骤的：

搭建node服务器，处理浏览器加载各种资源的请求
创建宿主html页面，以及入口js文件
创建vue的实例挂在到页面中（此处先不使用.vue文件）
页面展示正确的内容

```
//  server.js
const express = require("express");
const app = express();
const fs = require('fs')
const port = 3000;
// 处理路由
app.get("/", (req, res) => {
    // 设置响应类型
    res.setHeader('content-type','text/html');
    // 返回index.html页面
    res.send(fs.readFileSync('./src/index.html','utf8'));
})

app.listen(port, () => {
    console.log(`Example app listening on port ${port}`);
});
```
我们先本地搭建一个服务，并且返回index.html这个页面
```
<!DOCTYPE html>
<html lang="en">
<body>
    <div id="app"></div>
    <script src="./index.js" type="module"></script>
</body>
</html>
```
这里我们将script标签用module的类型
```
import {createApp} from 'vue';
createApp({

}).mount('#app')
```

```
// 正则匹配js后缀的文件
app.get(/(.*)\.js$/, (req, res) => {
    // 拿到js文件绝对路径
    const p = path.join(__dirname, "src\\" + req.url);
    // 设置响应类型为js content-type 和type都要设置
    res.setHeader("content-type", "text/javascript");
    // 返回js文件
    res.send(fs.readFileSync(p, "utf8"));
});
```
其实熟悉esm模块加载的都应该能明白，此时import只能知道相对地址或者绝对地址路径，对于import {createApp} from 'vue',浏览器是不知道vue这个裸模块的含义。所以这个时候，就需要我们来转换一下，将vue转换成浏览器能识别的模块地址,所以有接下来的操作：

将'vue'模块转换成一个相对地址，例如'@modules/vue'，发送相对地址的请求
服务器识别到带有@modules字段的url后，找到此模块的真实地址（node_modules下面）给返回出去
```
// 正则匹配js后缀的文件
app.get(/(.*)\.js$/, (req, res) => {
    // 拿到js文件绝对路径
    const p = path.join(__dirname, "src\\" + req.url);
    // 设置响应类型为js content-type 和type都要设置
    res.setHeader("content-type", "text/javascript");
    // 返回js文件
    let content = fs.readFileSync(p, "utf8");
    content = rewriteModules(content)
    res.send(content);
});

// 裸模快地址重新 vue=>@modules/vue
function rewriteModules(content){
    let reg = / from ['"](.*)['"]/g
    return content.replace(reg,(s1,s2)=>{
        // 相对路径地址直接返回不处理
        if (s2.startsWith(".") || s2.startsWith("./") || s2.startsWith("../")){
            return s1
        }else{
            // 裸模块
            return `from '/@modules/${s2}'`
        } 
    })
}
```
服务端读取vue文件内容，转换成AST
解析AST脚本获取export default导出的对象
解析AST的模板（会发送import请求）转换成render函数挂在上面的对象上
解析AST样式（会发送import请求）通过js操作方式挂载到dom上。
此对象最终会挂载在到vue的实例上
