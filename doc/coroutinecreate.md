coroutine.create
----------------
**语法:** *co = coroutine.create(f)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, init_by_lua*, ngx.timer.*, header_filter_by_lua*, body_filter_by_lua**

通过一个 Lua 函数创建一个用户的 Lua 协程，并返回一个协程对象。、

类似标准的 Lua [coroutine.create](http://www.lua.org/manual/5.1/manual.html#pdf-coroutine.create) API，但它是在 ngx_lua 创建的 Lua 协程环境中运行。

这个 API 在 [init_by_lua*](#init_by_lua) 的环境中可用，是从 `0.9.2` 开始的。

这个 API 在`v0.6.0`版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

coroutine.create
----------------
**syntax:** *co = coroutine.create(f)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, init_by_lua*, ngx.timer.*, header_filter_by_lua*, body_filter_by_lua**

Creates a user Lua coroutines with a Lua function, and returns a coroutine object.

Similar to the standard Lua [coroutine.create](http://www.lua.org/manual/5.1/manual.html#pdf-coroutine.create) API, but works in the context of the Lua coroutines created by ngx_lua.

This API was first usable in the context of [init_by_lua*](#init_by_lua) since the `0.9.2`.

This API was first introduced in the `v0.6.0` release.

[Back to TOC](#nginx-api-for-lua)
