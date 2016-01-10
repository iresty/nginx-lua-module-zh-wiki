ngx.req.set_body_data
---------------------
**语法:** *ngx.req.set_body_data(data)*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

使用 `data` 参数指定的内存数据设置当前请求的请求体。

如果当前请求的请求体尚未被读取，它将被安全地丢弃。当请求体已经被读进内存或缓存在磁盘文件中时，相应的内存或磁盘文件将被立即清理回收。

这个函数在 `v0.3.1rc18`` 版本中首次引入。

更多用法请参考 [ngx.req.set_body_file](#ngxreqset_body_file)。

> English Source

**syntax:** *ngx.req.set_body_data(data)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

Set the current request's request body using the in-memory data specified by the `data` argument.

If the current request's request body has not been read, then it will be properly discarded. When the current request's request body has been read into memory or buffered into a disk file, then the old request body's memory will be freed or the disk file will be cleaned up immediately, respectively.

This function was first introduced in the `v0.3.1rc18` release.

See also [ngx.req.set_body_file](#ngxreqset_body_file).

[返回目录](#nginx-api-for-lua)
