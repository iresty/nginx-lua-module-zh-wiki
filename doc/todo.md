未完成列表
====

* 对存在`*_by_lua`的小节，添加`*_by_lua_block`支持，这样我们就可以直接把Lua代码放到当前小节中，替代原有的字符串形式。例如：

```nginx

 content_by_lua_block {
     ngx.say("hello, world\r\n")
 }
```

等价于

```nginx

 content_by_lua '
     ngx.say("hello, world\\r\\n")
 ';
```

但是格式更加清晰、友好。

* cosocket：实现LuaSocket非连接的UDP API。
* 实现普通的TCP服务替代HTTP服务，并支持Lua代码。例如：

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

* 实现普通的UDP服务替代HTTP服务，并支持Lua代码。例如：

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
* ssl：实现`ssl_certificate_by_lua` 和 `ssl_certificate_by_lua_file`小节，使用Lua完成下游动态服务SSL证书和密钥的握手（在CloudFlare的私有分之中已经完成，给CloudFlare全球的SSL网络提供强有力支持。预期2015年3月进行开源）。
* shm：实现一个"shared queue API"，对[shared dict](#lua_shared_dict) 补充API。
* cosocket：在[init_by_lua*](#init_by_lua)添加支持。
* cosocket：对于流式cosocket，实现bind()方法。
* cosocket：基于池子的后端并发连接控制，当后端并发超过它的连接池限制，实现自动排队的`connect`。
* cosocket：查看合并aviramc's的[patch](https://github.com/openresty/lua-nginx-module/pull/290)，添加`bsdrecv`方法。
* 添加新的API函数完成标准`add_header`配置功能。
* [ngx.re](#ngxrematch) API: 当没有成功匹配结果时，返回结果使用`false`替换`nil`，这样我们可以避免数组的"holes"问题。
* 查看、合并 Jader H. Silva 的 补丁 `ngx.re.split()` 。
* 查看、合并 Jader H. Silva 的 补丁 `ngx.re.split()`.
* 查看、合并 vadim-pavlov 的 补丁，给[ngx.location.capture](#ngxlocationcapture)添加 `extra_headers` 选项
* 使用 `ngx_hash_t` 去优化内建的 header 查找，涉及[ngx.req.set_header](#ngxreqset_header), [ngx.header.HEADER](#ngxheaderheader) 等。
* cosocket连接池溢出，支持配置选项定义不同策略
* 添加新的小节，当nginx关闭时执行一段代码。
* 添加`ignore_resp_headers`, `ignore_resp_body`, 和 `ignore_resp`选项给[ngx.location.capture](#ngxlocationcapture)、[ngx.location.capture_multi](#ngxlocationcapture_multi)，对于用户提升微小性能。
* add automatic Lua code time slicing support by yielding and resuming the Lua VM actively via Lua's debug hooks.
<!-- todo 没懂，后面看看怎么翻译  -->
* 添加`stat`类似[mod_lua](https://httpd.apache.org/docs/trunk/mod/mod_lua.html)。

[返回目录](#table-of-contents)

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