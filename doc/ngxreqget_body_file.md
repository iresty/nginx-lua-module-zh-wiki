ngx.req.get_body_file
---------------------
**语法:** *file_name = ngx.req.get_body_file()*

**环境:** *rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

获取存储请求体数据的临时文件名。如果请求体尚未被读取或已被读取到内存中，此函数将返回 `nil`。

本函数返回的文件是只读的，通常情况下会被 Nginx 的内存池清理机制清理。不应该手工修改、更名或删除这个文件。

如果请求体尚未被读取，请先调用 [ngx.req.read_body](#ngxreqread_body) (或打开 [lua_need_request_body](#lua_need_request_body) 选项强制本模块读取请求体。此方法不推荐）。

如果请求体已经被读入内存，请使用 [ngx.req.get_body_data](#ngxreqget_body_data) 函数代替。

如需要强制在临时文件中保存请求体，请打开 [client_body_in_file_only](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_in_file_only) 选项。

这个函数在 `v0.3.1rc17`` 版本中首次引入。

更多用法请参考 [ngx.req.get_body_data](#ngxreqget_body_data)。


> English Source

**syntax:** *file_name = ngx.req.get_body_file()*

**context:** *rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

Retrieves the file name for the in-file request body data. Returns `nil` if the request body has not been read or has been read into memory.

The returned file is read only and is usually cleaned up by Nginx's memory pool. It should not be manually modified, renamed, or removed in Lua code.

If the request body has not been read yet, call [ngx.req.read_body](#ngxreqread_body) first (or turned on [lua_need_request_body](#lua_need_request_body) to force this module to read the request body. This is not recommended however).

If the request body has been read into memory, try calling the [ngx.req.get_body_data](#ngxreqget_body_data) function instead.

To force in-file request bodies, try turning on [client_body_in_file_only](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_in_file_only).

This function was first introduced in the `v0.3.1rc17` release.

See also [ngx.req.get_body_data](#ngxreqget_body_data).

[返回目录](#nginx-api-for-lua)
