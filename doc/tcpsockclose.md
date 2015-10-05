
tcpsock:close
-------------
**语法:** *ok, err = tcpsock:close()*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

关闭当前 TCP 或 unix domain socket 。成功情况下返回 `1` ，否则将返回 `nil` 和 错误描述信息。

注意，调用了 [setkeepalive](#tcpsocksetkeepalive) 方法的 socket 对象，不再需要调用这个方法，因为这个 socket 对象已经关闭（当前连接已经保存到内建的连接池内）。

当 socket 对象已经被 Lua GC（垃圾回收）或当前客户 HTTP 请求完成处理时，没有调用这个方法的 socket 对象（和其他关联连接）将会被关闭。

该特性是在 `v0.5.0rc1` 版本首次引入的。

[返回目录](#nginx-api-for-lua)


tcpsock:close
-------------
**syntax:** *ok, err = tcpsock:close()*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Closes the current TCP or stream unix domain socket. It returns the `1` in case of success and returns `nil` with a string describing the error otherwise.

Note that there is no need to call this method on socket objects that have invoked the [setkeepalive](#tcpsocksetkeepalive) method because the socket object is already closed (and the current connection is saved into the built-in connection pool).

Socket objects that have not invoked this method (and associated connections) will be closed when the socket object is released by the Lua GC (Garbage Collector) or the current client HTTP request finishes processing.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

