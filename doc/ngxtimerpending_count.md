ngx.timer.pending_count
-----------------------
**语法:** *count = ngx.timer.pending_count()*

**环境:** *init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

返回待定运行的 `timers` 数量。

该指令从 `v0.9.20` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.timer.pending_count
-----------------------
**syntax:** *count = ngx.timer.pending_count()*

**context:** *init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

Returns the number of pending timers.

This directive was first introduced in the `v0.9.20` release.

[Back to TOC](#nginx-api-for-lua)
