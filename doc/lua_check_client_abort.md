lua_check_client_abort
----------------------

**语法:** *lua_check_client_abort on|off*

**默认:** *lua_check_client_abort off*

**环境:** *http, server, location, location-if*

该指令控制是否探测客户端连接的过早终止。

当启用该指令，ngx_lua模块将会在下游连接上监控连接过早关闭事件。当有这样的事件时，它将调用用户指定 Lua 的回调函数（通过 [ngx.on_abort](#ngxon_abort) 注册），当这里没有用户回调函数注册时，将停止当前请求并清理所有当前请求中运行的 Lua "轻线程" 。

根据目前实现，无论如何，如果请求正在通过 [ngx.req.socket](#ngxreqsocket) 读取请求体，在它之前客户连接发生关闭，ngx_lua 将不会停止任何正在执行的“轻线程”也不会调用用户的回调（尽管已经调用 [ngx.on_abort](#ngxon_abort) ）。作为替代，使用 [ngx.req.socket](#ngxreqsocket) 的读操作第二个参数将直接返回错误信息 “client aborted” 作为返回值（第一个返回值确定是`nil`）。

当 TCP 长连接被禁用，它依靠客户端 socket 关闭的优雅实现（通过发送一个`FIN`包或类似的东西）。

对与（软）实时 Web 应用，强烈推荐使用系统 TCP 协议栈支持的选项对 [TCP keepalive](http://tldp.org/HOWTO/TCP-Keepalive-HOWTO/overview.html) 进行配置，以便及时发现“半开” TCP 连接。

例如，在 Linux，在你的`nginx.conf`文件中你能使用标准 [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) 指令配置，像这样：

```nginx

 listen 80 so_keepalive=2s:2s:8;
```

对于 FreeBSD，你可以只调整 TCP 长连接的系统范围的配置，例如：

    # sysctl net.inet.tcp.keepintvl=2000
    # sysctl net.inet.tcp.keepidle=2000

该指令是在`v0.7.4`版本首次引入的。

也可以看看 [ngx.on_abort](#ngxon_abort) 。

[返回目录](#directives)

> English source:

lua_check_client_abort
----------------------

**syntax:** *lua_check_client_abort on|off*

**default:** *lua_check_client_abort off*

**context:** *http, server, location, location-if*

This directive controls whether to check for premature client connection abortion.

When this directive is turned on, the ngx_lua module will monitor the premature connection close event on the downstream connections. And when there is such an event, it will call the user Lua function callback (registered by [ngx.on_abort](#ngxon_abort)) or just stop and clean up all the Lua "light threads" running in the current request's request handler when there is no user callback function registered.

According to the current implementation, however, if the client closes the connection before the Lua code finishes reading the request body data via [ngx.req.socket](#ngxreqsocket), then ngx_lua will neither stop all the running "light threads" nor call the user callback (if [ngx.on_abort](#ngxon_abort) has been called). Instead, the reading operation on [ngx.req.socket](#ngxreqsocket) will just return the error message "client aborted" as the second return value (the first return value is surely `nil`).

When TCP keepalive is disabled, it is relying on the client side to close the socket gracefully (by sending a `FIN` packet or something like that). For (soft) real-time web applications, it is highly recommended to configure the [TCP keepalive](http://tldp.org/HOWTO/TCP-Keepalive-HOWTO/overview.html) support in your system's TCP stack implementation in order to detect "half-open" TCP connections in time.

For example, on Linux, you can configure the standard [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directive in your `nginx.conf` file like this:

```nginx

 listen 80 so_keepalive=2s:2s:8;
```

On FreeBSD, you can only tune the system-wide configuration for TCP keepalive, for example:

    # sysctl net.inet.tcp.keepintvl=2000
    # sysctl net.inet.tcp.keepidle=2000

This directive was first introduced in the `v0.7.4` release.

See also [ngx.on_abort](#ngxon_abort).

[Back to TOC](#directives)
