ngx.socket.connect
------------------
**语法:** *tcpsock, err = ngx.socket.connect(host, port)*

**语法:** *tcpsock, err = ngx.socket.connect("unix:/path/to/unix-domain.socket")*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

该函数是融合 [ngx.socket.tcp()](#ngxsockettcp) 和 [connect()](#tcpsockconnect) 方法到一个单独操作的快捷方式。 它实际上可以这样实现：

```lua

 local sock = ngx.socket.tcp()
 local ok, err = sock:connect(...)
 if not ok then
     return nil, err
 end
 return sock
```

这里没办法使用 [settimeout](#tcpsocksettimeout) 方法来指定连接时间，只能通过指令 [lua_socket_connect_timeout](#lua_socket_connect_timeout) 预先配置作为替代方案。

该特性是在 `v0.5.0rc1` 版本首次引入的。
This feature was first introduced in the `v0.5.0rc1` release.

[返回目录](#nginx-api-for-lua)

> English source:

ngx.socket.connect
------------------
**syntax:** *tcpsock, err = ngx.socket.connect(host, port)*

**syntax:** *tcpsock, err = ngx.socket.connect("unix:/path/to/unix-domain.socket")*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

This function is a shortcut for combining [ngx.socket.tcp()](#ngxsockettcp) and the [connect()](#tcpsockconnect) method call in a single operation. It is actually implemented like this:

```lua

 local sock = ngx.socket.tcp()
 local ok, err = sock:connect(...)
 if not ok then
     return nil, err
 end
 return sock
```

There is no way to use the [settimeout](#tcpsocksettimeout) method to specify connecting timeout for this method and the [lua_socket_connect_timeout](#lua_socket_connect_timeout) directive must be set at configure time instead.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)