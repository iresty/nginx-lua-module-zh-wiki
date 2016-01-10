ngx.req.get_method
------------------
**语法:** *method_name = ngx.req.get_method()*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, balancer_by_lua&#42;*

获取当前请求的 HTTP 请求方法名称。结果为类似 `"GET"` 和 `"POST"` 的字符串，而不是 [HTTP 方法常量](#http-method-constants) 中定义的数值。

如果当前请求为 Nginx 子请求，将返回子请求的 HTTP 请求方法名称。

这个方法在 `v0.5.6` 版本中首次引入。

更多用法请参考 [ngx.req.set_method](#ngxreqset_method)。


> English Source

**syntax:** *method_name = ngx.req.get_method()*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, balancer_by_lua&#42;*

Retrieves the current request's request method name. Strings like `"GET"` and `"POST"` are returned instead of numerical [method constants](#http-method-constants).

If the current request is an Nginx subrequest, then the subrequest's method name will be returned.

This method was first introduced in the `v0.5.6` release.

See also [ngx.req.set_method](#ngxreqset_method).

[返回目录](#nginx-api-for-lua)
