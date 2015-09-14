ngx.req.clear_header
--------------------
**语法:** *ngx.req.clear_header(header_name)*

**环境:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\*

清除当前请求的名为 `header_name` 的请求头信息。已经存在的子请求不受影响，此命令之后发起的子请求将默认继承修改后的头信息。


> English Source

**syntax:** *ngx.req.clear_header(header_name)*

**context:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\*

Clears the current request's request header named `header_name`. None of the current request's existing subrequests will be affected but subsequently initiated subrequests will inherit the change by default.

[返回目录](#nginx-api-for-lua)