
udpsock:settimeout
------------------
**语法:** *udpsock:settimeout(time)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

设置随后的 socket 操作（例如 [receive](#udpsockreceive)）的超时时间（毫秒为单位）。

通过该方法设置内容相比这些配置指令有更高的优先级，例如：[lua_socket_read_timeout](#lua_socket_read_timeout)。

该特性在 `v0.5.7` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

udpsock:settimeout
------------------
**syntax:** *udpsock:settimeout(time)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Set the timeout value in milliseconds for subsequent socket operations (like [receive](#udpsockreceive)).

Settings done by this method takes priority over those config directives, like [lua_socket_read_timeout](#lua_socket_read_timeout).

This feature was first introduced in the `v0.5.7` release.

[Back to TOC](#nginx-api-for-lua)
