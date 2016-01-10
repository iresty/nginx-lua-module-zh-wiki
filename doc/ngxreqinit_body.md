ngx.req.init_body
-----------------
**语法:** *ngx.req.init_body(buffer_size?)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

为当前请求创建一个新的空请求体并初始化缓冲区，为后续通过 [ngx.req.append_body](#ngxreqappend_body) 和 [ngx.req.finish_body](#ngxreqfinish_body) API 写请求体数据做好准备。

如果设置了 `buffer_size` 参数，将设置该大小的内存缓冲区，用于后续的 [ngx.req.append_body](#ngxreqappend_body) 写请求体数据。如果省略此参数，将使用 Nginx 标准指令 [client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size) 中设置的值作为缓冲区大小。

当请求体数据过大，不再能保存在内存缓冲区中时，数据将被写入一个临时文件，类似 Nginx 内核中的标准请求体处理方式。

需要强调的是，在当前请求的所有请求体被写入完成后，必须调用 [ngx.req.finish_body](#ngxreqfinish_body) 以结束写入。另外，当此函数与 [ngx.req.socket](#ngxreqsocket) 一起使用时，需要在执行此函数 *之前* 调用 [ngx.req.socket](#ngxreqsocket)，否则将会报 "request body already exists" (请求体已经存在) 错。

此函数典型用法如下：

```lua

 ngx.req.init_body(128 * 1024)  -- 缓冲区 128KB
 for chunk in next_data_chunk() do
     ngx.req.append_body(chunk) -- 每块可以是 4KB
 end
 ngx.req.finish_body()
```

此函数可以与 [ngx.req.append_body](#ngxreqappend_body)，[ngx.req.finish_body](#ngxreqfinish_body)，和 [ngx.req.socket](#ngxreqsocket) 一起，使用纯 Lua 语言实现高效的输入过滤器 (在 [rewrite_by_lua](#rewrite_by_lua)* 或 [access_by_lua](#access_by_lua)* 环境中)，与其他 Nginx 内容处理程序或上游模块例如 [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) 和 [ngx_http_fastcgi_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html) 配合使用。
<!--todo content handler 翻译需要统一 -->

这个函数在 `v0.5.11` 版本中首次引入。


> English Source

**syntax:** *ngx.req.init_body(buffer_size?)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

Creates a new blank request body for the current request and inializes the buffer for later request body data writing via the [ngx.req.append_body](#ngxreqappend_body) and [ngx.req.finish_body](#ngxreqfinish_body) APIs.

If the `buffer_size` argument is specified, then its value will be used for the size of the memory buffer for body writing with [ngx.req.append_body](#ngxreqappend_body). If the argument is omitted, then the value specified by the standard [client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size) directive will be used instead.

When the data can no longer be hold in the memory buffer for the request body, then the data will be flushed onto a temporary file just like the standard request body reader in the Nginx core.

It is important to always call the [ngx.req.finish_body](#ngxreqfinish_body) after all the data has been appended onto the current request body. Also, when this function is used together with [ngx.req.socket](#ngxreqsocket), it is required to call [ngx.req.socket](#ngxreqsocket) *before* this function, or you will get the "request body already exists" error message.

The usage of this function is often like this:

```lua

 ngx.req.init_body(128 * 1024)  -- buffer is 128KB
 for chunk in next_data_chunk() do
     ngx.req.append_body(chunk) -- each chunk can be 4KB
 end
 ngx.req.finish_body()
```

This function can be used with [ngx.req.append_body](#ngxreqappend_body), [ngx.req.finish_body](#ngxreqfinish_body), and [ngx.req.socket](#ngxreqsocket) to implement efficient input filters in pure Lua (in the context of [rewrite_by_lua](#rewrite_by_lua)* or [access_by_lua](#access_by_lua)*), which can be used with other Nginx content handler or upstream modules like [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) and [ngx_http_fastcgi_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html).

This function was first introduced in the `v0.5.11` release.

[返回目录](#nginx-api-for-lua)
