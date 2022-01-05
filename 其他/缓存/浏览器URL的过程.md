从输入 URl 开始
与服务器交互首先要进行 DNS 查询，得到服务器的 IP 地址，浏览器会首先查询自己的缓存，之后会查询本地 HOSTS，如果仍然没找到会发起向 DNS 服务器查询的请求
在文档顶部我们可以将我们即将要请求的地址的 DNS 预先查询，通过插入一个 link 标签

<link rel="dns-prefetch" href="https://fonts.googleapis.com/">

建立 HTTP(TCP)连接
由于 TCP 的可靠性，每条独立的 TCP 连接都会进行一次三次握手，从上面的 Network 的分析中可以得到握手往往会消耗大部分时间，真正的数据传输反而会少一些(当然取决于内容多少)。HTTP1.0 和 HTTP1.1 为了解决这个问题在 header 中加入了 Connection: Keep-Alive，keep-alive 的连接会保持一段时间不断开，后续的请求都会复用这一条 TCP，不过由于管道化的原因也会发生队头阻塞的问题。
HTTP1.1 默认开启 Keep-Alive，HTTP1.0 可能现在不多见了，如果你还在用，可以升级一下版本，或者带上这个 header。

HTTP2
HTTP2 相对于 HTTP1.1 的一个主要升级是多路复用，多路复用通过更小的二进制帧构成多条数据流，交错的请求和响应可以并行传输而不被阻塞，这样就解决了 HTTP1.1 时复用会产生的队头阻塞的问题，同时 HTTP2 有首部压缩的功能，如果两个请求首部(headers)相同，那么会省去这一部分，只传输不同的首部字段，进一步减少请求的体积。
Nginx 开启 HTTP2 的方式特别容易，只需要加一句 http2 既可开启：
server {
 listen 443 ssl http2; #  加一句  http2.
 server_name domain.com;
}
缓存
HTTP 缓存主要分为两种，一种是强缓存，另一种是协商缓存，都通过 Headers 控制。
![avatar](https://github.com/freezestanley/rollstone/blob/main/%E5%85%B6%E4%BB%96/缓存/a.jpg)

> 强缓存

> 强缓存根据请求头的 Expires 和 Cache-Control 判断是否命中强缓存，命中强缓存的资源直接从本地加载，不会发起任何网络请求。

```
 Cache-Control 的值有很多:
 Cache-Control: max-age=<seconds>
 Cache-Control: max-stale[=<seconds>]
 Cache-Control: min-fresh=<seconds>
 Cache-control: no-cache
 Cache-control: no-store
 Cache-control: no-transform
 Cache-control: only-if-cached
```

常用的有 max-age，no-cache 和 no-store。
max-age 是资源从响应开始计时的最大新鲜时间，一般响应中还会出现 age 标明这个资源当前的新鲜程度。
no-cache 会让浏览器缓存这个文件到本地但是不用，Network 中 disable-cache 勾中的话就会在请求时带上这个 haader，会在下一次新鲜度验证通过后使用这个缓存。
no-store 会完全放弃缓存这个文件。
服务器响应时的 Cache-Control 略有不同，其中有两个需要注意下:

1. public, public 表明这个请求可以被任何对象缓存，代理/CDN 等中间商。
2. private，private 表明这个请求只能被终端缓存，不允许代理或者 CDN 等中间商缓存。
   Expires 是一个具体的日期，到了那个日期就会让这个缓存失活，优先级较低，存在 max-age 的情况下会被忽略，和本地时间绑定，修改本地时间可以绕过。

   另外，如果你的服务器的返回内容中不存在 Expires，Cache-Control: max-age，或 Cache-Control:s-maxage 但是存在 Last-Modified 时，那么浏览器默认会采用一个启发式的算法，即启发式缓存。
   通常会取响应头的 Date_value - Last-Modified_value 值的 10%作为缓存时间，之后浏览器仍然会按强缓存来对待这个资源一段时间，如果你不想要缓存的话务必确保有 no-cache 或 no-store 在响应头中。

协商缓存 etag

协商缓存一般会在强缓存新鲜度过期后发起，向服务器确认是否需要更新本地的缓存文件，如果不需要更新，服务器会返回 304 否则会重新返回整个文件。
服务器响应中会携带 ETag 和 Last-Modified，Last-Modified 表示本地文件最后修改日期，浏览器会在 request header 加上 If-Modified-Since（上次返回的 Last-Modified 的值），询问服务器在该日期后资源是否有更新，有更新的话就会将新的资源发送回来。
但是如果在本地打开缓存文件，就会造成 Last-Modified 被修改，所以在 HTTP / 1.1 出现了 ETag。
Etag 就像一个指纹，资源变化都会导致 ETag 变化，跟最后修改时间没有关系，ETag 可以保证每一个资源是唯一的
If-None-Match 的 header 会将上次返回的 ETag 发送给服务器，询问该资源的 ETag 是否有更新，有变动就会发送新的资源回来
ETag(If-None-Match)的优先级高于 Last-Modified(If-Modified-Since)，优先使用 ETag 进行确认。
协商缓存比强缓存稍慢，因为还是会发送请求到服务器进行确认。
