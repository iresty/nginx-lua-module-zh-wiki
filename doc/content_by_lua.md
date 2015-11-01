content_by_lua
--------------
**语法：** *content_by_lua &lt;lua-script-str&gt;*

**环境：** *location, location if*

**阶段：** *content*

**注意** 自从 `v0.9.17` 版本, 使用该指令是 *不爽* 的，应使用新的 [content_by_lua_block](#content_by_lua_block) 指令进行替代。

作为"内容处理程序"，为每一个请求执行`<lua-script-str>`中指定的Lua代码。

这些 Lua 代码可以调用[全部 API](#nginx-api-for-lua)，并作为一个新的协程，在一个独立的全局环境中执行（就像一个沙盒）。

不要将本指令和其他内容处理程序指令放到同一个location中。
比如，本指令和[proxy_pass](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)指令就不能在同一个location中使用。

[返回目录](#directives)

> English Source

**syntax:** *content_by_lua &lt;lua-script-str&gt;*

**context:** *location, location if*

**phase:** *content*

**WARNING** Since the `v0.9.17` release, use of this directive is *discouraged*;
use the new [content_by_lua_block](#content_by_lua_block) directive instead.

Acts as a "content handler" and executes Lua code string specified in `<lua-script-str>` for every request.
The Lua code may make [API calls](#nginx-api-for-lua) and is executed as a new spawned coroutine in an independent global environment (i.e. a sandbox).

Do not use this directive and other content handler directives in the same location. For example, this directive and the [proxy_pass](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) directive should not be used in the same location.

[Back to TOC](#directives)
