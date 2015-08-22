**语法:** *str = ngx.utctime()*

**环境:** *init_worker_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

返回nginx时间缓存(不像Lua的[os.date](http://www.lua.org/manual/5.1/manual.html#pdf-os.date)函数，该时间不涉及系统调用)的当前时间戳(格式：`yyyy-mm-dd hh:mm:ss`)。

这是个UTC时间。

> English Source

**syntax:** *str = ngx.utctime()*

**context:** *init_worker_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

Returns the current time stamp (in the format `yyyy-mm-dd hh:mm:ss`) of the nginx cached time (no syscall involved unlike Lua's [os.date](http://www.lua.org/manual/5.1/manual.html#pdf-os.date) function).

This is the UTC time.

[返回目录](#nginx-api-for-lua)
