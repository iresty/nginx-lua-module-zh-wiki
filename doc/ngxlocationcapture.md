ngx.location.capture
--------------------
**语法:** *res = ngx.location.capture(uri, options?)*

**环境:** *rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

向 `uri` 发起一个同步非阻塞 *Nginx 子请求*。

Nginx 子请求是一种非常强有力的方式，它可以发起非阻塞的内部请求来访问目标地址。目标地址可以是配置文件中其他文件目录地址，或 *任何* 其他 nginx C 模块，包括 `ngx_proxy`、`ngx_fastcgi`、`ngx_memc`、`ngx_postgres`、`ngx_drizzle`，甚至 ngx_lua 自身等等 。

需要注意的是，子请求只是模拟 HTTP 接口的形式， *没有* 额外的 HTTP/TCP 流量，也 *没有* IPC (进程间通信) 调用。所有工作在内部高效地在 C 语言级别完成。

子请求与 HTTP 301/302 重定向指令 (通过 [ngx.redirect](#ngxredirect)) 完全不同，也与内部重定向 ((通过 [ngx.exec](#ngxexec)) 完全不同。

在发起子请求前，用户程序需要读取完整的 HTTP 请求体 (通过调用 [ngx.req.read_body](#ngxreqread_body) 或设置 [lua_need_request_body](#lua_need_request_body) 指令为 on).

下面是一个简单例子：

```lua

 res = ngx.location.capture(uri)
```

返回一个包含四个元素的 Lua 表 (`res.status`, `res.header`, `res.body`, 和 `res.truncated`)。

`res.status` (状态) 保存子请求的响应状态码。

`res.header` (头) 用一个标准 Lua 表储子请求响应的所有头信息。如果是“多值”响应头，这些值将使用 Lua (数组) 表顺序存储。例如，如果子请求响应头包含下面的行：

```bash

 Set-Cookie: a=3
 Set-Cookie: foo=bar
 Set-Cookie: baz=blah
```

则 `res.header["Set-Cookie"]` 将存储 Lua 表 `{"a=3", "foo=bar", "baz=blah"}`。

`res.body` (体) 保存子请求的响应体数据，它可能被截断。用户需要检测 `res.truncated` (截断) 布尔值标记来判断 `res.body` 是否包含截断的数据。这种数据截断的原因只可能是因为子请求发生了不可恢复的错误，例如远端在发送响应体时过早中断了连接，或子请求在接收远端响应体时超时。

URI 请求串可以与 URI 本身连在一起，例如，

```lua

 res = ngx.location.capture('/foo/bar?a=3&b=4')
```

因为 Nginx 内核限制，子请求不允许类似 `@foo` 命名地址。请使用标准地址，并设置 `internal` 指令，仅服务内部请求。

可选的选项表可以作为第二个参数传入，支持以下选项：

* `method`
	指定子请求的请求方法, 只接受类似 `ngx.HTTP_POST` 的常量。
* `body`
	指定子请求的请求体 (仅接受字符串值)。
* `args`
	指定子请求的 URI 请求参数 (可以是字符串或者 Lua 表)。
* `ctx`
        指定一个 Lua 表作为子请求的 [ngx.ctx](#ngxctx) 表，可以是当前请求的 [ngx.ctx](#ngxctx) 表。这种方式可以让父请求和子请求共享完全相同的上下文环境。此选项最早出现在版本 `v0.3.1rc25` 中。
* `vars`
	用一个 Lua 表设置子请求中的 Nginx 变量值。此选项最早出现在版本 `v0.3.1rc31` 中。
* `copy_all_vars`
        设置是否复制所有当前请求的 Nginx 变量值到子请求中，修改子请求的 nginx 变量值不影响当前 (父) 请求。此选项最早出现在版本 `v0.3.1rc31` 中。
* `share_all_vars`
	设置是否共享所有当前 (父) 请求的 Nginx 变量值到子请求中，修改子请求的 nginx 变量值将影响当前 (父) 请求。应用此选项将可能导致非常难以发现的错误，这种副作用是非常有害的。所以只有当完全明确知道自己在做什么时才打开此选项。
* `always_forward_body`
        当设置为 true 时，如果没有设置 `body` 选项，当前 (父) 请求的请求体将被转发给子请求。被 [ngx.req.read_body()](#ngxreqread_body) 或 [lua_need_request_body on](#lua_need_request_body) 指令读取的请求体将直接转发给子请求，而不是在创建子请求时再复制整个请求体 (无论此请求体是缓存在内存中还是临时文件中)。默认情况下，此选项值为 `false`，当 `body` 选项没有设置，且当子请求的请求方法是 `PUT` 和 `POST` 时，当前 (父) 请求的请求体才被转发。

例如，发送一个 POST 子请求，可以这样做：

```lua

 res = ngx.location.capture(
     '/foo/bar',
     { method = ngx.HTTP_POST, body = 'hello, world' }
 )
```

除了 POST 的其他 HTTP 请求方法请参考 [HTTP method constants](#http-method-constants)。
`method` 选项默认值是 `ngx.HTTP_GET`。

`args` 选项可以设置附加的 URI 参数，例如：

```lua

 ngx.location.capture('/foo?a=1',
     { args = { b = 3, c = ':' } }
 )
```

等同于

```lua

 ngx.location.capture('/foo?a=1&b=3&c=%3a')
```

也就是说，这个方法将根据 URI 规则转义参数键和值，并将它们拼接在一起组成一个完整的请求串。`args` 选项要求的 Lua 表的格式与 [ngx.encode_args](#ngxencode_args) 方法中使用的完全相同。

`args` 选项也可以直接包含 (转义过的) 请求串：

```lua

 ngx.location.capture('/foo?a=1',
     { args = 'b=3&c=%3a' } }
 )
```

这个例子与上个例子的功能相同。

`share_all_vars` 选项控制是否让当前请求与其子请求共享 nginx 变量。如果设为 `true`，则当前请求与所有子请求共享所有 nginx 变量作用域。因此，在子请求中修改 nginx 变量将影响当前请求。

使用此选项时需要非常小心，因为共享变量作用域容易导致难以预测的副作用。一般来说，推荐使用 `args`、`vars`、或 `copy_all_vars` 来代替。

此选项默认值是 `false`。

```nginx

 location /other {
     set $dog "$dog world";
     echo "$uri dog: $dog";
 }

 location /lua {
     set $dog 'hello';
     content_by_lua '
         res = ngx.location.capture("/other",
             { share_all_vars = true });

         ngx.print(res.body)
         ngx.say(ngx.var.uri, ": ", ngx.var.dog)
     ';
 }
```

请求地址 `/lua` 将输出


    /other dog: hello world
    /lua: hello world


`copy_all_vars` 在子请求被创建时，提供给它一份父请求的 Nginx 变量拷贝。在子请求中对这些变量的修改将不会影响父请求和其他任何共享父请求变量的子请求。

```nginx

 location /other {
     set $dog "$dog world";
     echo "$uri dog: $dog";
 }

 location /lua {
     set $dog 'hello';
     content_by_lua '
         res = ngx.location.capture("/other",
             { copy_all_vars = true });

         ngx.print(res.body)
         ngx.say(ngx.var.uri, ": ", ngx.var.dog)
     ';
 }
```

请求 `GET /lua` 会输出


    /other dog: hello world
    /lua: hello


请注意，当 `share_all_vars` 和 `copy_all_vars` 都被设置为 true 时，`share_all_vars` 优先。

除了上面提到的两个设置方法外，也可以通过配置 `vars` 选项来设置子请求的变量值。在共享或复制父请求的 nginx 变量值后，子请求被设置这些值。和把特定参数通过 URL 参数编码传入子请求，再在 Nginx 配置文件中解码相比，这个方法效率更高。

```nginx

 location /other {
     content_by_lua '
         ngx.say("dog = ", ngx.var.dog)
         ngx.say("cat = ", ngx.var.cat)
     ';
 }

 location /lua {
     set $dog '';
     set $cat '';
     content_by_lua '
         res = ngx.location.capture("/other",
             { vars = { dog = "hello", cat = 32 }});

         ngx.print(res.body)
     ';
 }
```

访问 `/lua` 将输出


    dog = hello
    cat = 32


选项 `ctx` 被用于给子请求设定一个自定义的 Lua 表作为 [ngx.ctx](#ngxctx) 表。

```nginx

 location /sub {
     content_by_lua '
         ngx.ctx.foo = "bar";
     ';
 }
 location /lua {
     content_by_lua '
         local ctx = {}
         res = ngx.location.capture("/sub", { ctx = ctx })

         ngx.say(ctx.foo);
         ngx.say(ngx.ctx.foo);
     ';
 }
```

请求 `GET /lua` 输出


    bar
    nil


也可以通过 `ctx` 选项设置当前 (父) 请求与子请求共享同一个 [ngx.ctx](#ngxctx) 表。

```nginx

 location /sub {
     content_by_lua '
         ngx.ctx.foo = "bar";
     ';
 }
 location /lua {
     content_by_lua '
         res = ngx.location.capture("/sub", { ctx = ngx.ctx })
         ngx.say(ngx.ctx.foo);
     ';
 }
```

请求 `GET /lua` 输出


    bar


请注意，通过 [ngx.location.capture](#ngxlocationcapture) 创建的子请求默认继承当前请求的所有请求头信息，这有可能导致子请求响应中不可预测的副作用。例如，当使用标准的 `ngx_proxy` 模块服务子请求时，如果主请求头中包含 "Accept-Encoding: gzip"，可能导致子请求返回 Lua 代码无法正确处理的 gzip 压缩过的结果。通过设置 [proxy_pass_request_headers](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass_request_headers) 为 `off` ，在子查询地址中忽略原始请求头。

当没有设置 `body` 选项，且 `always_forward_body` 选项为 false (默认值) 时，`POST` 和 `PUT` 子请求将继承父请求的请求体 (如果有的话)。

Nginx 代码中有一个硬编码的数字，来控制每个主请求最多可以有多少个并发的子请求。在旧版 Nginx 中，这个数字是 `50`，自 Nginx `1.1.x` 以后，并发子请求数被提高到了 `200`。当超过这个限制时，`error.log` 文件中将出现下面的信息：


    [error] 13983#0: *1 subrequests cycle while processing "/uri"


如果需要修改这个限制，可以手工在 Nginx 源代码树的 `nginx/src/http/ngx_http_request.h` 文件中修改 `NGX_HTTP_MAX_SUBREQUESTS` 宏定义。

请参考 [subrequest directives of other modules](#locations-configured-by-subrequest-directives-of-other-modules) 了解目标地址的配置限制。



> English Source

**syntax:** *res = ngx.location.capture(uri, options?)*

**context:** *rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

Issue a synchronous but still non-blocking *Nginx Subrequest* using `uri`.

Nginx's subrequests provide a powerful way to make non-blocking internal requests to other locations configured with disk file directory or *any* other nginx C modules like `ngx_proxy`, `ngx_fastcgi`, `ngx_memc`,
`ngx_postgres`, `ngx_drizzle`, and even ngx_lua itself and etc etc etc.

Also note that subrequests just mimic the HTTP interface but there is *no* extra HTTP/TCP traffic *nor* IPC involved. Everything works internally, efficiently, on the C level.

Subrequests are completely different from HTTP 301/302 redirection (via [ngx.redirect](#ngxredirect)) and internal redirection (via [ngx.exec](#ngxexec)).

You should always read the request body (by either calling [ngx.req.read_body](#ngxreqread_body) or configuring [lua_need_request_body](#lua_need_request_body) on) before initiating a subrequest.

Here is a basic example:

```lua

 res = ngx.location.capture(uri)
```

Returns a Lua table with three slots (`res.status`, `res.header`, `res.body`, and `res.truncated`).

`res.status` holds the response status code for the subrequest response.

`res.header` holds all the response headers of the
subrequest and it is a normal Lua table. For multi-value response headers,
the value is a Lua (array) table that holds all the values in the order that
they appear. For instance, if the subrequest response headers contain the following
lines:

```bash

 Set-Cookie: a=3
 Set-Cookie: foo=bar
 Set-Cookie: baz=blah
```

Then `res.header["Set-Cookie"]` will be evaluated to the table value
`{"a=3", "foo=bar", "baz=blah"}`.

`res.body` holds the subrequest's response body data, which might be truncated. You always need to check the `res.truncated` boolean flag to see if `res.body` contains truncated data. The data truncation here can only be caused by those unrecoverable errors in your subrequests like the cases that the remote end aborts the connection prematurely in the middle of the response body data stream or a read timeout happens when your subrequest is receiving the response body data from the remote.

URI query strings can be concatenated to URI itself, for instance,

```lua

 res = ngx.location.capture('/foo/bar?a=3&b=4')
```

Named locations like `@foo` are not allowed due to a limitation in
the nginx core. Use normal locations combined with the `internal` directive to
prepare internal-only locations.

An optional option table can be fed as the second
argument, which supports the options:

* `method`
	specify the subrequest's request method, which only accepts constants like `ngx.HTTP_POST`.
* `body`
	specify the subrequest's request body (string value only).
* `args`
	specify the subrequest's URI query arguments (both string value and Lua tables are accepted)
* `ctx`
	specify a Lua table to be the [ngx.ctx](#ngxctx) table for the subrequest. It can be the current request's [ngx.ctx](#ngxctx) table, which effectively makes the parent and its subrequest to share exactly the same context table. This option was first introduced in the `v0.3.1rc25` release.
* `vars`
	take a Lua table which holds the values to set the specified Nginx variables in the subrequest as this option's value. This option was first introduced in the `v0.3.1rc31` release.
* `copy_all_vars`
	specify whether to copy over all the Nginx variable values of the current request to the subrequest in question. modifications of the nginx variables in the subrequest will not affect the current (parent) request. This option was first introduced in the `v0.3.1rc31` release.
* `share_all_vars`
	specify whether to share all the Nginx variables of the subrequest with the current (parent) request. modifications of the Nginx variables in the subrequest will affect the current (parent) request. Enabling this option may lead to hard-to-debug issues due to bad side-effects and is considered bad and harmful. Only enable this option when you completely know what you are doing.
* `always_forward_body`
	when set to true, the current (parent) request's request body will always be forwarded to the subrequest being created if the `body` option is not specified. The request body read by either [ngx.req.read_body()](#ngxreqread_body) or [lua_need_request_body on](#lua_need_request_body) will be directly forwarded to the subrequest without copying the whole request body data when creating the subrequest (no matter the request body data is buffered in memory buffers or temporary files). By default, this option is `false` and when the `body` option is not specified, the request body of the current (parent) request is only forwarded when the subrequest takes the `PUT` or `POST` request method.

Issuing a POST subrequest, for example, can be done as follows

```lua

 res = ngx.location.capture(
     '/foo/bar',
     { method = ngx.HTTP_POST, body = 'hello, world' }
 )
```

See HTTP method constants methods other than POST.
The `method` option is `ngx.HTTP_GET` by default.

The `args` option can specify extra URI arguments, for instance,

```lua

 ngx.location.capture('/foo?a=1',
     { args = { b = 3, c = ':' } }
 )
```

is equivalent to

```lua

 ngx.location.capture('/foo?a=1&b=3&c=%3a')
```

that is, this method will escape argument keys and values according to URI rules and
concatenate them together into a complete query string. The format for the Lua table passed as the `args` argument is identical to the format used in the [ngx.encode_args](#ngxencode_args) method.

The `args` option can also take plain query strings:

```lua

 ngx.location.capture('/foo?a=1',
     { args = 'b=3&c=%3a' } }
 )
```

This is functionally identical to the previous examples.

The `share_all_vars` option controls whether to share nginx variables among the current request and its subrequests. 
If this option is set to `true`, then the current request and associated subrequests will share the same Nginx variable scope. Hence, changes to Nginx variables made by a subrequest will affect the current request.

Care should be taken in using this option as variable scope sharing can have unexpected side effects. The `args`, `vars`, or `copy_all_vars` options are generally preferable instead.

This option is set to `false` by default

```nginx

 location /other {
     set $dog "$dog world";
     echo "$uri dog: $dog";
 }

 location /lua {
     set $dog 'hello';
     content_by_lua '
         res = ngx.location.capture("/other",
             { share_all_vars = true });

         ngx.print(res.body)
         ngx.say(ngx.var.uri, ": ", ngx.var.dog)
     ';
 }
```

Accessing location `/lua` gives


    /other dog: hello world
    /lua: hello world


The `copy_all_vars` option provides a copy of the parent request's Nginx variables to subrequests when such subrequests are issued. Changes made to these variables by such subrequests will not affect the parent request or any other subrequests sharing the parent request's variables.

```nginx

 location /other {
     set $dog "$dog world";
     echo "$uri dog: $dog";
 }

 location /lua {
     set $dog 'hello';
     content_by_lua '
         res = ngx.location.capture("/other",
             { copy_all_vars = true });

         ngx.print(res.body)
         ngx.say(ngx.var.uri, ": ", ngx.var.dog)
     ';
 }
```

Request `GET /lua` will give the output


    /other dog: hello world
    /lua: hello


Note that if both `share_all_vars` and `copy_all_vars` are set to true, then `share_all_vars` takes precedence.

In addition to the two settings above, it is possible to specify
values for variables in the subrequest using the `vars` option. These
variables are set after the sharing or copying of variables has been
evaluated, and provides a more efficient method of passing specific
values to a subrequest over encoding them as URL arguments and 
unescaping them in the Nginx config file.

```nginx

 location /other {
     content_by_lua '
         ngx.say("dog = ", ngx.var.dog)
         ngx.say("cat = ", ngx.var.cat)
     ';
 }

 location /lua {
     set $dog '';
     set $cat '';
     content_by_lua '
         res = ngx.location.capture("/other",
             { vars = { dog = "hello", cat = 32 }});

         ngx.print(res.body)
     ';
 }
```

Accessing `/lua` will yield the output


    dog = hello
    cat = 32


The `ctx` option can be used to specify a custom Lua table to serve as the [ngx.ctx](#ngxctx) table for the subrequest.

```nginx

 location /sub {
     content_by_lua '
         ngx.ctx.foo = "bar";
     ';
 }
 location /lua {
     content_by_lua '
         local ctx = {}
         res = ngx.location.capture("/sub", { ctx = ctx })

         ngx.say(ctx.foo);
         ngx.say(ngx.ctx.foo);
     ';
 }
```

Then request `GET /lua` gives


    bar
    nil


It is also possible to use this `ctx` option to share the same [ngx.ctx](#ngxctx) table between the current (parent) request and the subrequest:

```nginx

 location /sub {
     content_by_lua '
         ngx.ctx.foo = "bar";
     ';
 }
 location /lua {
     content_by_lua '
         res = ngx.location.capture("/sub", { ctx = ngx.ctx })
         ngx.say(ngx.ctx.foo);
     ';
 }
```

Request `GET /lua` yields the output


    bar


Note that subrequests issued by [ngx.location.capture](#ngxlocationcapture) inherit all the
request headers of the current request by default and that this may have unexpected side effects on the
subrequest responses. For example, when using the standard `ngx_proxy` module to serve
subrequests, an "Accept-Encoding: gzip" header in the main request may result
in gzipped responses that cannot be handled properly in Lua code. Original request headers should be ignored by setting 
[proxy_pass_request_headers](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass_request_headers) to `off` in subrequest locations.

When the `body` option is not specified and the `always_forward_body` option is false (the default value), the `POST` and `PUT` subrequests will inherit the request bodies of the parent request (if any).

There is a hard-coded upper limit on the number of concurrent subrequests possible for every main request. In older versions of Nginx, the limit was `50` concurrent subrequests and in more recent versions, Nginx `1.1.x` onwards, this was increased to `200` concurrent subrequests. When this limit is exceeded, the following error message is added to the `error.log` file:


    [error] 13983#0: *1 subrequests cycle while processing "/uri"


The limit can be manually modified if required by editing the definition of the `NGX_HTTP_MAX_SUBREQUESTS` macro in the `nginx/src/http/ngx_http_request.h` file in the Nginx source tree.

Please also refer to restrictions on capturing locations configured by [subrequest directives of other modules](#locations-configured-by-subrequest-directives-of-other-modules).

[返回目录](#nginx-api-for-lua)