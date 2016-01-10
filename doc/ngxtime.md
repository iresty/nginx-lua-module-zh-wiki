**语法:** *secs = ngx.time()*

**环境:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

返回从新纪元到从nginx时间缓存(不像Lua的日期库，该时间不涉及系统调用))获取的当前时间戳所经过的秒数。

通过先调用[ngx.update_time](#ngxupdate_time)会强制更新nginx的时间缓存。

> English Source

**syntax:** *secs = ngx.time()*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns the elapsed seconds from the epoch for the current time stamp from the nginx cached time (no syscall involved unlike Lua's date library).

Updates of the Nginx time cache an be forced by calling [ngx.update_time](#ngxupdate_time) first.

[返回目录](#nginx-api-for-lua)
