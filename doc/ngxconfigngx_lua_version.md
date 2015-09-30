ngx.config.ngx_lua_version
--------------------------

**语法:** *ver = ngx.config.ngx_lua_version*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, init_by_lua**

这个字段指名当前正在使用的 `ngx_lua` 模块版本数字标识。例如，版本 `0.9.3` 用 Lua 数字表示就是 9003 。

该 API 在 `0.9.3` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.config.ngx_lua_version
--------------------------

**syntax:** *ver = ngx.config.ngx_lua_version*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, init_by_lua**

This field take an integral value indicating the version number of the current `ngx_lua` module being used. For example, the version number `0.9.3` results in the Lua number 9003.

This API was first introduced in the `0.9.3` release.

[Back to TOC](#nginx-api-for-lua)
