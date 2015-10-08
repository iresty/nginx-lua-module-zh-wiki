
ngx.is_subrequest
-----------------
**语法:** *value = ngx.is_subrequest*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua**

如果当前请求是 nginx 子请求返回 `true` ，否则返回 `false` 。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.is_subrequest
-----------------
**syntax:** *value = ngx.is_subrequest*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua**

Returns `true` if the current request is an nginx subrequest, or `false` otherwise.

[Back to TOC](#nginx-api-for-lua)
