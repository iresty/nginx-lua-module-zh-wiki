HTTP 1.0 支持
=============

HTTP 1.0 协议不支持分块输出，当响应体不为空时，需要在响应头中明确指定 `Content-Length`，以支持 HTTP 1.0 长连接。所以当把 [lua_http10_buffering](#lua_http10_buffering) 设置为 `on` 输出 HTTP 1.0 响应时，ngx_lua 将缓存 [ngx.say](#ngxsay) 和 [ngx.print](#ngxprint) 的所有输出，同时延迟发送响应头直到接收到所有输出内容。这时 ngx_lua 可以计算响应体的总长度，并为 HTTP 1.0 客户端创建一个正确的 `Content-Length` 响应头。如果在正在执行的 Lua 代码中设置 `Content-Length` 响应头，这种缓冲模式将被禁用，即使已经将 [lua_http10_buffering](#lua_http10_buffering) 指令设置为 `on`。

对于大型流式响应输出，禁用 [lua_http10_buffering](#lua_http10_buffering) 以最小化内存占用非常重要。

请注意，一些常见的 HTTP 性能测试工具，例如 `ab` 和 `http_load` 默认发送 HTTP 1.0 请求。要强制 `curl` 发送 HTTP 1.0 请求，使用 `-0` 选项。

> Engslish Source

The HTTP 1.0 protocol does not support chunked output and requires an explicit `Content-Length` header when the response body is not empty in order to support the HTTP 1.0 keep-alive.
So when a HTTP 1.0 request is made and the [lua_http10_buffering](#lua_http10_buffering) directive is turned `on`, ngx_lua will buffer the
output of [ngx.say](#ngxsay) and [ngx.print](#ngxprint) calls and also postpone sending response headers until all the response body output is received.
At that time ngx_lua can calculate the total length of the body and construct a proper `Content-Length` header to return to the HTTP 1.0 client.
If the `Content-Length` response header is set in the running Lua code, however, this buffering will be disabled even if the [lua_http10_buffering](#lua_http10_buffering) directive is turned `on`.

For large streaming output responses, it is important to disable the [lua_http10_buffering](#lua_http10_buffering) directive to minimise memory usage.

Note that common HTTP benchmark tools such as `ab` and `http_load` issue HTTP 1.0 requests by default.
To force `curl` to send HTTP 1.0 requests, use the `-0` option.

[返回目录](#table-of-contents)