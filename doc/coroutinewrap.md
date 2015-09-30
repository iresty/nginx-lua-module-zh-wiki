coroutine.wrap
--------------
**语法:** *co = coroutine.wrap(f)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, init_by_lua*, ngx.timer.*, header_filter_by_lua*, body_filter_by_lua**

类似标准的 Lua [coroutine.wrap](http://www.lua.org/manual/5.1/manual.html#pdf-coroutine.wrap) API，但它是在 ngx_lua 创建的 Lua 协程环境中运行。

该 API 在 [init_by_lua*](#init_by_lua) 的环境中可用，是从 `0.9.2` 开始的。

该 API 在 `v0.6.0` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

coroutine.wrap
--------------
**syntax:** *co = coroutine.wrap(f)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, init_by_lua*, ngx.timer.*, header_filter_by_lua*, body_filter_by_lua**

Similar to the standard Lua [coroutine.wrap](http://www.lua.org/manual/5.1/manual.html#pdf-coroutine.wrap) API, but works in the context of the Lua coroutines created by ngx_lua.

This API was first usable in the context of [init_by_lua*](#init_by_lua) since the `0.9.2`.

This API was first introduced in the `v0.6.0` release.

[Back to TOC](#nginx-api-for-lua)
