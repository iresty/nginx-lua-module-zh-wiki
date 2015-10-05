
tcpsock:settimeout
------------------
**语法:** *tcpsock:settimeout(time)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

对 [connect](#tcpsockconnect)、[receive](#tcpsockreceive)、 基于 [receiveuntil](#tcpsockreceiveuntil) 迭代返回，设置随后的 socket 操作的超时时间（毫秒为单位）。

通过该方法设置内容相比这些配置指令有更高的优先级，例如：[lua_socket_connect_timeout](#lua_socket_connect_timeout)、 [lua_socket_send_timeout](#lua_socket_send_timeout)、 [lua_socket_read_timeout](#lua_socket_read_timeout)。

注意，该方法 *不* 会对 [lua_socket_keepalive_timeout](#lua_socket_keepalive_timeout) 有任何影响，这个目的应换用 [setkeepalive](#tcpsocksetkeepalive) 的 `timeout` 参数。

该特性是在 `v0.5.0rc1` 版本首次引入的。

[返回目录](#nginx-api-for-lua)

> English source:


tcpsock:settimeout
------------------
**syntax:** *tcpsock:settimeout(time)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Set the timeout value in milliseconds for subsequent socket operations ([connect](#tcpsockconnect), [receive](#tcpsockreceive), and iterators returned from [receiveuntil](#tcpsockreceiveuntil)).

Settings done by this method takes priority over those config directives, i.e., [lua_socket_connect_timeout](#lua_socket_connect_timeout), [lua_socket_send_timeout](#lua_socket_send_timeout), and [lua_socket_read_timeout](#lua_socket_read_timeout).

Note that this method does *not* affect the [lua_socket_keepalive_timeout](#lua_socket_keepalive_timeout) setting; the `timeout` argument to the [setkeepalive](#tcpsocksetkeepalive) method should be used for this purpose instead.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)
