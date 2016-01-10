ngx.req.get_body_data
---------------------
**语法:** *data = ngx.req.get_body_data()*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

取回内存中的请求体数据。本函数返回 Lua 字符串而不是包含解析过参数的 Lua table。如果想要返回 Lua table，请使用 [ngx.req.get_post_args](#ngxreqget_post_args) 函数。

当以下情况时，此函数返回 `nil`，

1. 请求体尚未被读取，
1. 请求体已经被存入磁盘上的临时文件，
1. 或请求体大小是 0。

如果请求体尚未被读取，请先调用 [ngx.req.read_body](#ngxreqread_body) (或打开 [lua_need_request_body](#lua_need_request_body) 选项强制本模块读取请求体。此方法不推荐）。

如果请求体已经被存入临时文件，请使用 [ngx.req.get_body_file](#ngxreqget_body_file) 函数代替。

如需要强制在内存中保存请求体，请设置 [client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size) 和 [client_max_body_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size) 为同样大小。

请注意，调用此函数比使用 `ngx.var.request_body` 或 `ngx.var.echo_request_body` 更有效率，因为本函数能够节省一次内存分配与数据复制。

这个函数在 `v0.3.1rc17`` 版本中首次引入。

更多用法请参考 [ngx.req.get_body_file](#ngxreqget_body_file)。


> English Source

**syntax:** *data = ngx.req.get_body_data()*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

Retrieves in-memory request body data. It returns a Lua string rather than a Lua table holding all the parsed query arguments. Use the [ngx.req.get_post_args](#ngxreqget_post_args) function instead if a Lua table is required.

This function returns `nil` if

1. the request body has not been read,
1. the request body has been read into disk temporary files,
1. or the request body has zero size.

If the request body has not been read yet, call [ngx.req.read_body](#ngxreqread_body) first (or turned on [lua_need_request_body](#lua_need_request_body) to force this module to read the request body. This is not recommended however).

If the request body has been read into disk files, try calling the [ngx.req.get_body_file](#ngxreqget_body_file) function instead.

To force in-memory request bodies, try setting [client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size) to the same size value in [client_max_body_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size).

Note that calling this function instead of using `ngx.var.request_body` or `ngx.var.echo_request_body` is more efficient because it can save one dynamic memory allocation and one data copy.

This function was first introduced in the `v0.3.1rc17` release.

See also [ngx.req.get_body_file](#ngxreqget_body_file).

[返回目录](#nginx-api-for-lua)
