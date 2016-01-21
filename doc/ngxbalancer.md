ngx.balancer
------------
**语法:** *local balancer = require "ngx.balancer"*

这是一个允许使用纯 Lua 语言，完整定义一个动态负载均衡 Lua API 模块。

该模块没与 ngx_lua 模块一起发行，而是与 [lua-resty-core](https://github.com/openresty/lua-resty-core) 库一起发行。

为获取更多信息，请参考 [lua-resty-core](https://github.com/openresty/lua-resty-core) Lua　模块的 `ngx.balancer` 章节，[文档](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/balancer.md) 。

该特性在 `v0.10.0` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.balancer
------------
**syntax:** *local balancer = require "ngx.balancer"*

This is a Lua module that provides a Lua API to allow defining completely dynamic load balancers
in pure Lua.

This Lua module does not ship with this ngx_lua module itself rather it is shipped with
the
[lua-resty-core](https://github.com/openresty/lua-resty-core) library.

Please refer to the [documentation](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/balancer.md)
for this `ngx.balancer` Lua module in [lua-resty-core](https://github.com/openresty/lua-resty-core)
for more details.

This feature requires at least ngx_lua `v0.10.0`.

[Back to TOC](#nginx-api-for-lua)