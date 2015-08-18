ngx.escape_uri
--------------
**语法:** *newstr = ngx.escape_uri(str)*

**环境:** *init_by_lua*\**, init_worker_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

将 `str` 作为 URI 组件进行转义。


> English Source

**syntax:** *newstr = ngx.escape_uri(str)*

**context:** *init_by_lua*\**, init_worker_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

Escape `str` as a URI component.


[返回目录](#nginx-api-for-lua)