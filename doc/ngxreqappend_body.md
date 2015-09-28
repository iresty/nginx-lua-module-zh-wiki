ngx.req.append_body
-------------------
**语法:** *ngx.req.append_body(data_chunk)*

**环境:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

向已存在的请求体中追加写入 `data_chunk` 参数指定的新数据块，此请求体最初使用 [ngx.req.init_body](#ngxreqinit_body) 创建。

当请求体数据过大，不再能保存在内存缓冲区中时，数据将被写入一个临时文件，类似 Nginx 内核中的标准请求体处理方式。

需要强调的是，在当前请求的所有请求体被写入完成后，必须调用 [ngx.req.finish_body](#ngxreqfinish_body) 以结束写入。

此函数可以与 [ngx.req.init_body](#ngxreqinit_body)，[ngx.req.finish_body](#ngxreqfinish_body)，和 [ngx.req.socket](#ngxreqsocket) 一起，使用纯 Lua 语言实现高效的输入过滤器 (在 [rewrite_by_lua](#rewrite_by_lua)* 或 [access_by_lua](#access_by_lua)* 环境中)，与其他 Nginx 内容处理程序或上游模块例如 [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) 和 [ngx_http_fastcgi_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html) 配合使用。
<!--todo content handler 翻译需要统一 -->

这个函数在 `v0.5.11` 版本中首次引入。

更多用法请参考 [ngx.req.init_body](#ngxreqinit_body)。


> English Source

**syntax:** *ngx.req.append_body(data_chunk)*

**context:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

Append new data chunk specified by the `data_chunk` argument onto the existing request body created by the [ngx.req.init_body](#ngxreqinit_body) call.

When the data can no longer be hold in the memory buffer for the request body, then the data will be flushed onto a temporary file just like the standard request body reader in the Nginx core.

It is important to always call the [ngx.req.finish_body](#ngxreqfinish_body) after all the data has been appended onto the current request body.

This function can be used with [ngx.req.init_body](#ngxreqinit_body), [ngx.req.finish_body](#ngxreqfinish_body), and [ngx.req.socket](#ngxreqsocket) to implement efficient input filters in pure Lua (in the context of [rewrite_by_lua](#rewrite_by_lua)* or [access_by_lua](#access_by_lua)*), which can be used with other Nginx content handler or upstream modules like [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) and [ngx_http_fastcgi_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html).

This function was first introduced in the `v0.5.11` release.

See also [ngx.req.init_body](#ngxreqinit_body).

[返回目录](#nginx-api-for-lua)
