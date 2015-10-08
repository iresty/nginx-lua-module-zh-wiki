
ngx.socket.udp
--------------
**语法:** *udpsock = ngx.socket.udp()*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

创建并得到一个 UDP 或 unix 域数据报 socket 对象（也被称为 "cosocket" 对象的一种类型）。该对象支持下面这些方法：

* [setpeername](#udpsocksetpeername)
* [send](#udpsocksend)
* [receive](#udpsockreceive)
* [close](#udpsockclose)
* [settimeout](#udpsocksettimeout)

兼容 [LuaSocket](http://w3.impa.br/~diego/software/luasocket/udp.html) 库的 UDP API 是需要的，额外的它却是 100% 非阻塞。

该特性是在 `v0.5.7` 版本首次引入的。

也可以看看 [ngx.socket.tcp](#ngxsockettcp) 。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.socket.udp
--------------
**syntax:** *udpsock = ngx.socket.udp()*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Creates and returns a UDP or datagram-oriented unix domain socket object (also known as one type of the "cosocket" objects). The following methods are supported on this object:

* [setpeername](#udpsocksetpeername)
* [send](#udpsocksend)
* [receive](#udpsockreceive)
* [close](#udpsockclose)
* [settimeout](#udpsocksettimeout)

It is intended to be compatible with the UDP API of the [LuaSocket](http://w3.impa.br/~diego/software/luasocket/udp.html) library but is 100% nonblocking out of the box.

This feature was first introduced in the `v0.5.7` release.

See also [ngx.socket.tcp](#ngxsockettcp).

[Back to TOC](#nginx-api-for-lua)
