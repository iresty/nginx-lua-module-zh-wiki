ngx.req.finish_body
-------------------
**语法:** *ngx.req.finish_body()*

**环境:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

结束新请求体构造过程，该请求体由 [ngx.req.init_body](#ngxreqinit_body) 和 [ngx.req.append_body](#ngxreqappend_body) 创建。

此函数可以与 [ngx.req.init_body](#ngxreqinit_body)，[ngx.req.append_body](#ngxreqappend_body)，和 [ngx.req.socket](#ngxreqsocket) 一起，使用纯 Lua 语言实现高效的输入过滤器 (在 [rewrite_by_lua](#rewrite_by_lua)* 或 [access_by_lua](#access_by_lua)* 环境中)，与其他 Nginx 内容处理程序或上游模块例如 [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) 和 [ngx_http_fastcgi_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html) 配合使用。
<!--todo content handler 翻译需要统一 -->

这个函数在 `v0.5.11` 版本中首次引入。

更多用法请参考 [ngx.req.init_body](#ngxreqinit_body)。


> English Source

**syntax:** *ngx.req.finish_body()*

**context:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

Completes the construction process of the new request body created by the [ngx.req.init_body](#ngxreqinit_body) and [ngx.req.append_body](#ngxreqappend_body) calls.

This function can be used with [ngx.req.init_body](#ngxreqinit_body), [ngx.req.append_body](#ngxreqappend_body), and [ngx.req.socket](#ngxreqsocket) to implement efficient input filters in pure Lua (in the context of [rewrite_by_lua](#rewrite_by_lua)* or [access_by_lua](#access_by_lua)*), which can be used with other Nginx content handler or upstream modules like [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) and [ngx_http_fastcgi_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html).

This function was first introduced in the `v0.5.11` release.

See also [ngx.req.init_body](#ngxreqinit_body).

[返回目录](#nginx-api-for-lua)
