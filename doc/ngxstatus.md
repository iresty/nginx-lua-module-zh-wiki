ngx.status
----------
**环境:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\*

读写当前请求的响应状态码。这个方法需要在发送响应头前调用。

```lua

 ngx.status = ngx.HTTP_CREATED
 status = ngx.status
```

在发送响应头之后设置 `ngx.status` 不会生效，且 nginx 的错误日志中会有下面一条记录：


    attempt to set ngx.status after sending out response headers


> English Source

**context:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\*

Read and write the current request's response status. This should be called
before sending out the response headers.

```lua

 ngx.status = ngx.HTTP_CREATED
 status = ngx.status
```

Setting `ngx.status` after the response header is sent out has no effect but leaving an error message in your nginx's error log file:


    attempt to set ngx.status after sending out response headers


[Back to TOC](#nginx-api-for-lua)