lua_check_client_abort
----------------------

**语法:** *lua_check_client_abort on|off*

**默认:** *lua_check_client_abort off*

**环境:** *http, server, location, location-if*

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
