tcpsock:send
------------
**语法:** *bytes, err = tcpsock:send(data)*

**内容:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

在当前 TCP 或 Unix Domain Socket 连接上非阻塞的发送数据。

该方法是个异步操作，直到 *所有* 的数据全部被刷写到系统 socket 发送缓冲区或有错误发生，否则不会返回。

成功情况下，返回已经发送数据字节数的总数。其他情况，返回 `nil` 和错误描述信息。

输入参数 `data` 可以是 Lua 字符串，也可以是包含字符串的（嵌套）Lua 表。对于输入参数是表的情况，该方法将逐一拷贝所有的字符串对象到底层的 Nginx socket 发送缓冲区，这是比 Lua 层面完成字符串拼接更好的优化方案。

发送操作超时控制，是由 [lua_socket_send_timeout](#lua_socket_send_timeout) 配置指令和 [settimeout](#tcpsocksettimeout) 方法设置的。而后者有更高的优先级，例如：

```lua

 sock:settimeout(1000)  -- one second timeout
 local bytes, err = sock:send(request)
```

调用这个方法 *之前* 调用 [settimeout](#tcpsocksettimeout) 方法设置超时时间，是非常重要的。

一旦有任何错误发生，该方法将自动关闭当前连接。

该特性在 `v0.5.0rc1` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

tcpsock:send
------------
**syntax:** *bytes, err = tcpsock:send(data)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Sends data without blocking on the current TCP or Unix Domain Socket connection.

This method is a synchronous operation that will not return until *all* the data has been flushed into the system socket send buffer or an error occurs.

In case of success, it returns the total number of bytes that have been sent. Otherwise, it returns `nil` and a string describing the error.

The input argument `data` can either be a Lua string or a (nested) Lua table holding string fragments. In case of table arguments, this method will copy all the string elements piece by piece to the underlying Nginx socket send buffers, which is usually optimal than doing string concatenation operations on the Lua land.

Timeout for the sending operation is controlled by the [lua_socket_send_timeout](#lua_socket_send_timeout) config directive and the [settimeout](#tcpsocksettimeout) method. And the latter takes priority. For example:

```lua

 sock:settimeout(1000)  -- one second timeout
 local bytes, err = sock:send(request)
```

It is important here to call the [settimeout](#tcpsocksettimeout) method *before* calling this method.

In case of any connection errors, this method always automatically closes the current connection.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)


