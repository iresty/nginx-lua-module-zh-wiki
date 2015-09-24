ngx.req.discard_body
--------------------
**语法:** *ngx.req.discard_body()*

**环境:** *rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

明确丢弃请求体，也就是说，读取连接中的数据后立即丢弃。请注意，忽略请求体并不是丢弃请求体的正确方式，为避免破坏 HTTP 1.1 长连接或 HTTP 1.1 流水线 (pipelining)，必须使用本函数。

这个函数是异步调用，将立即返回。

如果请求体已经被读取，此函数将不会执行，立即返回。

这个函数在 `v0.3.1rc17`` 版本中首次引入。

更多用法请参考 [ngx.req.read_body](#ngxreqread_body).


> English Source

**syntax:** *ngx.req.discard_body()*

**context:** *rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

Explicitly discard the request body, i.e., read the data on the connection and throw it away immediately. Please note that ignoring request body is not the right way to discard it, and that this function must be called to avoid breaking things under HTTP 1.1 keepalive or HTTP 1.1 pipelining.

This function is an asynchronous call and returns immediately.

If the request body has already been read, this function does nothing and returns immediately.

This function was first introduced in the `v0.3.1rc17` release.

See also [ngx.req.read_body](#ngxreqread_body).

[返回目录](#nginx-api-for-lua)
