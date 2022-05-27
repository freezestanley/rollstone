Cookie 还有两个常用属性 Secure 和 HttpOnly
其中 Secure 是只允许 Cookie 在 HTTPS 请求中被使用

而 HttpOnly 则用来禁止使用 JS 访问 cookie

SameSite
在前段时间，Chrome 更新 80 版本时，将 Cookie 的跨站策略（SameSite）默认设置为了 Lax，即仅允许同站或者子站访问 Cookie，而老版本是 None，即允许所有跨站 Cookie

这会导致用户访问 xyz.com 时，浏览器默认将不会发送 Cookie 给 taobao.com，导致第三方 Cookie 失效的问题

要解决的话，在返回请求的 header 里将 SameSite 设置为 None 即可