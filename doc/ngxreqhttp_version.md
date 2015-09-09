ngx.req.http_version
--------------------
**语法:** *num = ngx.req.http_version()*

**环境:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\*

返回一个 Lua 数字代表当前请求的 HTTP 版本号。

当前的可能结果值为 1.0, 1.1 和 0.9。无法识别时值时返回 `nil`。

这个方法在 `v0.7.17` 版本中首次引入。


> English Source

**syntax:** *num = ngx.req.http_version()*

**context:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\*

Returns the HTTP version number for the current request as a Lua number.

Current possible values are 1.0, 1.1, and 0.9. Returns `nil` for unrecognized values.

This method was first introduced in the `v0.7.17` release.

[返回目录](#nginx-api-for-lua)