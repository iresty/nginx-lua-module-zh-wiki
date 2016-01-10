**语法:** *str = ngx.localtime()*

**环境:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

返回nginx时间缓存(不像Lua的[os.date](http://www.lua.org/manual/5.1/manual.html#pdf-os.date)函数，该时间不涉及系统调用)的当前时间戳(格式：`yyyy-mm-dd hh:mm:ss`)。

这是个本地时间。

> English Source

**syntax:** *str = ngx.localtime()*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns the current time stamp (in the format `yyyy-mm-dd hh:mm:ss`) of the nginx cached time (no syscall involved unlike Lua's [os.date](http://www.lua.org/manual/5.1/manual.html#pdf-os.date) function).

This is the local time.

[返回目录](#nginx-api-for-lua)
