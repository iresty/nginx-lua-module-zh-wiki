lua_socket_connect_timeout
--------------------------

**语法:** *lua_socket_connect_timeout &lt;time&gt;*

**默认:** *lua_socket_connect_timeout 60s*

**环境:** *http, server, location*

该指令控制 TCP/unix-domain 套接字对象的[connect](#tcpsockconnect)方法默认超时时间，这个值可以被[settimeout](#tcpsocksettimeout)方法覆盖。

`<time>`参数可以是整数，后面可以跟着像`s` (秒), `ms` (毫秒), `m` (分钟)的单位可选项。 默认的时间单位是`s`，也就是"秒"。默认值是`60s`。

这个指令最早出现在版本 `v0.5.0rc1` 中。

> English source:

lua_socket_connect_timeout
--------------------------

**syntax:** *lua_socket_connect_timeout &lt;time&gt;*

**default:** *lua_socket_connect_timeout 60s*

**context:** *http, server, location*

This directive controls the default timeout value used in TCP/unix-domain socket object's [connect](#tcpsockconnect) method and can be overridden by the [settimeout](#tcpsocksettimeout) method.

The `<time>` argument can be an integer, with an optional time unit, like `s` (second), `ms` (millisecond), `m` (minute). The default time unit is `s`, i.e., "second". The default setting is `60s`.

This directive was first introduced in the `v0.5.0rc1` release.
