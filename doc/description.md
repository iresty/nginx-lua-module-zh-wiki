概述
===========

该模块嵌入 Lua，是通过标准 Lua 解释器或 [LuaJIT 2.0/2.1](http://luajit.org/luajit.html) 植入到 Nginx，并借用 Nginx 的子请求，混合强大的 Lua 线程（Lua协程）到 Nginx 的事件模型。

与 [Apache's mod_lua](https://httpd.apache.org/docs/trunk/mod/mod_lua.html)、[Lighttpd's mod_magnet](http://redmine.lighttpd.net/wiki/1/Docs:ModMagnet) 不太一样, 该模块中的 Lua 代码在网络上是 100% 非阻塞的。包括该模块的 [Nginx API for Lua](#nginx-api-for-lua)，上游请求服务如：MySQL、PostgreSQL、Memcached、Redis或upstream HTTP web 服务等，都是100%非阻塞的。

至少下面这些Lua库、Nginx模块是可以与本ngx_lua模块配合使用的：

* [lua-resty-memcached](https://github.com/openresty/lua-resty-memcached)
* [lua-resty-mysql](https://github.com/openresty/lua-resty-mysql)
* [lua-resty-redis](https://github.com/openresty/lua-resty-redis)
* [lua-resty-dns](https://github.com/openresty/lua-resty-dns)
* [lua-resty-upload](https://github.com/openresty/lua-resty-upload)
* [lua-resty-websocket](https://github.com/openresty/lua-resty-websocket)
* [lua-resty-lock](https://github.com/openresty/lua-resty-lock)
* [lua-resty-string](https://github.com/openresty/lua-resty-string)
* [ngx_memc](http://github.com/openresty/memc-nginx-module)
* [ngx_postgres](https://github.com/FRiCKLE/ngx_postgres)
* [ngx_redis2](http://github.com/openresty/redis2-nginx-module)
* [ngx_redis](http://wiki.nginx.org/HttpRedisModule)
* [ngx_proxy](http://nginx.org/en/docs/http/ngx_http_proxy_module.html)
* [ngx_fastcgi](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html)

几乎所有的Nginx模块都可以通过 [ngx.location.capture](#ngxlocationcapture) 或 [ngx.location.capture_multi](#ngxlocationcapture_multi) 与 ngx_lua 模块完成调用，但这里更推荐使用类似 `lua-resty-*` 的库替代子请求访问Nginx上游模块的方式，因为前者更加灵活且内存利用率更高。

在单个 nginx worker 中，标准 Lua 或 LuaJIT 的实例在所有请求中是共享使用的，但每个请求上下文是通过轻量的Lua协程做到隔离的。

在 nginx worker 进程中加载 Lua 模块，坚持小内存的使用，甚至在重负载下依然如此。

[返回目录](#table-of-contents)

> English Source:

Description
===========

This module embeds Lua, via the standard Lua 5.1 interpreter or [LuaJIT 2.0/2.1](http://luajit.org/luajit.html), into Nginx and by leveraging Nginx's subrequests, allows the integration of the powerful Lua threads (Lua coroutines) into the Nginx event model.

Unlike [Apache's mod_lua](https://httpd.apache.org/docs/trunk/mod/mod_lua.html) and [Lighttpd's mod_magnet](http://redmine.lighttpd.net/wiki/1/Docs:ModMagnet), Lua code executed using this module can be *100% non-blocking* on network traffic as long as the [Nginx API for Lua](#nginx-api-for-lua) provided by this module is used to handle
requests to upstream services such as MySQL, PostgreSQL, Memcached, Redis, or upstream HTTP web services.

At least the following Lua libraries and Nginx modules can be used with this ngx_lua module:

* [lua-resty-memcached](https://github.com/openresty/lua-resty-memcached)
* [lua-resty-mysql](https://github.com/openresty/lua-resty-mysql)
* [lua-resty-redis](https://github.com/openresty/lua-resty-redis)
* [lua-resty-dns](https://github.com/openresty/lua-resty-dns)
* [lua-resty-upload](https://github.com/openresty/lua-resty-upload)
* [lua-resty-websocket](https://github.com/openresty/lua-resty-websocket)
* [lua-resty-lock](https://github.com/openresty/lua-resty-lock)
* [lua-resty-string](https://github.com/openresty/lua-resty-string)
* [ngx_memc](http://github.com/openresty/memc-nginx-module)
* [ngx_postgres](https://github.com/FRiCKLE/ngx_postgres)
* [ngx_redis2](http://github.com/openresty/redis2-nginx-module)
* [ngx_redis](http://wiki.nginx.org/HttpRedisModule)
* [ngx_proxy](http://nginx.org/en/docs/http/ngx_http_proxy_module.html)
* [ngx_fastcgi](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html)

Almost all the Nginx modules can be used with this ngx_lua module by means of [ngx.location.capture](#ngxlocationcapture) or [ngx.location.capture_multi](#ngxlocationcapture_multi) but it is recommended to use those `lua-resty-*` libraries instead of creating subrequests to access the Nginx upstream modules because the former is usually much more flexible and memory-efficient.

The Lua interpreter or LuaJIT instance is shared across all the requests in a single nginx worker process but request contexts are segregated using lightweight Lua coroutines.

Loaded Lua modules persist in the nginx worker process level resulting in a small memory footprint in Lua even when under heavy loads.

[Back to TOC](#table-of-contents)