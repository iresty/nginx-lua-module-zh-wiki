log_by_lua
----------

**语法:** *log_by_lua &lt;lua-script-str&gt;*

**环境:** *http, server, location, location if*

**阶段:** *log*

**注意** 自从 `v0.9.17` 版本, 使用该指令是 *不爽* 的，应使用新的 [log_by_lua_block](#log_by_lua_block) 指令进行替代。

在 `log` 请求处理阶段执行内嵌在`<lua-script-str>`的 Lua 代码。它不替代当前access的日志，而是在其后执行。

注意，当前环境中以下 API 函数当前是被禁用的：

* 输出API函数类（例如：[ngx.say](#ngxsay) 和 [ngx.send_headers](#ngxsend_headers)）
* 控制API函数类（例如：[ngx.exit](#ngxexit) 和 [ngx.exec](#ngxexec)）
* 子请求函数类（例如：[ngx.location.capture](#ngxlocationcapture) 和 [ngx.location.capture_multi](#ngxlocationcapture_multi)）
* cosocket 函数类（例如：[ngx.socket.tcp](#ngxsockettcp) 和 [ngx.req.socket](#ngxreqsocket)）

这是一个收集 [$upstream_response_time](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_response_time) 平均处理的例子：

```nginx

 lua_shared_dict log_dict 5M;

 server {
     location / {
         proxy_pass http://mybackend;

         log_by_lua '
             local log_dict = ngx.shared.log_dict
             local upstream_time = tonumber(ngx.var.upstream_response_time)

             local sum = log_dict:get("upstream_time-sum") or 0
             sum = sum + upstream_time
             log_dict:set("upstream_time-sum", sum)

             local newval, err = log_dict:incr("upstream_time-nb", 1)
             if not newval and err == "not found" then
                 log_dict:add("upstream_time-nb", 0)
                 log_dict:incr("upstream_time-nb", 1)
             end
         ';
     }

     location = /status {
         content_by_lua '
             local log_dict = ngx.shared.log_dict
             local sum = log_dict:get("upstream_time-sum")
             local nb = log_dict:get("upstream_time-nb")

             if nb and sum then
                 ngx.say("average upstream response time: ", sum / nb,
                         " (", nb, " reqs)")
             else
                 ngx.say("no data yet")
             end
         ';
     }
 }
```

该指令在`v0.5.0rc31`版本被首次引入。

[返回目录](#directives)

> English source:

log_by_lua
----------

**syntax:** *log_by_lua &lt;lua-script-str&gt;*

**context:** *http, server, location, location if*

**phase:** *log*

**WARNING** Since the `v0.9.17` release, use of this directive is *discouraged*;
use the new [log_by_lua_block](#log_by_lua_block) directive instead.

Run the Lua source code inlined as the `<lua-script-str>` at the `log` request processing phase. This does not replace the current access logs, but runs after.

Note that the following API functions are currently disabled within this context:

* Output API functions (e.g., [ngx.say](#ngxsay) and [ngx.send_headers](#ngxsend_headers))
* Control API functions (e.g., [ngx.exit](#ngxexit))
* Subrequest API functions (e.g., [ngx.location.capture](#ngxlocationcapture) and [ngx.location.capture_multi](#ngxlocationcapture_multi))
* Cosocket API functions (e.g., [ngx.socket.tcp](#ngxsockettcp) and [ngx.req.socket](#ngxreqsocket)).

Here is an example of gathering average data for [$upstream_response_time](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_response_time):

```nginx

 lua_shared_dict log_dict 5M;

 server {
     location / {
         proxy_pass http://mybackend;

         log_by_lua '
             local log_dict = ngx.shared.log_dict
             local upstream_time = tonumber(ngx.var.upstream_response_time)

             local sum = log_dict:get("upstream_time-sum") or 0
             sum = sum + upstream_time
             log_dict:set("upstream_time-sum", sum)

             local newval, err = log_dict:incr("upstream_time-nb", 1)
             if not newval and err == "not found" then
                 log_dict:add("upstream_time-nb", 0)
                 log_dict:incr("upstream_time-nb", 1)
             end
         ';
     }

     location = /status {
         content_by_lua '
             local log_dict = ngx.shared.log_dict
             local sum = log_dict:get("upstream_time-sum")
             local nb = log_dict:get("upstream_time-nb")

             if nb and sum then
                 ngx.say("average upstream response time: ", sum / nb,
                         " (", nb, " reqs)")
             else
                 ngx.say("no data yet")
             end
         ';
     }
 }
```

This directive was first introduced in the `v0.5.0rc31` release.

[Back to TOC](#directives)