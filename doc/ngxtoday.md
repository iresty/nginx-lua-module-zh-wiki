**语法:** *str = ngx.today()*

**环境:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

从nginx的时间缓存(不像Lua的日期库，该时间不涉及系统调用)返回当前的日期(格式：`yyyy-mm-dd`)。

这是个本地时间。

> English Source

**syntax:** *str = ngx.today()*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns current date (in the format `yyyy-mm-dd`) from the nginx cached time (no syscall involved unlike Lua's date library).

This is the local time.

[返回目录](#nginx-api-for-lua)
