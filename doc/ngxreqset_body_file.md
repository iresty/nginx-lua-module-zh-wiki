ngx.req.set_body_file
---------------------
**语法:** *ngx.req.set_body_file(file_name, auto_clean?)*

**环境:** *rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

使用 `file_name` 参数指定的数据文件设置当前请求的请求体。

当可选参数 `auto_clean` 设置为 `true` 时，在本次请求完成，或本次请求内再次调用本函数或 [ngx.req.set_body_data](#ngxreqset_body_data) 时，`file_name` 文件将被删除。`auto_clean` 默认值是 `false`。

请确保 `file_name` 参数指定的文件存在，并设置合适的操作权限对 Nginx worker 可读，以避免抛出 Lua 异常。

如果当前请求的请求体尚未被读取，它将被安全地丢弃。当请求体已经被读进内存或缓存在磁盘文件中时，相应的内存或磁盘文件将被立即清理回收。

这个函数在 `v0.3.1rc18`` 版本中首次引入。

更多用法请参考 [ngx.req.set_body_data](#ngxreqset_body_data)。


> English Source

**syntax:** *ngx.req.set_body_file(file_name, auto_clean?)*

**context:** *rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

Set the current request's request body using the in-file data specified by the `file_name` argument.

If the optional `auto_clean` argument is given a `true` value, then this file will be removed at request completion or the next time this function or [ngx.req.set_body_data](#ngxreqset_body_data) are called in the same request. The `auto_clean` is default to `false`.

Please ensure that the file specified by the `file_name` argument exists and is readable by an Nginx worker process by setting its permission properly to avoid Lua exception errors.

If the current request's request body has not been read, then it will be properly discarded. When the current request's request body has been read into memory or buffered into a disk file, then the old request body's memory will be freed or the disk file will be cleaned up immediately, respectively.

This function was first introduced in the `v0.3.1rc18` release.

See also [ngx.req.set_body_data](#ngxreqset_body_data).

[返回目录](#nginx-api-for-lua)
