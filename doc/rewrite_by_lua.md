rewrite_by_lua
--------------

**语法:** *rewrite_by_lua &lt;lua-script-str&gt;*

**环境:** *http, server, location, location if*

**阶段:** *rewrite tail*

**注意** 自从 `v0.9.17` 版本, 使用该指令是 *不爽* 的，应使用新的 [rewrite_by_lua_block](#rewrite_by_lua_block) 指令进行替代。

作为一个重写阶段的处理程序，为每个请求执行由`<lua-script-str>`指定的 Lua 代码。

这些 Lua 代码可以调用[全部 API](#nginx-api-for-lua)，并作为一个新的协程，在一个独立的全局环境中执行（就像一个沙盒）。

注意这个处理过程总是在标准[ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)的 *后* 面。所以下面的示例可以按照预期执行：

```nginx

 location /foo {
     set $a 12; # create and initialize $a
     set $b ""; # create and initialize $b
     rewrite_by_lua 'ngx.var.b = tonumber(ngx.var.a) + 1';
     echo "res = $b";
 }
```

因为 `set $a 12` 和 `set $b ""` 的执行都在[rewrite_by_lua](#rewrite_by_lua)的前面。

另一方面，下面的示例是不能按照预期执行：

```nginx

 ?  location /foo {
 ?      set $a 12; # create and initialize $a
 ?      set $b ''; # create and initialize $b
 ?      rewrite_by_lua 'ngx.var.b = tonumber(ngx.var.a) + 1';
 ?      if ($b = '13') {
 ?         rewrite ^ /bar redirect;
 ?         break;
 ?      }
 ?
 ?      echo "res = $b";
 ?  }
```

因为 `if` 是[rewrite_by_lua](#rewrite_by_lua)之 *前* 执行的，尽管在配置中它被放到[rewrite_by_lua](#rewrite_by_lua)后面。

正确的处理方式应该是这样：

```nginx

 location /foo {
     set $a 12; # create and initialize $a
     set $b ''; # create and initialize $b
     rewrite_by_lua '
         ngx.var.b = tonumber(ngx.var.a) + 1
         if tonumber(ngx.var.b) == 13 then
             return ngx.redirect("/bar");
         end
     ';

     echo "res = $b";
 }
```

注意，[ngx_eval](http://www.grid.net.ru/nginx/eval.en.html)模块可以近似的使用[rewrite_by_lua](#rewrite_by_lua)。例如：

```nginx

 location / {
     eval $res {
         proxy_pass http://foo.com/check-spam;
     }

     if ($res = 'spam') {
         rewrite ^ /terms-of-use.html redirect;
     }

     fastcgi_pass ...;
 }
```

在ngx_lua中可以这样实施：

```nginx

 location = /check-spam {
     internal;
     proxy_pass http://foo.com/check-spam;
 }

 location / {
     rewrite_by_lua '
         local res = ngx.location.capture("/check-spam")
         if res.body == "spam" then
             return ngx.redirect("/terms-of-use.html")
         end
     ';

     fastcgi_pass ...;
 }
```

如同其他重写阶段处理， [rewrite_by_lua](#rewrite_by_lua)在子请求中也执行的。

注意，在[rewrite_by_lua](#rewrite_by_lua)处理内部，当调用`ngx.exit(ngx.OK)`时，nginx请求将继续下一阶段的内容处理。要在[rewrite_by_lua](#rewrite_by_lua)处理中终结当前请求，调用[ngx.exit](#ngxexit)，成功的请求设定 status >= 200 (`ngx.HTTP_OK`) 并 status < 300 (`ngx.HTTP_SPECIAL_RESPONSE`)，失败的请求设定`ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR)` (或其他相关的)。

如果使用了[ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)的[rewrite](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite)指令来改变 URI 并发起内部重定向，那么在当前 location 内任何有序的[rewrite_by_lua](#rewrite_by_lua) 或 [rewrite_by_lua_file](#rewrite_by_lua_file)代码都将不被执行。例如：

```nginx

 location /foo {
     rewrite ^ /bar;
     rewrite_by_lua 'ngx.exit(503)';
 }
 location /bar {
     ...
 }
```

<!-- 这里的 Lua 代码 `ngx.exit(503)` 是永远不能执行的。 waiting todo completely -->
Here the Lua code `ngx.exit(503)` will never run. This will be the case if `rewrite ^ /bar last` is used as this will similarly initiate an internal redirection. If the `break` modifier is used instead, there will be no internal redirection and the `rewrite_by_lua` code will be executed.

`rewrite_by_lua`代码将永远在 `rewrite` 请求处理阶段后面，除非[rewrite_by_lua_no_postpone](#rewrite_by_lua_no_postpone)配置开启。

[返回目录](#directives)

> English source:

rewrite_by_lua
--------------

**syntax:** *rewrite_by_lua &lt;lua-script-str&gt;*

**context:** *http, server, location, location if*

**phase:** *rewrite tail*

**WARNING** Since the `v0.9.17` release, use of this directive is *discouraged*;
use the new [rewrite_by_lua_block](#rewrite_by_lua_block) directive instead.

Acts as a rewrite phase handler and executes Lua code string specified in `<lua-script-str>` for every request.
The Lua code may make [API calls](#nginx-api-for-lua) and is executed as a new spawned coroutine in an independent global environment (i.e. a sandbox).

Note that this handler always runs *after* the standard [ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html). So the following will work as expected:

```nginx

 location /foo {
     set $a 12; # create and initialize $a
     set $b ""; # create and initialize $b
     rewrite_by_lua 'ngx.var.b = tonumber(ngx.var.a) + 1';
     echo "res = $b";
 }
```

because `set $a 12` and `set $b ""` run *before* [rewrite_by_lua](#rewrite_by_lua).

On the other hand, the following will not work as expected:

```nginx

 ?  location /foo {
 ?      set $a 12; # create and initialize $a
 ?      set $b ''; # create and initialize $b
 ?      rewrite_by_lua 'ngx.var.b = tonumber(ngx.var.a) + 1';
 ?      if ($b = '13') {
 ?         rewrite ^ /bar redirect;
 ?         break;
 ?      }
 ?
 ?      echo "res = $b";
 ?  }
```

because `if` runs *before* [rewrite_by_lua](#rewrite_by_lua) even if it is placed after [rewrite_by_lua](#rewrite_by_lua) in the config.

The right way of doing this is as follows:

```nginx

 location /foo {
     set $a 12; # create and initialize $a
     set $b ''; # create and initialize $b
     rewrite_by_lua '
         ngx.var.b = tonumber(ngx.var.a) + 1
         if tonumber(ngx.var.b) == 13 then
             return ngx.redirect("/bar");
         end
     ';

     echo "res = $b";
 }
```

Note that the [ngx_eval](http://www.grid.net.ru/nginx/eval.en.html) module can be approximated by using [rewrite_by_lua](#rewrite_by_lua). For example,

```nginx

 location / {
     eval $res {
         proxy_pass http://foo.com/check-spam;
     }

     if ($res = 'spam') {
         rewrite ^ /terms-of-use.html redirect;
     }

     fastcgi_pass ...;
 }
```

can be implemented in ngx_lua as:

```nginx

 location = /check-spam {
     internal;
     proxy_pass http://foo.com/check-spam;
 }

 location / {
     rewrite_by_lua '
         local res = ngx.location.capture("/check-spam")
         if res.body == "spam" then
             return ngx.redirect("/terms-of-use.html")
         end
     ';

     fastcgi_pass ...;
 }
```

Just as any other rewrite phase handlers, [rewrite_by_lua](#rewrite_by_lua) also runs in subrequests.

Note that when calling `ngx.exit(ngx.OK)` within a [rewrite_by_lua](#rewrite_by_lua) handler, the nginx request processing control flow will still continue to the content handler. To terminate the current request from within a [rewrite_by_lua](#rewrite_by_lua) handler, calling [ngx.exit](#ngxexit) with status >= 200 (`ngx.HTTP_OK`) and status < 300 (`ngx.HTTP_SPECIAL_RESPONSE`) for successful quits and `ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR)` (or its friends) for failures.

If the [ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)'s [rewrite](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite) directive is used to change the URI and initiate location re-lookups (internal redirections), then any [rewrite_by_lua](#rewrite_by_lua) or [rewrite_by_lua_file](#rewrite_by_lua_file) code sequences within the current location will not be executed. For example,

```nginx

 location /foo {
     rewrite ^ /bar;
     rewrite_by_lua 'ngx.exit(503)';
 }
 location /bar {
     ...
 }
```

Here the Lua code `ngx.exit(503)` will never run. This will be the case if `rewrite ^ /bar last` is used as this will similarly initiate an internal redirection. If the `break` modifier is used instead, there will be no internal redirection and the `rewrite_by_lua` code will be executed.

The `rewrite_by_lua` code will always run at the end of the `rewrite` request-processing phase unless [rewrite_by_lua_no_postpone](#rewrite_by_lua_no_postpone) is turned on.

[Back to TOC](#directives)
