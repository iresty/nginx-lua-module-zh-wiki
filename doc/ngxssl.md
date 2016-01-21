ngx.ssl
-------
**语法:** *local ssl = require "ngx.ssl"*

该 Lua 模块提供 API 函数，可以在诸如 [ssl_certificate_by_lua*](#ssl_certificate_by_lua_block) 环境中完成 SSL 握手过程的控制。

该模块没与 ngx_lua 模块一起发行，而是与 [lua-resty-core](https://github.com/openresty/lua-resty-core) 库一起发行。

获取更多 `ngx.ssl` Lua 模块信息，请参考 [文档](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/ssl.md) 。

该特性在　`v0.10.0`　版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:


ngx.ssl
-------
**syntax:** *local ssl = require "ngx.ssl"*

This Lua module provides API functions to control the SSL handshake process in contexts like
[ssl_certificate_by_lua*](#ssl_certificate_by_lua_block).

This Lua module does not ship with this ngx_lua module itself rather it is shipped with
the
[lua-resty-core](https://github.com/openresty/lua-resty-core) library.

Please refer to the [documentation](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/ssl.md)
for this `ngx.ssl` Lua module for more details.

This feature requires at least ngx_lua `v0.10.0`.

[Back to TOC](#nginx-api-for-lua)