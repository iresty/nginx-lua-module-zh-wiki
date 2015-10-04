ngx.thread.spawn
----------------
**语法:** *co = ngx.thread.spawn(func, arg1, arg2, ...)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

使用 Lua 函数 `func` 以及其他可选参数 `arg1`、`arg2` 等， 产生一个新的用户 "轻线程" 。 返回一个 Lua 线程（或者说是 Lua 协程）对象，这里称之为“轻线程”。

“轻线程”是特殊类型的 Lua 协程，它是由 ngx_lua 模块调度的。

在 `ngx.thread.spawn` 返回之前，`func` 将被其他可选参数调用直到它返回，例如错误的终止或调用 [Nginx API for Lua](#nginx-api-for-lua) (比如 [tcpsock:receive](#tcpsockreceive)) 引起了 I/O 操作被挂起。

`ngx.thread.spawn` 返回后，新创建的“轻线程”将开始异步方式在各个 I/O 事件上执行。

在 [rewrite_by_lua](#rewrite_by_lua)、[access_by_lua](#access_by_lua) 中的 Lua 代码块是在 ngx_lua 自动创建的“轻线程”样板执行的。这类样板的“轻线程”也被称为“进入线程”。

默认的，相应的 Nginx 处理部分（例如 [rewrite_by_lua](#rewrite_by_lua) 部分）将不会终止，直到：
By default, the corresponding Nginx handler (e.g., [rewrite_by_lua](#rewrite_by_lua) handler) will not terminate until

1. “进入线程”和用户所有的“轻线程”都终止了
1. 一个“轻线程”（无论“进入线程”或用户“轻线程”），由于调用 [ngx.exit](#ngxexit)、 [ngx.exec](#ngxexec)、 [ngx.redirect](#ngxredirect)、 [ngx.req.set_uri(uri, true)](#ngxreqset_uri) 中止执行
1. “进入线程” 因为 Lua 错误的终止

当用户"轻线程"因为 Lua 错误而中止，无论如何，它将不会中止其他运行的“轻线程”，就像“进入线程”一样。

归咎于 Nginx 子请求模型的限制，一般来说终止一个正在运行的 Nginx 子请求是不被允许的。挂起在一个或多个 Nginx 子请求结果的“轻线程”，也是禁止终止的。你必须调用 [ngx.thread.wait](#ngxthreadwait) 等待这些“轻线程”退出这个“世界”。这里有个非正式异常办法，你可以调用 [ngx.exit](#ngxexit) 完成挂起子请求的终止，可以使用的状态码有：`ngx.ERROR` (-1)， `408`， `444`， 或 `499`。

这些“轻线程”不会以抢占方式调度。换句话说，不会有自动执行的时间分片。“轻线程”将继续保持运行状态，只有当 CPU 出现下面情况：

1. 一个 I/O 操作（异步）不能在独立操作执行里完成
1. 调用 [coroutine.yield](#coroutineyield) 声明放弃执行
1. 因为 Lua 错误或 [ngx.exit](#ngxexit)、 [ngx.exec](#ngxexec)、 [ngx.redirect](#ngxredirect)、 或 [ngx.req.set_uri(uri, true)](#ngxreqset_uri) 导致的终止

对于前两个示例，“轻线程”将通常稍候被 ngx_lua 调度唤醒，除非发生 “全局停止” 事件。

用户“轻线程”可以创建它们自己的“轻线程”。并且由 [coroutine.create](#coroutinecreate) 创建的普通用户协程 也能 “轻线程”。直接产生“轻线程”的协程（可以是普通 Lua 协程或“轻线程”）被称为“父协程”，新产生的是“轻线程”。

“父协程”可以调用 [ngx.thread.wait](#ngxthreadwait) 等待子“轻线程”的终止。

你可以在“轻线程”协程中调用  coroutine.status() 和 coroutine.yield() 。

下面情况,“轻线程”协程可能是“僵尸”状态:

1. 当前“轻线程”已经终止（无论成功还是错误地）
1. 它的父协程还存活，但是他的父协程已经不再使用 [ngx.thread.wait](#ngxthreadwait) 进行等待

下面的例子说明在“轻线程”协程中通过 coroutine.yield() 手工完成时间-切片：

```lua

 local yield = coroutine.yield

 function f()
     local self = coroutine.running()
     ngx.say("f 1")
     yield(self)
     ngx.say("f 2")
     yield(self)
     ngx.say("f 3")
 end

 local self = coroutine.running()
 ngx.say("0")
 yield(self)

 ngx.say("1")
 ngx.thread.spawn(f)

 ngx.say("2")
 yield(self)

 ngx.say("3")
 yield(self)

 ngx.say("4")
```

然后它将生成下面的输出：


    0
    1
    f 1
    2
    f 2
    3
    f 3
    4


 在一个独立的 Nginx 请求中完成上游请求的并发执行，“轻线程” 是非常有用，有点像 [ngx.location.capture_multi](#ngxlocationcapture_multi) 的一个普通版本，而且它能使用所有 [Nginx Lua 的 API](#nginx-api-for-lua) 。下面的例子说明在一个独立的 Nginx 请求中并行请求到 MySQL、Memcached 和上游 HTTP 服务，并以他们实际返回结果的顺序进行输出（与 Facebook BigPipe 模型非常相像）：

```lua

 -- 同时查询 mysql、 memcached 和一个远程 HTTP 服务
 -- 以它们实际返回结果的顺序进行输出

 local mysql = require "resty.mysql"
 local memcached = require "resty.memcached"

 local function query_mysql()
     local db = mysql:new()
     db:connect{
                 host = "127.0.0.1",
                 port = 3306,
                 database = "test",
                 user = "monty",
                 password = "mypass"
               }
     local res, err, errno, sqlstate =
             db:query("select * from cats order by id asc")
     db:set_keepalive(0, 100)
     ngx.say("mysql done: ", cjson.encode(res))
 end

 local function query_memcached()
     local memc = memcached:new()
     memc:connect("127.0.0.1", 11211)
     local res, err = memc:get("some_key")
     ngx.say("memcached done: ", res)
 end

 local function query_http()
     local res = ngx.location.capture("/my-http-proxy")
     ngx.say("http done: ", res.body)
 end

 ngx.thread.spawn(query_mysql)      -- 创建线程 1
 ngx.thread.spawn(query_memcached)  -- 创建线程 2
 ngx.thread.spawn(query_http)       -- 创建线程 3
```

该 API 是从 `v0.7.0` 版本首次引入。

<!-- todo 有点难度的翻译，大家帮忙仔细滤滤 -->

[返回目录](#nginx-api-for-lua)

> English source:

ngx.thread.spawn
----------------
**syntax:** *co = ngx.thread.spawn(func, arg1, arg2, ...)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Spawns a new user "light thread" with the Lua function `func` as well as those optional arguments `arg1`, `arg2`, and etc. Returns a Lua thread (or Lua coroutine) object represents this "light thread".

"Light threads" are just a special kind of Lua coroutines that are scheduled by the ngx_lua module.

Before `ngx.thread.spawn` returns, the `func` will be called with those optional arguments until it returns, aborts with an error, or gets yielded due to I/O operations via the [Nginx API for Lua](#nginx-api-for-lua) (like [tcpsock:receive](#tcpsockreceive)).

After `ngx.thread.spawn` returns, the newly-created "light thread" will keep running asynchronously usually at various I/O events.

All the Lua code chunks running by [rewrite_by_lua](#rewrite_by_lua), [access_by_lua](#access_by_lua), and [content_by_lua](#content_by_lua) are in a boilerplate "light thread" created automatically by ngx_lua. Such boilerplate "light thread" are also called "entry threads".

By default, the corresponding Nginx handler (e.g., [rewrite_by_lua](#rewrite_by_lua) handler) will not terminate until

1. both the "entry thread" and all the user "light threads" terminates,
1. a "light thread" (either the "entry thread" or a user "light thread" aborts by calling [ngx.exit](#ngxexit), [ngx.exec](#ngxexec), [ngx.redirect](#ngxredirect), or [ngx.req.set_uri(uri, true)](#ngxreqset_uri), or
1. the "entry thread" terminates with a Lua error.

When the user "light thread" terminates with a Lua error, however, it will not abort other running "light threads" like the "entry thread" does.

Due to the limitation in the Nginx subrequest model, it is not allowed to abort a running Nginx subrequest in general. So it is also prohibited to abort a running "light thread" that is pending on one ore more Nginx subrequests. You must call [ngx.thread.wait](#ngxthreadwait) to wait for those "light thread" to terminate before quitting the "world". A notable exception here is that you can abort pending subrequests by calling [ngx.exit](#ngxexit) with and only with the status code `ngx.ERROR` (-1), `408`, `444`, or `499`.

The "light threads" are not scheduled in a pre-emptive way. In other words, no time-slicing is performed automatically. A "light thread" will keep running exclusively on the CPU until

1. a (nonblocking) I/O operation cannot be completed in a single run,
1. it calls [coroutine.yield](#coroutineyield) to actively give up execution, or
1. it is aborted by a Lua error or an invocation of [ngx.exit](#ngxexit), [ngx.exec](#ngxexec), [ngx.redirect](#ngxredirect), or [ngx.req.set_uri(uri, true)](#ngxreqset_uri).

For the first two cases, the "light thread" will usually be resumed later by the ngx_lua scheduler unless a "stop-the-world" event happens.

User "light threads" can create "light threads" themselves. And normal user coroutines created by [coroutine.create](#coroutinecreate) can also create "light threads". The coroutine (be it a normal Lua coroutine or a "light thread") that directly spawns the "light thread" is called the "parent coroutine" for the "light thread" newly spawned.

The "parent coroutine" can call [ngx.thread.wait](#ngxthreadwait) to wait on the termination of its child "light thread".

You can call coroutine.status() and coroutine.yield() on the "light thread" coroutines.

The status of the "light thread" coroutine can be "zombie" if

1. the current "light thread" already terminates (either successfully or with an error),
1. its parent coroutine is still alive, and
1. its parent coroutine is not waiting on it with [ngx.thread.wait](#ngxthreadwait).

The following example demonstrates the use of coroutine.yield() in the "light thread" coroutines
to do manual time-slicing:

```lua

 local yield = coroutine.yield

 function f()
     local self = coroutine.running()
     ngx.say("f 1")
     yield(self)
     ngx.say("f 2")
     yield(self)
     ngx.say("f 3")
 end

 local self = coroutine.running()
 ngx.say("0")
 yield(self)

 ngx.say("1")
 ngx.thread.spawn(f)

 ngx.say("2")
 yield(self)

 ngx.say("3")
 yield(self)

 ngx.say("4")
```

Then it will generate the output


    0
    1
    f 1
    2
    f 2
    3
    f 3
    4


"Light threads" are mostly useful for doing concurrent upstream requests in a single Nginx request handler, kinda like a generalized version of [ngx.location.capture_multi](#ngxlocationcapture_multi) that can work with all the [Nginx API for Lua](#nginx-api-for-lua). The following example demonstrates parallel requests to MySQL, Memcached, and upstream HTTP services in a single Lua handler, and outputting the results in the order that they actually return (very much like the Facebook BigPipe model):

```lua

 -- query mysql, memcached, and a remote http service at the same time,
 -- output the results in the order that they
 -- actually return the results.

 local mysql = require "resty.mysql"
 local memcached = require "resty.memcached"

 local function query_mysql()
     local db = mysql:new()
     db:connect{
                 host = "127.0.0.1",
                 port = 3306,
                 database = "test",
                 user = "monty",
                 password = "mypass"
               }
     local res, err, errno, sqlstate =
             db:query("select * from cats order by id asc")
     db:set_keepalive(0, 100)
     ngx.say("mysql done: ", cjson.encode(res))
 end

 local function query_memcached()
     local memc = memcached:new()
     memc:connect("127.0.0.1", 11211)
     local res, err = memc:get("some_key")
     ngx.say("memcached done: ", res)
 end

 local function query_http()
     local res = ngx.location.capture("/my-http-proxy")
     ngx.say("http done: ", res.body)
 end

 ngx.thread.spawn(query_mysql)      -- create thread 1
 ngx.thread.spawn(query_memcached)  -- create thread 2
 ngx.thread.spawn(query_http)       -- create thread 3
```

This API was first enabled in the `v0.7.0` release.

[Back to TOC](#nginx-api-for-lua)
