lua_http10_buffering
--------------------

**语法:** *lua_http10_buffering on|off*

**默认:** *lua_http10_buffering on*

**环境:** *http, server, location, location-if*

对 HTTP 1.0（或更老）请求，启用或禁用自动应答缓冲区。这个缓冲机制主要用于应答头包含合适`Content-Length`长度的 HTTP 1.0 长连接。

如果 Lua 代码在发送应答头之前明确设置了应答头的`Content-Length`（调用 [ngx.send_headers](#ngxsend_headers) 或 隐式首次调用 [ngx.say](#ngxsay) 或 [ngx.print](#ngxprint) 其中任何一个），HTTP 1.0应答缓冲区都将被禁用，即使这个指令是打开的。

流式输出（例如，调用 [ngx.flush](#ngxflush)）非常大的应答体，为了占用内存最小，该指令必须设置为`off`。

该指令默认值是`on`。

该指令是在`v0.5.0rc19`版本首次引入的。

[返回目录](#directives)

> English source:

lua_http10_buffering
--------------------

**syntax:** *lua_http10_buffering on|off*

**default:** *lua_http10_buffering on*

**context:** *http, server, location, location-if*

Enables or disables automatic response buffering for HTTP 1.0 (or older) requests. This buffering mechanism is mainly used for HTTP 1.0 keep-alive which replies on a proper `Content-Length` response header.

If the Lua code explicitly sets a `Content-Length` response header before sending the headers (either explicitly via [ngx.send_headers](#ngxsend_headers) or implicitly via the first [ngx.say](#ngxsay) or [ngx.print](#ngxprint) call), then the HTTP 1.0 response buffering will be disabled even when this directive is turned on.

To output very large response data in a streaming fashion (via the [ngx.flush](#ngxflush) call, for example), this directive MUST be turned off to minimize memory usage.

This directive is turned `on` by default.

This directive was first introduced in the `v0.5.0rc19` release.

[Back to TOC](#directives)
