从输入 URl 开始
与服务器交互首先要进行 DNS 查询，得到服务器的 IP 地址，浏览器会首先查询自己的缓存，之后会查询本地 HOSTS，如果仍然没找到会发起向 DNS 服务器查询的请求
在文档顶部我们可以将我们即将要请求的地址的 DNS 预先查询，通过插入一个 link 标签

<link rel="dns-prefetch" href="https://fonts.googleapis.com/">
建立HTTP(TCP)连接
由于TCP的可靠性，每条独立的TCP连接都会进行一次三次握手，从上面的Network的分析中可以得到握手往往会消耗大部分时间，真正的数据传输反而会少一些(当然取决于内容多少)。HTTP1.0和HTTP1.1为了解决这个问题在header中加入了Connection: Keep-Alive，keep-alive的连接会保持一段时间不断开，后续的请求都会复用这一条TCP，不过由于管道化的原因也会发生队头阻塞的问题。
HTTP1.1默认开启Keep-Alive，HTTP1.0可能现在不多见了，如果你还在用，可以升级一下版本，或者带上这个header。

HTTP2
HTTP2 相对于 HTTP1.1 的一个主要升级是多路复用，多路复用通过更小的二进制帧构成多条数据流，交错的请求和响应可以并行传输而不被阻塞，这样就解决了 HTTP1.1 时复用会产生的队头阻塞的问题，同时 HTTP2 有首部压缩的功能，如果两个请求首部(headers)相同，那么会省去这一部分，只传输不同的首部字段，进一步减少请求的体积。
Nginx 开启 HTTP2 的方式特别容易，只需要加一句 http2 既可开启：
server {
 listen 443 ssl http2; #  加一句  http2.
 server_name domain.com;
}
