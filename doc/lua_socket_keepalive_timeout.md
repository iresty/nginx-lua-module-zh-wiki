lua_socket_keepalive_timeout
----------------------------

**语法:** *lua_socket_keepalive_timeout &lt;time&gt;*

**默认:** *lua_socket_keepalive_timeout 60s*

**环境:** *http, server, location*

该指令控制在 cosocket 连接池中连接的默认最大空闲时间。当这个时间到达，空闲的连接将被关闭并从连接池中移除。这个值可以使用 cosocket 对象的[setkeepalive](#tcpsocksetkeepalive)方法覆盖。

`<time>`参数可以是整数，后面可以跟着像`s` (秒), `ms` (毫秒), `m` (分钟)的单位可选项。 默认的时间单位是`s`，也就是"秒"。默认值是`60s`。

这个指令最早出现在版本 `v0.5.0rc1` 。

[返回目录](#directives)

> English source:

lua_socket_keepalive_timeout
----------------------------

**syntax:** *lua_socket_keepalive_timeout &lt;time&gt;*

**default:** *lua_socket_keepalive_timeout 60s*

**context:** *http, server, location*

This directive controls the default maximal idle time of the connections in the cosocket built-in connection pool. When this timeout reaches, idle connections will be closed and removed from the pool. This setting can be overridden by cosocket objects' [setkeepalive](#tcpsocksetkeepalive) method.

The `<time>` argument can be an integer, with an optional time unit, like `s` (second), `ms` (millisecond), `m` (minute). The default time unit is `s`, i.e., "second". The default setting is `60s`.

This directive was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#directives)