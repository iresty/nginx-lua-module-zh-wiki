ngx.get_phase
-------------
**语法:** *str = ngx.get_phase()*

**环境:** *init_by_lua*, init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

检索当前正在执行的阶段名称。返回值可能有：
Retrieves the current running phase name. Possible return values are

* `init`
    [init_by_lua](#init_by_lua) 或 [init_by_lua_file](#init_by_lua_file) 的运行环境。
* `init_worker`
    [init_worker_by_lua](#init_worker_by_lua) 或 [init_worker_by_lua_file](#init_worker_by_lua_file) 的运行环境。
* `set`
    [set_by_lua](#set_by_lua) 或 [set_by_lua_file](#set_by_lua_file) 的运行环境。
* `rewrite`
    [rewrite_by_lua](#rewrite_by_lua) 或 [rewrite_by_lua_file](#rewrite_by_lua_file) 的运行环境。
* `access`
    [access_by_lua](#access_by_lua) 或 [access_by_lua_file](#access_by_lua_file)。
* `content`
    [content_by_lua](#content_by_lua) 或 [content_by_lua_file](#content_by_lua_file) 的运行环境。
* `header_filter`
    [header_filter_by_lua](#header_filter_by_lua) 或 [header_filter_by_lua_file](#header_filter_by_lua_file) 的运行环境。
* `body_filter`
    [body_filter_by_lua](#body_filter_by_lua) 或 [body_filter_by_lua_file](#body_filter_by_lua_file) 的运行环境。
* `log`
    [log_by_lua](#log_by_lua) 或 [log_by_lua_file](#log_by_lua_file) 的运行环境。
* `timer`
    [ngx.timer.*](#ngxtimerat) 类的用户回调函数运行环境。

该 API 是从 `v0.5.10` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.get_phase
-------------
**syntax:** *str = ngx.get_phase()*

**context:** *init_by_lua*, init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

Retrieves the current running phase name. Possible return values are

* `init`
    for the context of [init_by_lua](#init_by_lua) or [init_by_lua_file](#init_by_lua_file).
* `init_worker`
    for the context of [init_worker_by_lua](#init_worker_by_lua) or [init_worker_by_lua_file](#init_worker_by_lua_file).
* `set`
    for the context of [set_by_lua](#set_by_lua) or [set_by_lua_file](#set_by_lua_file).
* `rewrite`
    for the context of [rewrite_by_lua](#rewrite_by_lua) or [rewrite_by_lua_file](#rewrite_by_lua_file).
* `access`
    for the context of [access_by_lua](#access_by_lua) or [access_by_lua_file](#access_by_lua_file).
* `content`
    for the context of [content_by_lua](#content_by_lua) or [content_by_lua_file](#content_by_lua_file).
* `header_filter`
    for the context of [header_filter_by_lua](#header_filter_by_lua) or [header_filter_by_lua_file](#header_filter_by_lua_file).
* `body_filter`
    for the context of [body_filter_by_lua](#body_filter_by_lua) or [body_filter_by_lua_file](#body_filter_by_lua_file).
* `log`
    for the context of [log_by_lua](#log_by_lua) or [log_by_lua_file](#log_by_lua_file).
* `timer`
    for the context of user callback functions for [ngx.timer.*](#ngxtimerat).

This API was first introduced in the `v0.5.10` release.

[Back to TOC](#nginx-api-for-lua)

