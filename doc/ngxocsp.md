ngx.ocsp
--------
**语法:** *local ocsp = require "ngx.ocsp"*

该模块提供 API 完成 OCSP 查询、OCSP 响应验证和 OCSP stapling planting 。
<!-- todo，最后的单词，不知道如何翻译 -->

通常，该模块与 [ngx.ssl](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/ssl.md) 模块一起配合在 [ssl_certificate_by_lua*](#ssl_certificate_by_lua_block) 的环境中使用。

该模块没与 ngx_lua 模块一起发行，而是与 [lua-resty-core](https://github.com/openresty/lua-resty-core) 库一起发行。

获取更多 `ngx.ocsp` Lua 模块信息，请参考 [文档](https://github.com/openresty/lua-resty-core/blob/ocsp-cert-by-lua-2/lib/ngx/ocsp.md)。

该特性在　`v0.10.0`　版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.ocsp
--------
**syntax:** *local ocsp = require "ngx.ocsp"*

This Lua module provides API to perform OCSP queries, OCSP response validations, and
OCSP stapling planting.

Usually, this module is used together with the [ngx.ssl](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/ssl.md)
module in the
context of [ssl_certificate_by_lua*](#ssl_certificate_by_lua_block).

This Lua module does not ship with this ngx_lua module itself rather it is shipped with
the
[lua-resty-core](https://github.com/openresty/lua-resty-core) library.

Please refer to the [documentation](https://github.com/openresty/lua-resty-core/blob/ocsp-cert-by-lua-2/lib/ngx/ocsp.md)
for this `ngx.ocsp` Lua module for more details.

This feature requires at least ngx_lua `v0.10.0`.

[Back to TOC](#nginx-api-for-lua)