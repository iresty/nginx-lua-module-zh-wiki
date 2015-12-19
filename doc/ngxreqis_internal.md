ngx.req.is_internal
-------------------
**语法:** *is_internal = ngx.req.is_internal()*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua**

返回一个布尔值，说明当前请求是否是一个“内部请求”，既：一个请求的初始化是在当前 nginx 服务端完成初始化，不是在客户端。

子请求都是内部请求，并且都是内部重定向后的请求。

该 API 在 `v0.9.20` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.req.is_internal
-------------------
**syntax:** *is_internal = ngx.req.is_internal()*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua**

Returns a boolean indicating whether the current request is an "internal request", i.e.,
a request initiated from inside the current nginx server instead of from the client side.

Subrequests are all internal requests and so are requests after internal redirects.

This API was first introduced in the `v0.9.20` release.

[Back to TOC](#nginx-api-for-lua)
