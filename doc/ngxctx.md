ngx.ctx
-------
**环境:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;, ngx.timer.&#42;, balancer_by_lua&#42;*

本 Lua 表可以用来存储基于请求的 Lua 环境数据，其生存周期与当前请求相同 (类似 Nginx 变量)。

参考下面例子，

```nginx

 location /test {
     rewrite_by_lua '
         ngx.ctx.foo = 76
     ';
     access_by_lua '
         ngx.ctx.foo = ngx.ctx.foo + 3
     ';
     content_by_lua '
         ngx.say(ngx.ctx.foo)
     ';
 }
```

访问 `GET /test` 输出

```bash

 79
```

也就是说，`ngx.ctx.foo` 条目跨越一个请求的 rewrite (重写)，access (访问)，和 content (内容) 各处理阶段保持一致。

每个请求，包括子请求，都有一份自己的 `ngx.ctx` 表。例如：

```nginx

 location /sub {
     content_by_lua '
         ngx.say("sub pre: ", ngx.ctx.blah)
         ngx.ctx.blah = 32
         ngx.say("sub post: ", ngx.ctx.blah)
     ';
 }

 location /main {
     content_by_lua '
         ngx.ctx.blah = 73
         ngx.say("main pre: ", ngx.ctx.blah)
         local res = ngx.location.capture("/sub")
         ngx.print(res.body)
         ngx.say("main post: ", ngx.ctx.blah)
     ';
 }
```

访问 `GET /main` 输出

```bash

 main pre: 73
 sub pre: nil
 sub post: 32
 main post: 73
```

这里，在子请求中修改 `ngx.ctx.blah` 条目并不影响父请求中的同名条目，因为它们各自维护不同版本的 `ngx.ctx.blah`。

内部重定向将摧毁原始请求中的 `ngx.ctx` 数据 (如果有)，新请求将会有一个空白的 `ngx.ctx` 表。例如，

```nginx

 location /new {
     content_by_lua '
         ngx.say(ngx.ctx.foo)
     ';
 }

 location /orig {
     content_by_lua '
         ngx.ctx.foo = "hello"
         ngx.exec("/new")
     ';
 }
```

访问 `GET /orig` 将输出

```bash

 nil
```

而不是原始的 `"hello"` 值。

任意数据值，包括 Lua 闭包与嵌套表，都可以被插入这个“魔法”表，也允许注册自定义元方法。

也可以将 `ngx.ctx` 覆盖为一个新 Lua 表，例如，

```lua

 ngx.ctx = { foo = 32, bar = 54 }
```

当用在 [init_worker_by_lua*](#init_worker_by_lua) 环境中，这个表与当前 Lua 句柄生命周期相同。

The `ngx.ctx` lookup requires relatively expensive metamethod calls and it is much slower than explicitly passing per-request data along by your own function arguments. So do not abuse this API for saving your own function arguments because it usually has quite some performance impact.

`ngx.ctx` 表查询需要相对昂贵的元方法调用，这比通过用户自己的函数参数直接传递基于请求的数据要慢得多。所以不要为了节约用户函数参数而滥用此 API，因为它可能对性能有明显影响。

Because of the metamethod magic, never "local" the `ngx.ctx` table outside your Lua function scope on the Lua module level level due to [worker-level data sharing](#data-sharing-within-an-nginx-worker). For example, the following is bad:

而且由于元方法“魔法”，不要在 lua 模块级别试图使用 "local" 级别的 `ngx.ctx` ，例如 [worker-level data sharing](#data-sharing-within-an-nginx-worker)。下面示例是糟糕的：

```lua

 -- mymodule.lua
 local _M = {}

 -- 下面一行的 ngx.ctx 是属于单个请求的，但 `ctx` 变量是在 Lua 模块级别
 -- 并且属于单个 worker 的。
 local ctx = ngx.ctx

 function _M.main()
     ctx.foo = "bar"
 end

 return _M
```

应使用下面方式替代：

```lua

 -- mymodule.lua
 local _M = {}

 function _M.main(ctx)
     ctx.foo = "bar"
 end

 return _M
```

就是说，调用者对 `ctx` 表调用应通过函数传参方式完成。

> English Source

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;, ngx.timer.&#42;, balancer_by_lua&#42;*

This table can be used to store per-request Lua context data and has a life time identical to the current request (as with the Nginx variables). 

Consider the following example,

```nginx

 location /test {
     rewrite_by_lua '
         ngx.ctx.foo = 76
     ';
     access_by_lua '
         ngx.ctx.foo = ngx.ctx.foo + 3
     ';
     content_by_lua '
         ngx.say(ngx.ctx.foo)
     ';
 }
```

Then `GET /test` will yield the output

```bash

 79
```

That is, the `ngx.ctx.foo` entry persists across the rewrite, access, and content phases of a request.

Every request, including subrequests, has its own copy of the table. For example:

```nginx

 location /sub {
     content_by_lua '
         ngx.say("sub pre: ", ngx.ctx.blah)
         ngx.ctx.blah = 32
         ngx.say("sub post: ", ngx.ctx.blah)
     ';
 }

 location /main {
     content_by_lua '
         ngx.ctx.blah = 73
         ngx.say("main pre: ", ngx.ctx.blah)
         local res = ngx.location.capture("/sub")
         ngx.print(res.body)
         ngx.say("main post: ", ngx.ctx.blah)
     ';
 }
```

Then `GET /main` will give the output

```bash

 main pre: 73
 sub pre: nil
 sub post: 32
 main post: 73
```

Here, modification of the `ngx.ctx.blah` entry in the subrequest does not affect the one in the parent request. This is because they have two separate versions of `ngx.ctx.blah`.

Internal redirection will destroy the original request `ngx.ctx` data (if any) and the new request will have an empty `ngx.ctx` table. For instance,

```nginx

 location /new {
     content_by_lua '
         ngx.say(ngx.ctx.foo)
     ';
 }

 location /orig {
     content_by_lua '
         ngx.ctx.foo = "hello"
         ngx.exec("/new")
     ';
 }
```

Then `GET /orig` will give

```bash

 nil
```

rather than the original `"hello"` value.

Arbitrary data values, including Lua closures and nested tables, can be inserted into this "magic" table. It also allows the registration of custom meta methods.

Overriding `ngx.ctx` with a new Lua table is also supported, for example,

```lua

 ngx.ctx = { foo = 32, bar = 54 }
```

When being used in the context of [init_worker_by_lua*](#init_worker_by_lua), this table just has the same lifetime of the current Lua handler.

The `ngx.ctx` lookup requires relatively expensive metamethod calls and it is much slower than explicitly passing per-request data along by your own function arguments. So do not abuse this API for saving your own function arguments because it usually has quite some performance impact.

Because of the metamethod magic, never "local" the `ngx.ctx` table outside your Lua function scope on the Lua module level level due to [worker-level data sharing](#data-sharing-within-an-nginx-worker). For example, the following is bad:

```lua

 -- mymodule.lua
 local _M = {}

 -- the following line is bad since ngx.ctx is a per-request
 -- data while this `ctx` variable is on the Lua module level
 -- and thus is per-nginx-worker.
 local ctx = ngx.ctx

 function _M.main()
     ctx.foo = "bar"
 end

 return _M
```

Use the following instead:

```lua

 -- mymodule.lua
 local _M = {}

 function _M.main(ctx)
     ctx.foo = "bar"
 end

 return _M
```

That is, let the caller pass the `ctx` table explicitly via a function argument.

[返回目录](#nginx-api-for-lua)