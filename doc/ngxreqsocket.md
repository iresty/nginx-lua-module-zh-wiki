ngx.req.socket
--------------
**语法:** *tcpsock, err = ngx.req.socket()*

**语法:** *tcpsock, err = ngx.req.socket(raw)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

返回一个包含下游连接的只读 cosocket 对象。只有 [receive](#tcpsockreceive) 和 [receiveuntil](#tcpsockreceiveuntil) 方法在该对象上是支持的。
Returns a read-only cosocket object that wraps the downstream connection. Only [receive](#tcpsockreceive) and [receiveuntil](#tcpsockreceiveuntil) methods are supported on this object.

错误情况，将返回 `nil` 和错误字符描述信息。
In case of error, `nil` will be returned as well as a string describing the error.

通过该方法返回的 socket 对象，通常是用流式格式读取当前请求体。不要开启 [lua_need_request_body](#lua_need_request_body) 指令，并且不要混合调用 [ngx.req.read_body](#ngxreqread_body) 和 [ngx.req.discard_body](#ngxreqdiscard_body)。
The socket object returned by this method is usually used to read the current request's body in a streaming fashion. Do not turn on the [lua_need_request_body](#lua_need_request_body) directive, and do not mix this call with [ngx.req.read_body](#ngxreqread_body) and [ngx.req.discard_body](#ngxreqdiscard_body).

如果任何的请求体数据已经被预读到 Nginx 内核请求缓冲区，得到的 cosocket 对象需要小心对待，应避免由这种预读导致的潜在数据丢失。
If any request body data has been pre-read into the Nginx core request header buffer, the resulting cosocket object will take care of this to avoid potential data loss resulting from such pre-reading.
Chunked request bodies are not yet supported in this API.

从 `v0.9.0` 版本开始，该函数接受一个可选的布尔值参数 `raw` 。当该参数为 `true` 时，该方法将返回一个包含原生下游连接的全双工 cosocket 对象，你能对它调用 [receive](#tcpsockreceive)， [receiveuntil](#tcpsockreceiveuntil) 和 [send](#tcpsocksend) 。
Since the `v0.9.0` release, this function accepts an optional boolean `raw` argument. When this argument is `true`, this function returns a full-duplex cosocket object wrapping around the raw downstream connection socket, upon which you can call the [receive](#tcpsockreceive), [receiveuntil](#tcpsockreceiveuntil), and [send](#tcpsocksend) methods.

当指定 `raw` 参数为 `true` ，这里需要没有任何来自 [ngx.say](#ngxsay)、[ngx.print](#ngxprint) 或 [ngx.send_headers](#ngxsend_headers) 方法调用的待处理数据。所以如果你有下游输出调用，你应当在调用 `ngx.req.socket(true)` 之前调用 [ngx.flush(true)](#ngxflush) 确保这里没有任何待处理数据。如果请求体还没有读取，那么这个“原生 socket”也能用来读取请求体。
When the `raw` argument is `true`, it is required that no pending data from any previous [ngx.say](#ngxsay), [ngx.print](#ngxprint), or [ngx.send_headers](#ngxsend_headers) calls exists. So if you have these downstream output calls previously, you should call [ngx.flush(true)](#ngxflush) before calling `ngx.req.socket(true)` to ensure that there is no pending output data. If the request body has not been read yet, then this "raw socket" can also be used to read the request body.

你可以使用通过 `ngx.req.socket(true)` 返回的“原生请求 socket”来实现各种样式协议如 [WebSocket](http://en.wikipedia.org/wiki/WebSocket) ，或仅发出自己的 HTTP 请求头或体数据。真实世界，你可以参考 [lua-resty-websocket](https://github.com/openresty/lua-resty-websocket) 库。
You can use the "raw request socket" returned by `ngx.req.socket(true)` to implement fancy protocols like [WebSocket](http://en.wikipedia.org/wiki/WebSocket), or just emit your own raw HTTP response header or body data. You can refer to the [lua-resty-websocket library](https://github.com/openresty/lua-resty-websocket) for a real world example.

该函数是在 `v0.5.0rc1` 版本首次引入的。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.req.socket
--------------
**syntax:** *tcpsock, err = ngx.req.socket()*

**syntax:** *tcpsock, err = ngx.req.socket(raw)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

Returns a read-only cosocket object that wraps the downstream connection. Only [receive](#tcpsockreceive) and [receiveuntil](#tcpsockreceiveuntil) methods are supported on this object.

In case of error, `nil` will be returned as well as a string describing the error.

The socket object returned by this method is usually used to read the current request's body in a streaming fashion. Do not turn on the [lua_need_request_body](#lua_need_request_body) directive, and do not mix this call with [ngx.req.read_body](#ngxreqread_body) and [ngx.req.discard_body](#ngxreqdiscard_body).

If any request body data has been pre-read into the Nginx core request header buffer, the resulting cosocket object will take care of this to avoid potential data loss resulting from such pre-reading.
Chunked request bodies are not yet supported in this API.

Since the `v0.9.0` release, this function accepts an optional boolean `raw` argument. When this argument is `true`, this function returns a full-duplex cosocket object wrapping around the raw downstream connection socket, upon which you can call the [receive](#tcpsockreceive), [receiveuntil](#tcpsockreceiveuntil), and [send](#tcpsocksend) methods.

When the `raw` argument is `true`, it is required that no pending data from any previous [ngx.say](#ngxsay), [ngx.print](#ngxprint), or [ngx.send_headers](#ngxsend_headers) calls exists. So if you have these downstream output calls previously, you should call [ngx.flush(true)](#ngxflush) before calling `ngx.req.socket(true)` to ensure that there is no pending output data. If the request body has not been read yet, then this "raw socket" can also be used to read the request body.

You can use the "raw request socket" returned by `ngx.req.socket(true)` to implement fancy protocols like [WebSocket](http://en.wikipedia.org/wiki/WebSocket), or just emit your own raw HTTP response header or body data. You can refer to the [lua-resty-websocket library](https://github.com/openresty/lua-resty-websocket) for a real world example.

This function was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)
