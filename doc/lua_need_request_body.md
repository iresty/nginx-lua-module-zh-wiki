lua_need_request_body
---------------------

**语法:** *lua_need_request_body &lt;on|off&gt;*

**默认:** *off*

**环境:** *http, server, location, location if*

**阶段:** *depends on usage*

在运行 rewrite/access/access_by_lua* 之前决定是否强制获取请求体数据。 Nginx 内部默认不读取客户端请求体，如果需要读取请求体数据，需要使用该指令设置为 `on` 或者在 Lua 代码中调用 [ngx.req.read_body](#ngxreqread_body) 函数。

为了读取请求体数据到[$request_body](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_request_body)变量，[client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size)必须要与[client_max_body_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size)有同样的大小。因为内容大小超过[client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size)但是小于[client_max_body_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size)时， Nginx 将把缓冲内存数据存到一个磁盘的临时文件上，这将导致[$request_body](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_request_body)变量是一个空值。

如果当前location包含 [rewrite_by_lua](#rewrite_by_lua) 或 [rewrite_by_lua_file](#rewrite_by_lua_file) 指令，请求体将在[rewrite_by_lua](#rewrite_by_lua) 或 [rewrite_by_lua_file](#rewrite_by_lua_file)代码运行之前（还是在`rewrite`阶段）被读取。如果只有[content_by_lua](#content_by_lua)指令，请求体直到内容生成的 Lua 代码执行时才会读取（既，请求体在处理生成返回数据阶段才回被读取）。

无论如何都非常推荐，使用[ngx.req.read_body](#ngxreqread_body)和[ngx.req.discard_body](#ngxreqdiscard_body)函数，可以更好的控制请求体的读取过程。

这些也适用于 [access_by_lua](#access_by_lua) 和 [access_by_lua_file](#access_by_lua_file)。

[返回目录](#directives)

> English source:

lua_need_request_body
---------------------

**syntax:** *lua_need_request_body &lt;on|off&gt;*

**default:** *off*

**context:** *http, server, location, location if*

**phase:** *depends on usage*

Determines whether to force the request body data to be read before running rewrite/access/access_by_lua* or not. The Nginx core does not read the client request body by default and if request body data is required, then this directive should be turned `on` or the [ngx.req.read_body](#ngxreqread_body) function should be called within the Lua code.

To read the request body data within the [$request_body](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_request_body) variable,
[client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size) must have the same value as [client_max_body_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size). Because when the content length exceeds [client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size) but less than [client_max_body_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size), Nginx will buffer the data into a temporary file on the disk, which will lead to empty value in the [$request_body](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_request_body) variable.

If the current location includes [rewrite_by_lua](#rewrite_by_lua) or [rewrite_by_lua_file](#rewrite_by_lua_file) directives,
then the request body will be read just before the [rewrite_by_lua](#rewrite_by_lua) or [rewrite_by_lua_file](#rewrite_by_lua_file) code is run (and also at the
`rewrite` phase). Similarly, if only [content_by_lua](#content_by_lua) is specified,
the request body will not be read until the content handler's Lua code is
about to run (i.e., the request body will be read during the content phase).

It is recommended however, to use the [ngx.req.read_body](#ngxreqread_body) and [ngx.req.discard_body](#ngxreqdiscard_body) functions for finer control over the request body reading process instead.

This also applies to [access_by_lua](#access_by_lua) and [access_by_lua_file](#access_by_lua_file).

[Back to TOC](#directives)
