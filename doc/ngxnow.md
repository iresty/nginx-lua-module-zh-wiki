**语法:** *secs = ngx.now()*

**环境:** *init_worker_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

返回一个浮点型的数字，该数字是从新纪元到从nginx时间缓存(不像Lua的日期库，该时间不涉及系统调用)获取的当前时间戳所经过的时间(以秒为单位，小数部分是毫秒)。

通过先调用[ngx.update_time](#ngxupdate_time)，你可以强制更新nginx时间缓存。

这个API最早出现在 `v0.3.1rc32` 版本中。

> English Source

**syntax:** *secs = ngx.now()*

**context:** *init_worker_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

Returns a floating-point number for the elapsed time in seconds (including milliseconds as the decimal part) from the epoch for the current time stamp from the nginx cached time (no syscall involved unlike Lua's date library).

You can forcibly update the Nginx time cache by calling [ngx.update_time](#ngxupdate_time) first.

This API was first introduced in v0.3.1rc32.

[返回目录](#nginx-api-for-lua)
