coroutine.running
-----------------
**语法:** *co = coroutine.running()*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, init_by_lua*, ngx.timer.*, header_filter_by_lua*, body_filter_by_lua**

与标准的 Lua [coroutine.running](http://www.lua.org/manual/5.1/manual.html#pdf-coroutine.running) API 相同。

该 API 在 [init_by_lua*](#init_by_lua) 的环境中可用，是从 `0.9.2` 开始的。

该 API 在 `v0.6.0` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

coroutine.running
-----------------
**syntax:** *co = coroutine.running()*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, init_by_lua*, ngx.timer.*, header_filter_by_lua*, body_filter_by_lua**

Identical to the standard Lua [coroutine.running](http://www.lua.org/manual/5.1/manual.html#pdf-coroutine.running) API.

This API was first usable in the context of [init_by_lua*](#init_by_lua) since the `0.9.2`.

This API was first enabled in the `v0.6.0` release.

[Back to TOC](#nginx-api-for-lua)

