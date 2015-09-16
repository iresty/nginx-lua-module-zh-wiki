ngx.req.read_body
-----------------
**语法:** *ngx.req.read_body()*

**环境:** *rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

同步读取客户端请求体，不阻塞 Nginx 事件循环。
Reads the client request body synchronously without blocking the Nginx event loop.

```lua

 ngx.req.read_body()
 local args = ngx.req.get_post_args()
```

如果请求体已经通过打开 [lua_need_request_body](#lua_need_request_body) 选项或其他模块读取，此函数将不会执行，立即返回。
If the request body is already read previously by turning on [lua_need_request_body](#lua_need_request_body) or by using other modules, then this function does not run and returns immediately.

如果请求体已经通过 [ngx.req.discard_body](#ngxreqdiscard_body) 函数或其他模块扔掉，此函数将不会执行，立即返回。
If the request body has already been explicitly discarded, either by the [ngx.req.discard_body](#ngxreqdiscard_body) function or other modules, this function does not run and returns immediately.

当出错时，例如读取数据时连接出错，此方法将立即抛出 Lua 异常 *或* 以 500 状态码中断当前请求。
In case of errors, such as connection errors while reading the data, this method will throw out a Lua exception *or* terminate the current request with a 500 status code immediately.

通过此函数读取的请求体，之后可以通过 [ngx.req.get_body_data](#ngxreqget_body_data) 获得，或者，通过 [ngx.req.get_body_file](#ngxreqget_body_file) 得到请求体数据缓存在磁盘上的临时文件名。这取决于：
The request body data read using this function can be retrieved later via [ngx.req.get_body_data](#ngxreqget_body_data) or, alternatively, the temporary file name for the body data cached to disk using [ngx.req.get_body_file](#ngxreqget_body_file). This depends on

1. 是否当前读求体已经大于 [client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size)，
1. 是否 [client_body_in_file_only](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_in_file_only) 选项被打开。
1. whether the current request body is already larger than the [client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size),
1. and whether [client_body_in_file_only](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_in_file_only) has been switched on.

在当前请求包含请求体，但请求体不需要时，必须使用 [ngx.req.discard_body](#ngxreqdiscard_body) 明确放弃请求体，以避免影响 HTTP 1.1 长连接或 HTTP 1.1 pipelining。
In cases where current request may have a request body and the request body data is not required, The [ngx.req.discard_body](#ngxreqdiscard_body) function must be used to explicitly discard the request body to avoid breaking things under HTTP 1.1 keepalive or HTTP 1.1 pipelining.

This function was first introduced in the `v0.3.1rc17` release.


> English Source

**syntax:** *ngx.req.read_body()*

**context:** *rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

Reads the client request body synchronously without blocking the Nginx event loop.

```lua

 ngx.req.read_body()
 local args = ngx.req.get_post_args()
```

If the request body is already read previously by turning on [lua_need_request_body](#lua_need_request_body) or by using other modules, then this function does not run and returns immediately.

If the request body has already been explicitly discarded, either by the [ngx.req.discard_body](#ngxreqdiscard_body) function or other modules, this function does not run and returns immediately.

In case of errors, such as connection errors while reading the data, this method will throw out a Lua exception *or* terminate the current request with a 500 status code immediately.

The request body data read using this function can be retrieved later via [ngx.req.get_body_data](#ngxreqget_body_data) or, alternatively, the temporary file name for the body data cached to disk using [ngx.req.get_body_file](#ngxreqget_body_file). This depends on

1. whether the current request body is already larger than the [client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size),
1. and whether [client_body_in_file_only](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_in_file_only) has been switched on.

In cases where current request may have a request body and the request body data is not required, The [ngx.req.discard_body](#ngxreqdiscard_body) function must be used to explicitly discard the request body to avoid breaking things under HTTP 1.1 keepalive or HTTP 1.1 pipelining.

This function was first introduced in the `v0.3.1rc17` release.

[返回目录](#nginx-api-for-lua)