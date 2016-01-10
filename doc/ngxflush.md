ngx.flush
---------
**语法:** *ok, err = ngx.flush(wait?)*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

向客户端刷新响应输出。

自 `v0.3.1rc34` 版本开始，`ngx.flush` 接受一个布尔型可选参数 `wait` (默认值 `false`)。当通过默认参数调用时，本函数发起一个异步调用 (将直接返回，不等待输出数据被写入系统发送缓冲区)。当把 `wait` 参数设置为 `true` 时，本函数将以同步模式执行。

在同步模式下，本函数不会立即返回，一直到所有输出数据被写入系统输出缓冲区，或者到达发送超时  [send_timeout](http://nginx.org/en/docs/http/ngx_http_core_module.html#send_timeout) 时间。请注意，因为使用了 Lua 协程机制，本函数即使在同步模式下也不会阻塞 Nginx 事件循环。

当 `ngx.flush(true)` 在 [ngx.print](#ngxprint) 或 [ngx.say](#ngxsay) 之后被立刻调用时，它将使这两个函数以同步模式执行。这在流式输出时非常有用。

请注意，`ngx.flush` 在 HTTP 1.0 缓冲输出模式下不起作用。详情请参考 [HTTP 1.0 support](#http-10-support)。

自 `v0.8.3` 版本开始，本函数执行成功是返回 `1`，否则返回 `nil` 和错误信息串。

> English Source

**syntax:** *ok, err = ngx.flush(wait?)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

Flushes response output to the client. 

`ngx.flush` accepts an optional boolean `wait` argument (Default: `false`) first introduced in the `v0.3.1rc34` release. When called with the default argument, it issues an asynchronous call (Returns immediately without waiting for output data to be written into the system send buffer). Calling the function with the `wait` argument set to `true` switches to synchronous mode. 

In synchronous mode, the function will not return until all output data has been written into the system send buffer or until the [send_timeout](http://nginx.org/en/docs/http/ngx_http_core_module.html#send_timeout) setting has expired. Note that using the Lua coroutine mechanism means that this function does not block the Nginx event loop even in the synchronous mode.

When `ngx.flush(true)` is called immediately after [ngx.print](#ngxprint) or [ngx.say](#ngxsay), it causes the latter functions to run in synchronous mode. This can be particularly useful for streaming output.

Note that `ngx.flush` is not functional when in the HTTP 1.0 output buffering mode. See [HTTP 1.0 support](#http-10-support).

Since `v0.8.3` this function returns `1` on success, or returns `nil` and a string describing the error otherwise.


[返回目录](#nginx-api-for-lua)