balancer_by_lua_block
---------------------

**语法:** *balancer_by_lua_block { lua-script }*

**环境:** *upstream*

**阶段:** *content*

该指令执行上游的负载均衡 Lua 代码（任何上游实体），代码配置在 `upstream {}` 小节中。

举例：

```nginx

 upstream foo {
     server 127.0.0.1;
     balancer_by_lua_block {
         -- 使用 Lua 作为一个动态均衡器完成一些有趣的事情
     }
 }

 server {
     location / {
         proxy_pass http://foo;
     }
 }
```

Lua 的负载均衡可以和任何已经存在的 nginx 上游模块一起工作，例如：[ngx_proxy](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) 和
[ngx_fastcgi](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html)。

同样， Lua 负载均衡可以和标准的上游连接池机制一起工作，例如标准的 [keepalive](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive) 指令。只要确保在单个`upstream {}` 配置小节中 [keepalive](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive) 指令要放在 `balancer_by_lua_block` 小节的 *后* 面。

Lua 负载均衡能完全忽略配置在 `upstream {}` 小节中定义的服务列表，并且从一个完全动态的服务列表中挑选一个节点（甚至每次请求都在变），所有这些均是通过 [lua-resty-core](https://github.com/openresty/lua-resty-core) 库的 [ngx.balancer](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/balancer.md) 完成。

该指令配置的 Lua 代码在单个下游请求中可能被调用多次，例如使用 [proxy_next_upstream](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_next_upstream) 配置小节，这是 nginx 自身上游尝试请求机制。

这里 Lua 代码的执行环境不支持 yield 操作，所以可能 yield 的 Lua API （例如 cosockets 和 “轻线程”），在这个环境中是被禁用的。一个可以使用并绕过这个限制的玩法，是可以在更早的阶段处理，比如 [access_by_lua*](#access_by_lua))，并传递结果到环境的 [ngx.ctx](#ngxctx) 表。

该指令在 `v0.10.0` 版本首次引入。

[返回目录](#directives)

> English source:

balancer_by_lua_block
---------------------

**syntax:** *balancer_by_lua_block { lua-script }*

**context:** *upstream*

**phase:** *content*

This directive runs Lua code as an upstream balancer for any upstream entities defined
by the `upstream {}` configuration block.

For instance,

```nginx

 upstream foo {
     server 127.0.0.1;
     balancer_by_lua_block {
         -- use Lua to do something interesting here
         -- as a dynamic balancer
     }
 }

 server {
     location / {
         proxy_pass http://foo;
     }
 }
```

The resulting Lua load balancer can work with any existing nginx upstream modules
like [ngx_proxy](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) and
[ngx_fastcgi](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html).

Also, the Lua load balancer can work with the standard upstream connection pool mechanism,
i.e., the standard [keepalive](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive) directive.
Just ensure that the [keepalive](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive) directive
is used *after* this `balancer_by_lua_block` directive in a single `upstream {}` configuration block.

The Lua load balancer can totally ignore the list of servers defined in the `upstream {}` block
and select peer from a completely dynamic server list (even changing per request) via the
[ngx.balancer](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/balancer.md) module
from the [lua-resty-core](https://github.com/openresty/lua-resty-core) library.

The Lua code handler registered by this directive might get called more than once in a single
downstream request when the nginx upstream mechanism retries the request on conditions
specified by directives like the [proxy_next_upstream](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_next_upstream)
directive.

This Lua code execution context does not support yielding, so Lua APIs that may yield
(like cosockets and "light threads") are disabled in this context. One can usually work
around this limitation by doing such operations in an earlier phase handler (like
[access_by_lua*](#access_by_lua)) and passing along the result into this context
via the [ngx.ctx](#ngxctx) table.

This directive was first introduced in the `v0.10.0` release.

[Back to TOC](#directives)