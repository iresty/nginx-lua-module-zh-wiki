lua_socket_log_errors
---------------------

**语法:** *lua_socket_log_errors on|off*

**默认:** *lua_socket_log_errors on*

**环境:** *http, server, location*

当TCP 或 UDP cosockets出现失败时，该指令可被用来切换错误日志输出。如果你已经正确处理了你的 Lua 代码错误日志，这里就推荐设置当前指令的开关为 off ，防止数据刷写到你的 nginx 错误日志文件（通常这个代价是比较昂贵的）。

这个指令最早出现在版本 `v0.5.13` 中。

[Back to TOC](#directives)

> English source:

lua_socket_log_errors
---------------------

**syntax:** *lua_socket_log_errors on|off*

**default:** *lua_socket_log_errors on*

**context:** *http, server, location*

This directive can be used to toggle error logging when a failure occurs for the TCP or UDP cosockets. If you are already doing proper error handling and logging in your Lua code, then it is recommended to turn this directive off to prevent data flushing in your nginx error log files (which is usually rather expensive).

This directive was first introduced in the `v0.5.13` release.

[Back to TOC](#directives)
