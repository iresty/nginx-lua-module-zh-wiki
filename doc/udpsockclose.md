
udpsock:close
-------------
**syntax:** *ok, err = udpsock:close()*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

关闭当前 UDP 或 unix 域数据报 socket 。成功情况返回 `1` ，反之错误情况返回 `nil` 和 错误描述信息。
Closes the current UDP or datagram unix domain socket. It returns the `1` in case of success and returns `nil` with a string describing the error otherwise.

当 socket 对象被 Lua GC（垃圾回收）释放或当前客户端请求处理完毕，还没有调用该方法的 socket 对象（关联连接）都将被关闭。

该特性是在 `v0.5.7` 版本首次引入的。

[返回目录](#nginx-api-for-lua)

> English source:

udpsock:close
-------------
**syntax:** *ok, err = udpsock:close()*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Closes the current UDP or datagram unix domain socket. It returns the `1` in case of success and returns `nil` with a string describing the error otherwise.

Socket objects that have not invoked this method (and associated connections) will be closed when the socket object is released by the Lua GC (Garbage Collector) or the current client HTTP request finishes processing.

This feature was first introduced in the `v0.5.7` release.

[Back to TOC](#nginx-api-for-lua)

