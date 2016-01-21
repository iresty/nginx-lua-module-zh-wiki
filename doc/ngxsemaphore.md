ngx.semaphore
-------------
**语法:** *local semaphore = require "ngx.semaphore"*

该 Lua 模块，实现了一个经典的 semaphore API，可以高效的完成不同“轻线程”之间的同步。在不同的请求环境中创建的“轻线程”，可以共享同一个 semaphore 对象是支持的，只要这些“轻线程”是在同一个 nginx 工作进程中即可，此外要求 [lua_code_cache](#lua_code_cache) 指令是开启的（默认是开启的）。

该模块没与 ngx_lua 模块一起发行，而是与 [lua-resty-core](https://github.com/openresty/lua-resty-core) 库一起发行。

获取更多信息，请参考Lua 模块　[lua-resty-core](https://github.com/openresty/lua-resty-core)　的　`ngx.semaphore`　信息，[文档](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/semaphore.md)。

该特性在　`v0.10.0`　版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.semaphore
-------------
**syntax:** *local semaphore = require "ngx.semaphore"*

This is a Lua module that implements a classic-style semaphore API for efficient synchronizations among
different "light threads". Sharing the same semaphore among different "light threads" created in different (request)
contexts are also supported as long as the "light threads" reside in the same NGINX worker process
and the [lua_code_cache](#lua_code_cache) directive is turned on (which is the default).

This Lua module does not ship with this ngx_lua module itself rather it is shipped with
the
[lua-resty-core](https://github.com/openresty/lua-resty-core) library.

Please refer to the [documentation](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/semaphore.md)
for this `ngx.semaphore` Lua module in [lua-resty-core](https://github.com/openresty/lua-resty-core)
for more details.

This feature requires at least ngx_lua `v0.10.0`.

[Back to TOC](#nginx-api-for-lua)
