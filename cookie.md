SameSite 可以有下面三种值：

Strict ：仅允许一方请求携带 Cookie，即浏览器只发送相同站点请求的 Cookie，即当前网页 URL 与请求目标 URL 完全一致；
Lax ：允许部分第三方请求携带 Cookie；
None ：无论是否跨站都会发送 Cookie

SameSite 属性有三个枚举值，分别是 strict/lax/none。Strict 最为严格，完全禁止第三方 Cookie，跨站点时，任何情况下都不会发送 Cookie。换言之，只有当前网页的 URL 与请求目标一致，才会带上 Cookie。Lax 规则稍稍放宽，大多数情况也是不发送第三方 Cookie，但是导航到目标网址的 Get 请求除外。

设置了 Strict 或 Lax 以后，基本就杜绝了 CSRF 攻击。当然，前提是用户浏览器支持 SameSite 属性。Chrome 计划将 Lax 变为默认设置。这时，网站可以选择显式关闭 SameSite 属性，将其设为 None。不过，前提是必须同时设置 Secure 属性（Cookie 只能通过 HTTPS 协议发送），否则无效。

Chorme80 之后默认是 Lax

https://i-want-offer.github.io/FE-Essay/%E5%89%8D%E5%90%8E%E7%AB%AF%E9%80%9A%E4%BF%A1/Cookie%E7%9A%84SameSite%E5%B1%9E%E6%80%A7.html#cookie


# 根DNS服务器、顶级域DNS服务器、权威DNS服务器

从请求主机到本地DNS服务器的查询是 递归 的，其余查询是 迭代 的。

DNS 提供了两种查询过程：

# 递归查询
递归查询：在该模式下DNS服务器接收客户请求，必须使用一个准确的查询结果回复客户机，如果DNS服务器没有存储DNS值，那么该服务器会询问其他服务器，并将返回一个查询结果给客户机。

# 迭代查询
迭代查询：DNS服务器会向客户机提供其他能够解释查询请求的DNS服务器，当客户机发送查询时，DNS并不直接回复查询结果，而是告诉客户机，另一台DNS服务器的地址，客户再向这台DNS服务器提交请求，依次循环直接返回结果
