
ngx.socket.tcp
--------------
**语法:** *tcpsock = ngx.socket.tcp()*

**内容:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

创建并得到一个 TCP 或 unix 域流式 socket 对象（也被称为 "cosocket" 对象的一种类型）。该对象支持下面这些方法：

* [connect](#tcpsockconnect)
* [sslhandshake](#tcpsocksslhandshake)
* [send](#tcpsocksend)
* [receive](#tcpsockreceive)
* [close](#tcpsockclose)
* [settimeout](#tcpsocksettimeout)
* [setoption](#tcpsocksetoption)
* [receiveuntil](#tcpsockreceiveuntil)
* [setkeepalive](#tcpsocksetkeepalive)
* [getreusedtimes](#tcpsockgetreusedtimes)

兼容 [LuaSocket](http://w3.impa.br/~diego/software/luasocket/tcp.html) 库的 TCP API 是需要的，额外的它却是 100% 非阻塞。此外，我们引入了一些新的API，提供更多的功能。

通过该 API 函数创建的 cosocket 对象，与创造它的 Lua 环境拥有相同的生命周期。所以永远不要把 cosocket 对象传递给其他 Lua 环境（包括 ngx.timer 回调函数），并且永远不要在两个不同的 Nginx 请求之间共享 cosocket 对象。

对于任何一个 cosocket 对象的底层连接，如果你没有显式关闭（通过 [close](#tcpsockclose)）或把它放到连接池中（通过 [setkeepalive](#tcpsocksetkeepalive)），一旦下面的的两个事件中任何一个发生，它将被自动关闭：

* 当前请求处理执行完毕
* Lua cosocket 对象被 Lua GC（垃圾回收机制）回收

进行 cosocket 操作时发生致命错误总是会自动关闭当前连接（注意，读超时是这里唯一的非致命错误），并且如果你对一个已经关闭的连接调用 [close](#tcpsockclose) ，你将得到 "closed" 的错误信息。

从 `0.9.9` 版本开始，cosocket 对象是全双工的，也就是说，一个专门读取的 "light thread"，一个专门写入的 "light thread"，它们可以同时对同一个 cosocket 对象进行操作（两个 "light threads" 必须运行在同一个 Lua 环境中，原因见上）。但是你不能让两个 "light threads" 对同一个 cosocket 对象都进行读（或者写入、或者连接）操作，否则当调用 cosocket 对象时，你将得到一个类似 "socket busy reading" 的错误。

该特性在 `v0.5.0rc1` 版本首次引入。

也可以看看 [ngx.socket.udp](#ngxsocketudp)。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.socket.tcp
--------------
**syntax:** *tcpsock = ngx.socket.tcp()*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Creates and returns a TCP or stream-oriented unix domain socket object (also known as one type of the "cosocket" objects). The following methods are supported on this object:

* [connect](#tcpsockconnect)
* [sslhandshake](#tcpsocksslhandshake)
* [send](#tcpsocksend)
* [receive](#tcpsockreceive)
* [close](#tcpsockclose)
* [settimeout](#tcpsocksettimeout)
* [setoption](#tcpsocksetoption)
* [receiveuntil](#tcpsockreceiveuntil)
* [setkeepalive](#tcpsocksetkeepalive)
* [getreusedtimes](#tcpsockgetreusedtimes)

It is intended to be compatible with the TCP API of the [LuaSocket](http://w3.impa.br/~diego/software/luasocket/tcp.html) library but is 100% nonblocking out of the box. Also, we introduce some new APIs to provide more functionalities.

The cosocket object created by this API function has exactly the same lifetime as the Lua handler creating it. So never pass the cosocket object to any other Lua handler (including ngx.timer callback functions) and never share the cosocket object between different NGINX requests.

For every cosocket object's underlying connection, if you do not
explicitly close it (via [close](#tcpsockclose)) or put it back to the connection
pool (via [setkeepalive](#tcpsocksetkeepalive)), then it is automatically closed when one of
the following two events happens:

* the current request handler completes, or
* the Lua cosocket object value gets collected by the Lua GC.

Fatal errors in cosocket operations always automatically close the current
connection (note that, read timeout error is the only error that is
not fatal), and if you call [close](#tcpsockclose) on a closed connection, you will get
the "closed" error.

Starting from the `0.9.9` release, the cosocket object here is full-duplex, that is, a reader "light thread" and a writer "light thread" can operate on a single cosocket object simultaneously (both "light threads" must belong to the same Lua handler though, see reasons above). But you cannot have two "light threads" both reading (or writing or connecting) the same cosocket, otherwise you might get an error like "socket busy reading" when calling the methods of the cosocket object.

This feature was first introduced in the `v0.5.0rc1` release.

See also [ngx.socket.udp](#ngxsocketudp).

[Back to TOC](#nginx-api-for-lua)


