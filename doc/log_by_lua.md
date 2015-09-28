log_by_lua
----------

**syntax:** *log_by_lua &lt;lua-script-str&gt;*

**context:** *http, server, location, location if*

**phase:** *log*

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

> English source:

log_by_lua
----------

**syntax:** *log_by_lua &lt;lua-script-str&gt;*

**context:** *http, server, location, location if*

**phase:** *log*

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