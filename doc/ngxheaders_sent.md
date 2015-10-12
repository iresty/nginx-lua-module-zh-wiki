ngx.headers_sent
----------------
**语法:** *value = ngx.headers_sent*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua**

如果应答头部已经被发送（通过 ngx_lua）返回 `true` ，否则返回 `false` 。

该 API 是在 `v0.3.1rc6` 版本首次引入的。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.headers_sent
----------------
**syntax:** *value = ngx.headers_sent*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua**

Returns `true` if the response headers have been sent (by ngx_lua), and `false` otherwise.

This API was first introduced in ngx_lua v0.3.1rc6.

[Back to TOC](#nginx-api-for-lua)

