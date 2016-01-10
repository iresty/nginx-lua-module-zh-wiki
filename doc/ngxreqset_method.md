ngx.req.set_method
------------------
**语法:** *ngx.req.set_method(method_id)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;*

用 `method_id` 参数的值改写当前请求的 HTTP 请求方法。当前仅支持 [HTTP 请求方法](#http-method-constants) 中定义的数值常量，例如 `ngx.HTTP_POST` 和 `ngx.HTTP_GET`。

如果当前请求是 Nginx 子请求，子请求的 HTTP 请求方法将被改写。

这个方法在 `v0.5.6` 版本中首次引入。

更多用法请参考 [ngx.req.get_method](#ngxreqget_method)。


> English Source

**syntax:** *ngx.req.set_method(method_id)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;*

Overrides the current request's request method with the `method_id` argument. Currently only numerical [method constants](#http-method-constants) are supported, like `ngx.HTTP_POST` and `ngx.HTTP_GET`.

If the current request is an Nginx subrequest, then the subrequest's method will be overridden.

This method was first introduced in the `v0.5.6` release.

See also [ngx.req.get_method](#ngxreqget_method).

[返回目录](#nginx-api-for-lua)
