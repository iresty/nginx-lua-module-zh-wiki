
udpsock:receive
---------------
**语法:** *data, err = udpsock:receive(size?)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

使用一个可选的接收缓冲区大小参数 `size` ，从 UDP 或 unix 域数据报 socket 对象中读取数据。

该方法是异步操作并且 100% 非阻塞。

成功情况，返回已经接收的数据；错误情况，返回 `nil` 和错误描述信息。

如果指定了 `size` 参数，该方法将使用它作为缓冲区大小。但是当该值比 `8192` 大时，将继续使用 `8192` 这个大小。

如果没有参数指定，那么最大的缓冲区大小，将假定 `8192` 。

读操作超时控制，是由 [lua_socket_read_timeout](#lua_socket_read_timeout) 配置指令和 [settimeout](#udpsocksettimeout) 方法设置的。而后者有更高的优先级，例如：

```lua

 sock:settimeout(1000)  -- one second timeout
 local data, err = sock:receive()
 if not data then
     ngx.say("failed to read a packet: ", data)
     return
 end
 ngx.say("successfully read a packet: ", data)
```

调用这个方法 *之前* 调用 [settimeout](#udpsocksettimeout) 方法设置超时时间，是非常重要的。

该特性是在 `v0.5.7` 版本首次引入的。

[返回目录](#nginx-api-for-lua)

> English source:

udpsock:receive
---------------
**syntax:** *data, err = udpsock:receive(size?)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Receives data from the UDP or datagram unix domain socket object with an optional receive buffer size argument, `size`.

This method is a synchronous operation and is 100% nonblocking.

In case of success, it returns the data received; in case of error, it returns `nil` with a string describing the error.

If the `size` argument is specified, then this method will use this size as the receive buffer size. But when this size is greater than `8192`, then `8192` will be used instead.

If no argument is specified, then the maximal buffer size, `8192` is assumed.

Timeout for the reading operation is controlled by the [lua_socket_read_timeout](#lua_socket_read_timeout) config directive and the [settimeout](#udpsocksettimeout) method. And the latter takes priority. For example:

```lua

 sock:settimeout(1000)  -- one second timeout
 local data, err = sock:receive()
 if not data then
     ngx.say("failed to read a packet: ", data)
     return
 end
 ngx.say("successfully read a packet: ", data)
```

It is important here to call the [settimeout](#udpsocksettimeout) method *before* calling this method.

This feature was first introduced in the `v0.5.7` release.

[Back to TOC](#nginx-api-for-lua)
