ngx.balancer
------------
**语法:** *local balancer = require "ngx.balancer"*

该 Lua 模块允许在纯 Lua 中可以完全自定义动态负载均衡。

该模块不是与当前 ngx_lua 模块绑定运行，而是与 [lua-resty-core](https://github.com/openresty/lua-resty-core) 库一起运行。

对于 [lua-resty-core](https://github.com/openresty/lua-resty-core) 的 `ngx.balancer` Lua 模块更多信息，请参考 [documentation](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/balancer.md) 。

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