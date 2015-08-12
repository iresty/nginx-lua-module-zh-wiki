content_by_lua
--------------
**语法：** *content_by_lua &lt;lua-script-str&gt;*

**环境：** *location, location if*

**nginx执行阶段：** *content*

充当"内容句柄"，其主要作用是针对每一个请求执行写在`<lua-script-str>`中的lua 代码。这些lua代码可以调用[各种API](#nginx-api-for-lua)，这些lua代码是在一个独立的全局环境中作为新的协程执行的（就是一个沙盒）。

不要讲本指令和其他内容句柄指令放到同一个location中。比如，本指令和[proxy_pass](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)指令就不能在同一个location中使用。

[返回目录](#directives)
> English Source


**syntax:** *content_by_lua &lt;lua-script-str&gt;*

**context:** *location, location if*

**phase:** *content*

Acts as a "content handler" and executes Lua code string specified in `<lua-script-str>` for every request. 
The Lua code may make [API calls](#nginx-api-for-lua) and is executed as a new spawned coroutine in an independent global environment (i.e. a sandbox).

Do not use this directive and other content handler directives in the same location. For example, this directive and the [proxy_pass](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) directive should not be used in the same location.

[Back to TOC](#directives)
