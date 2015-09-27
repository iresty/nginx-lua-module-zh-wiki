access_by_lua
-------------

**语法:** *access_by_lua &lt;lua-script-str&gt;*

**环境:** *http, server, location, location if*

**阶段:** *access tail*

扮演 access 阶段处理，对每次请求执行在`<lua-script-str>`中指名的 Lua 代码。

这里的 Lua 代码可以使用 [API calls](#nginx-api-for-lua)，它们都是在一个独立的全局环境中新建的子协程中执行的（比如：沙盒）。

注意：本指令的处理总是在标准[ngx_http_access_module](http://nginx.org/en/docs/http/ngx_http_access_module.html)的*后*面。所以下面的示例可以按照预期工作：

```nginx

 location / {
     deny    192.168.1.1;
     allow   192.168.1.0/24;
     allow   10.1.1.0/16;
     deny    all;

     access_by_lua '
         local res = ngx.location.capture("/mysql", { ... })
         ...
     ';

     # proxy_pass/fastcgi_pass/...
 }
```

换句话说，如果一个客户端 IP 地址在黑名单中，它将在 [access_by_lua](#access_by_lua)中的 Mysql 复杂认证请求之前被拒绝。

注意，[ngx_auth_request](http://mdounin.ru/hg/ngx_http_auth_request_module/)模块可以近似的被[access_by_lua](#access_by_lua)实现：

```nginx

 location / {
     auth_request /auth;

     # proxy_pass/fastcgi_pass/postgres_pass/...
 }
```

使用 ngx_lua 是这样：

```nginx

 location / {
     access_by_lua '
         local res = ngx.location.capture("/auth")

         if res.status == ngx.HTTP_OK then
             return
         end

         if res.status == ngx.HTTP_FORBIDDEN then
             ngx.exit(res.status)
         end

         ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR)
     ';

     # proxy_pass/fastcgi_pass/postgres_pass/...
 }
```

和其他access阶段处理实现，[access_by_lua](#access_by_lua)将*不*能运行在子请求中。

注意，在[access_by_lua](#access_by_lua)处理内部，当调用`ngx.exit(ngx.OK)`时，nginx请求将继续下一阶段的内容处理。要在[access_by_lua](#access_by_lua)处理中终结当前请求，调用[ngx.exit](#ngxexit)，成功的请求设定 status >= 200 (`ngx.HTTP_OK`) 并 status < 300 (`ngx.HTTP_SPECIAL_RESPONSE`)，失败的请求设定`ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR)` (或其他相关的)。


[返回目录](#directives)

> English source:

access_by_lua
-------------

**syntax:** *access_by_lua &lt;lua-script-str&gt;*

**context:** *http, server, location, location if*

**phase:** *access tail*

Acts as an access phase handler and executes Lua code string specified in `<lua-script-str>` for every request.
The Lua code may make [API calls](#nginx-api-for-lua) and is executed as a new spawned coroutine in an independent global environment (i.e. a sandbox).

Note that this handler always runs *after* the standard [ngx_http_access_module](http://nginx.org/en/docs/http/ngx_http_access_module.html). So the following will work as expected:

```nginx

 location / {
     deny    192.168.1.1;
     allow   192.168.1.0/24;
     allow   10.1.1.0/16;
     deny    all;

     access_by_lua '
         local res = ngx.location.capture("/mysql", { ... })
         ...
     ';

     # proxy_pass/fastcgi_pass/...
 }
```

That is, if a client IP address is in the blacklist, it will be denied before the MySQL query for more complex authentication is executed by [access_by_lua](#access_by_lua).

Note that the [ngx_auth_request](http://mdounin.ru/hg/ngx_http_auth_request_module/) module can be approximated by using [access_by_lua](#access_by_lua):

```nginx

 location / {
     auth_request /auth;

     # proxy_pass/fastcgi_pass/postgres_pass/...
 }
```

can be implemented in ngx_lua as:

```nginx

 location / {
     access_by_lua '
         local res = ngx.location.capture("/auth")

         if res.status == ngx.HTTP_OK then
             return
         end

         if res.status == ngx.HTTP_FORBIDDEN then
             ngx.exit(res.status)
         end

         ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR)
     ';

     # proxy_pass/fastcgi_pass/postgres_pass/...
 }
```

As with other access phase handlers, [access_by_lua](#access_by_lua) will *not* run in subrequests.

Note that when calling `ngx.exit(ngx.OK)` within a [access_by_lua](#access_by_lua) handler, the nginx request processing control flow will still continue to the content handler. To terminate the current request from within a [access_by_lua](#access_by_lua) handler, calling [ngx.exit](#ngxexit) with status >= 200 (`ngx.HTTP_OK`) and status < 300 (`ngx.HTTP_SPECIAL_RESPONSE`) for successful quits and `ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR)` (or its friends) for failures.

[Back to TOC](#directives)