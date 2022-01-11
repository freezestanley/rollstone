http:目前绝大多数是 http1.1 版本，最原始的 web 协议，默认 80 端口，基于 TCP 协议。
https：加密的 http 协议，默认 443 端口，基于 TCP 协议。
http2：第二代 http 协议，相较于 HTTP1.x，大幅度的提升了 web 性能。在与 HTTP/1.1 完全语义兼容的基础上，进一步减少了网络延迟和传输的安全性，基于 TCP。
websocket：服务端推送，实现服务端客户端全双工通信，基于 TCP。
以上 http,websocket 都属于应用层协议，tcp 属于传输层协议。
TPC/IP 协议是传输层协议，主要解决数据如何在网络中传输，而 HTTP 是应用层协议，主要解决如何包装数据。
详情：TCP/IP、Http、Socket 的区别

实际上，传输层的 TCP 是基于网络层的 IP 协议的，而应用层的 HTTP 协议又是基于传输层的 TCP 协议的，而 Socket 本身不算是协议，就像上面所说，它只是提供了一个针对 TCP 或者 UDP 编程的接口。

HTTP2.0 可以说是 SPDY 的升级版（基于 SPDY 设计的），但是依然存在一些不同点：

HTTP2.0 支持明文传输，而 SPDY 强制使用 HTTPS；
HTTP2.0 消息头的压缩算法采用 HPACK，而非 SPDY 采用的 DEFLATE。

异同：
HTTP 协议为单向协议，即浏览器只能向服务器请求资源，服务器才能将数据传送给浏览器，而服务器不能主动向浏览器传递数据。分为长连接和短连接，短连接是每次 http 请求时都需要三次握手才能发送自己的请求，每个 request 对应一个 response；长连接是短时间内保持连接，保持 TCP 不断开，指的是 TCP 连接。

> “我们在传输数据时，可以只使用（传输层）TCP/IP 协议，但是那样的话，如果没有应用层，便无法识别数据内容，如果想要使传输的数据有意义，则必须使用到应用层协议，应用层协议有很多，比如 HTTP、FTP、TELNET 等，也可以自己定义应用层协议。WEB 使用 HTTP 协议作应用层协议，以封装 HTTP 文本信息，然后使用 TCP/IP 做传输层协议将它发到网络上。”

js

```
var evtSource = new EventSource("http://localhost:3000");
let eventList = document.getElementsByTagName('body')[0]

evtSource.addEventListener("ping", function(e) {
    console.log(2222,e);
    var newElement = document.createElement("li");
    let eventList = document.getElementsByTagName('body')[0]
    var obj = JSON.parse(e.data);
    newElement.innerHTML = "ping at " + obj.date;
    eventList.appendChild(newElement);

}, false);

evtSource.addEventListener("error",function(e){
    console.log("服务器发送给客户端的数据为:" + e.data);
});

//只要和服务器连接，就会触发open事件
evtSource.addEventListener("open",function(){
    console.log("和服务器建立连接");
 });

 //处理服务器响应报文中的load事件
 evtSource.addEventListener("load",function(e){
     console.log("服务器发送给客户端的数据为:" + e.data);
 });
```

node

```
http.createServer((req, res) => {
    res.writeHead(200, {
        'Content-Type' : 'text/event-stream',
        'Access-Control-Allow-Origin':'*'
    })
    let i = 0;
    const timer = setInterval(()=>{
        const date = {date:new Date()}
        var content = 'event: ping\n'+"data:"+JSON.stringify(date)+"" + "\n\n";
        res.write(content);
    },1000)

    res.connection.on("close", function(){
        res.end();
        clearInterval(timer);
        console.log("Client closed connection. Aborting.");
        });

}).listen(3000)
console.log('server is run http://localhost:3000');
```
