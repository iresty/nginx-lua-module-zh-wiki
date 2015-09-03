TODO
====

* add `*_by_lua_block` directives for existing `*_by_lua` directives so that we put literal Lua code directly in curly braces instead of an nginx literal string. For example,
```nginx

 content_by_lua_block {
     ngx.say("hello, world\r\n")
 }
```
    which is equivalent to
```nginx

 content_by_lua '
     ngx.say("hello, world\\r\\n")
 ';
```
    but the former is much cleaner and nicer.
* cosocket: implement LuaSocket's unconnected UDP API.
* add support for implementing general TCP servers instead of HTTP servers in Lua. For example,
```lua

 tcp {
     server {
         listen 11212;
         handler_by_lua '
             -- custom Lua code implementing the special TCP server...
         ';
     }
 }
```
* add support for implementing general UDP servers instead of HTTP servers in Lua. For example,
```lua

 udp {
     server {
         listen 1953;
         handler_by_lua '
             -- custom Lua code implementing the special UDP server...
         ';
     }
 }
```
* ssl: implement directives `ssl_certificate_by_lua` and `ssl_certificate_by_lua_file` to allow using Lua to dynamically serve SSL certificates and keys for downstream SSL handshake. (already done in CloudFlare's private branch and powering CloudFlare's SSL gateway of its global network. expected to be opensourced in March 2015.)
* shm: implement a "shared queue API" to complement the existing [shared dict](#lua_shared_dict) API.
* cosocket: add support in the context of [init_by_lua*](#init_by_lua).
* cosocket: implement the `bind()` method for stream-typed cosockets.
* cosocket: pool-based backend concurrency level control: implement automatic `connect` queueing when the backend concurrency exceeds its connection pool limit.
* cosocket: review and merge aviramc's [patch](https://github.com/openresty/lua-nginx-module/pull/290) for adding the `bsdrecv` method.
* add new API function `ngx.resp.add_header` to emulate the standard `add_header` config directive.
* [ngx.re](#ngxrematch) API: use `false` instead of `nil` in the resulting match table to indicate non-existent submatch captures, such that we can avoid "holes" in the array table.
* review and apply Jader H. Silva's patch for `ngx.re.split()`.
* review and apply vadim-pavlov's patch for [ngx.location.capture](#ngxlocationcapture)'s `extra_headers` option
* use `ngx_hash_t` to optimize the built-in header look-up process for [ngx.req.set_header](#ngxreqset_header), [ngx.header.HEADER](#ngxheaderheader), and etc.
* add configure options for different strategies of handling the cosocket connection exceeding in the pools.
* add directives to run Lua codes when nginx stops.
* add `ignore_resp_headers`, `ignore_resp_body`, and `ignore_resp` options to [ngx.location.capture](#ngxlocationcapture) and [ngx.location.capture_multi](#ngxlocationcapture_multi) methods, to allow micro performance tuning on the user side.
* add automatic Lua code time slicing support by yielding and resuming the Lua VM actively via Lua's debug hooks.
* add `stat` mode similar to [mod_lua](https://httpd.apache.org/docs/trunk/mod/mod_lua.html).

[Back to TOC](#table-of-contents)

> English source:

TODO
====

* add `*_by_lua_block` directives for existing `*_by_lua` directives so that we put literal Lua code directly in curly braces instead of an nginx literal string. For example,
```nginx

 content_by_lua_block {
     ngx.say("hello, world\r\n")
 }
```
    which is equivalent to
```nginx

 content_by_lua '
     ngx.say("hello, world\\r\\n")
 ';
```
    but the former is much cleaner and nicer.
* cosocket: implement LuaSocket's unconnected UDP API.
* add support for implementing general TCP servers instead of HTTP servers in Lua. For example,
```lua

 tcp {
     server {
         listen 11212;
         handler_by_lua '
             -- custom Lua code implementing the special TCP server...
         ';
     }
 }
```
* add support for implementing general UDP servers instead of HTTP servers in Lua. For example,
```lua

 udp {
     server {
         listen 1953;
         handler_by_lua '
             -- custom Lua code implementing the special UDP server...
         ';
     }
 }
```
* ssl: implement directives `ssl_certificate_by_lua` and `ssl_certificate_by_lua_file` to allow using Lua to dynamically serve SSL certificates and keys for downstream SSL handshake. (already done in CloudFlare's private branch and powering CloudFlare's SSL gateway of its global network. expected to be opensourced in March 2015.)
* shm: implement a "shared queue API" to complement the existing [shared dict](#lua_shared_dict) API.
* cosocket: add support in the context of [init_by_lua*](#init_by_lua).
* cosocket: implement the `bind()` method for stream-typed cosockets.
* cosocket: pool-based backend concurrency level control: implement automatic `connect` queueing when the backend concurrency exceeds its connection pool limit.
* cosocket: review and merge aviramc's [patch](https://github.com/openresty/lua-nginx-module/pull/290) for adding the `bsdrecv` method.
* add new API function `ngx.resp.add_header` to emulate the standard `add_header` config directive.
* [ngx.re](#ngxrematch) API: use `false` instead of `nil` in the resulting match table to indicate non-existent submatch captures, such that we can avoid "holes" in the array table.
* review and apply Jader H. Silva's patch for `ngx.re.split()`.
* review and apply vadim-pavlov's patch for [ngx.location.capture](#ngxlocationcapture)'s `extra_headers` option
* use `ngx_hash_t` to optimize the built-in header look-up process for [ngx.req.set_header](#ngxreqset_header), [ngx.header.HEADER](#ngxheaderheader), and etc.
* add configure options for different strategies of handling the cosocket connection exceeding in the pools.
* add directives to run Lua codes when nginx stops.
* add `ignore_resp_headers`, `ignore_resp_body`, and `ignore_resp` options to [ngx.location.capture](#ngxlocationcapture) and [ngx.location.capture_multi](#ngxlocationcapture_multi) methods, to allow micro performance tuning on the user side.
* add automatic Lua code time slicing support by yielding and resuming the Lua VM actively via Lua's debug hooks.
* add `stat` mode similar to [mod_lua](https://httpd.apache.org/docs/trunk/mod/mod_lua.html).

[Back to TOC](#table-of-contents)