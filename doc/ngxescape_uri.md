ngx.escape_uri
--------------
**语法:** *newstr = ngx.escape_uri(str)*

**环境:** *init_by_lua&#42;, init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

对 `str` 进行 URI 编码。


> English Source

**syntax:** *newstr = ngx.escape_uri(str)*

**context:** *init_by_lua&#42;, init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Escape `str` as a URI component.


[返回目录](#nginx-api-for-lua)