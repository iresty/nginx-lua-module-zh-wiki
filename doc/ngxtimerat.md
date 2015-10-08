ngx.timer.at
------------
**语法:** *ok, err = ngx.timer.at(delay, callback, user_arg1, user_arg2, ...)*

**环境:** *init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

使用一个可选参数的用户回调函数，创建一个 Nginx 定时器。

第一个参数 `delay`，指定这个定时器的延迟时间，秒为单位。我们可以指定带有小数部分的时间像`0.001`，这里代表 1 毫秒。当需要立即执行当前挂起的处理，指定延迟为`0`即可。

第二个参数`callback`，可以是任何 Lua 函数，在指定的延迟时间之后，将会在一个后台的“轻线程”中被调用。这个调用是 Nginx 内核使用参数 `premature`，`user_arg1`， `user_arg2`等参数自动完成的，`premature`参数代表这个定时器是否过期的布尔值，`user_arg1`， `user_arg2`等其他参数是调用`ngx.timer.at`的其余参数。

当 Nginx 工作进程正在尝试关闭时，定时器会过早失效，例如 `HUP` 信号触发的 Nginx 配置文件重载或 Nginx 服务关闭。当 Nginx 工作进程正在关闭，是不能调用 `ngx.timer.at` 来创建新的非零时间延迟定时器，这种情况下`ngx.timer.at`将返回`nil`和一个描述这个错误的字符串信息："process exiting"。

从`v0.9.3`版本开始，当 Nginx 工作进程开始关闭时，是允许创建零延迟定时器的。

当定时器到期，定时器中的 Lua 代码是在一个“请线程”中运行的，它与创造它的原始请求是完全分离的。因此，和创造它的请求有相同生命周期的对象，比如 [cosockets](#ngxsockettcp)，是不能在原始请求和定时器中的回调函数共享使用的。

这是个简单的例子：

```nginx

 location / {
     ...
     log_by_lua '
         local function push_data(premature, uri, args, status)
             -- push the data uri, args, and status to the remote
             -- via ngx.socket.tcp or ngx.socket.udp
             -- (one may want to buffer the data in Lua a bit to
             -- save I/O operations)
         end
         local ok, err = ngx.timer.at(0, push_data,
                                      ngx.var.uri, ngx.var.args, ngx.header.status)
         if not ok then
             ngx.log(ngx.ERR, "failed to create timer: ", err)
             return
         end
     ';
 }
```

我们可以创建反复循环使用的定时器，例如，得到一个每`5`秒触发一次的定时器，可以在定时器中递归调用`ngx.timer.at`。这里有个这样的例子：

```lua

 local delay = 5
 local handler
 handler = function (premature)
     -- do some routine job in Lua just like a cron job
     if premature then
         return
     end
     local ok, err = ngx.timer.at(delay, handler)
     if not ok then
         ngx.log(ngx.ERR, "failed to create the timer: ", err)
         return
     end
 end

 local ok, err = ngx.timer.at(delay, handler)
 if not ok then
     ngx.log(ngx.ERR, "failed to create the timer: ", err)
     return
 end
```

因为定时调用是在后台运行，并且他们的执行不会增加任何客户端的响应时长，所以它们很容易让服务端累积错误并耗尽系统资源，有可能是 Lua 编程错误也可能仅仅是太多客户端请求。为了防止这种极端的恶果（例如： Nginx 服务崩溃），在 Nginx 工作进程里内建了 "pending timers" 和 "running timers" 两个数量限制。这里的 "pending timers" 代表还没有过期的定时器，"running timers" 代表用户回调函数当前正在运行的定时器。

在 Nginx 进程内 "pending timers" 的最大数控制是 [lua_max_pending_timers](#lua_max_pending_timers) 指令完成的。 "running timers" 的最大数控制是 [lua_max_running_timers](#lua_max_running_timers) 指令完成的。

根据当前实现，每一个 "running timer" ，都将从全局连接列表中占用一个（假）连接，全局列表通过 `nginx.conf` 的标准指令 [worker_connections](http://nginx.org/en/docs/ngx_core_module.html#worker_connections) 配置。所以要确保 [worker_connections](http://nginx.org/en/docs/ngx_core_module.html#worker_connections) 指令设置了足够大的值，用来存放真正的连接和定时器需要假连接（受限于 [lua_max_running_timers](#lua_max_running_timers) 指令）。

在定时器的回调函数中，很多 Nginx 的 Lua API 是可用的，像数据流/数据报 cosocket 的 ([ngx.socket.tcp](#ngxsockettcp) 和 [ngx.socket.udp](#ngxsocketudp))， 共享内存字典 ([ngx.shared.DICT](#ngxshareddict))， 用户协同程序 ([coroutine.*](#coroutinecreate))，用户 "轻线程" ([ngx.thread.*](#ngxthreadspawn))， [ngx.exit](#ngxexit)， [ngx.now](#ngxnow)/[ngx.time](#ngxtime)，[ngx.md5](#ngxmd5)/[ngx.sha1_bin](#ngxsha1_bin)， 都是允许的。但是子请求 API （如 [ngx.location.capture](#ngxlocationcapture))， [ngx.req.*](#ngxreqstart_time) , 下游输出 API（如 [ngx.say](#ngxsay)， [ngx.print](#ngxprint) 和 [ngx.flush](#ngxflush)）， 在这个环境是明确被禁用的。

定时器的回调函数可传入大多数标准 Lua 值（空、布尔值、数字、字符串、表、闭包、文件句柄等），无论是明确的用户参数或回调闭包的隐式值。这里有几个例外，通过 [coroutine.create](#coroutinecreate) 和 [ngx.thread.spawn](#ngxthreadspawn) 的线程对象，或通过 [ngx.socket.tcp](#ngxsockettcp)、 [ngx.socket.udp](#ngxsocketudp) 和 [ngx.req.socket](#ngxreqsocket) 得到的 cosocket 对象，他们都是 *不能* 作为传入对象的，因为这些对象的生命周期都是绑定到创建定时器的请求环境的，但定时器回调与创建环境是完全隔离的（设计上），并且是在自己的（假）请求环境中运行。如果你尝试在创建请求边界共享线程或 cosocket 对象，你将得到错误信息 "no co ctx found"（对于线程），"bad request"（对于 cosockets）。它是好的，所以，在你的定时器回调中创建所有这些对象。

这个 API 是从`v0.8.0`首次引入的。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.timer.at
------------
**syntax:** *ok, err = ngx.timer.at(delay, callback, user_arg1, user_arg2, ...)*

**context:** *init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

Creates an Nginx timer with a user callback function as well as optional user arguments.

The first argument, `delay`, specifies the delay for the timer,
in seconds. One can specify fractional seconds like `0.001` to mean 1
millisecond here. `0` delay can also be specified, in which case the
timer will immediately expire when the current handler yields
execution.

The second argument, `callback`, can
be any Lua function, which will be invoked later in a background
"light thread" after the delay specified. The user callback will be
called automatically by the Nginx core with the arguments `premature`,
`user_arg1`, `user_arg2`, and etc, where the `premature`
argument takes a boolean value indicating whether it is a premature timer
expiration or not, and `user_arg1`, `user_arg2`, and etc, are
those (extra) user arguments specified when calling `ngx.timer.at`
as the remaining arguments.

Premature timer expiration happens when the Nginx worker process is
trying to shut down, as in an Nginx configuration reload triggered by
the `HUP` signal or in an Nginx server shutdown. When the Nginx worker
is trying to shut down, one can no longer call `ngx.timer.at` to
create new timers with nonzero delays and in that case `ngx.timer.at` will return `nil` and
a string describing the error, that is, "process exiting".

Starting from the `v0.9.3` release, it is allowed to create zero-delay timers even when the Nginx worker process starts shutting down.

When a timer expires, the user Lua code in the timer callback is
running in a "light thread" detached completely from the original
request creating the timer. So objects with the same lifetime as the
request creating them, like [cosockets](#ngxsockettcp), cannot be shared between the
original request and the timer user callback function.

Here is a simple example:

```nginx

 location / {
     ...
     log_by_lua '
         local function push_data(premature, uri, args, status)
             -- push the data uri, args, and status to the remote
             -- via ngx.socket.tcp or ngx.socket.udp
             -- (one may want to buffer the data in Lua a bit to
             -- save I/O operations)
         end
         local ok, err = ngx.timer.at(0, push_data,
                                      ngx.var.uri, ngx.var.args, ngx.header.status)
         if not ok then
             ngx.log(ngx.ERR, "failed to create timer: ", err)
             return
         end
     ';
 }
```

One can also create infinite re-occuring timers, for instance, a timer getting triggered every `5` seconds, by calling `ngx.timer.at` recursively in the timer callback function. Here is such an example,

```lua

 local delay = 5
 local handler
 handler = function (premature)
     -- do some routine job in Lua just like a cron job
     if premature then
         return
     end
     local ok, err = ngx.timer.at(delay, handler)
     if not ok then
         ngx.log(ngx.ERR, "failed to create the timer: ", err)
         return
     end
 end

 local ok, err = ngx.timer.at(delay, handler)
 if not ok then
     ngx.log(ngx.ERR, "failed to create the timer: ", err)
     return
 end
```

Because timer callbacks run in the background and their running time
will not add to any client request's response time, they can easily
accumulate in the server and exhaust system resources due to either
Lua programming mistakes or just too much client traffic. To prevent
extreme consequences like crashing the Nginx server, there are
built-in limitations on both the number of "pending timers" and the
number of "running timers" in an Nginx worker process. The "pending
timers" here mean timers that have not yet been expired and "running
timers" are those whose user callbacks are currently running.

The maximal number of pending timers allowed in an Nginx
worker is constrolled by the [lua_max_pending_timers](#lua_max_pending_timers)
directive. The maximal number of running timers is controlled by the
[lua_max_running_timers](#lua_max_running_timers) directive.

According to the current implementation, each "running timer" will
take one (fake) connection record from the global connection record
list configured by the standard [worker_connections](http://nginx.org/en/docs/ngx_core_module.html#worker_connections) directive in
`nginx.conf`. So ensure that the
[worker_connections](http://nginx.org/en/docs/ngx_core_module.html#worker_connections) directive is set to
a large enough value that takes into account both the real connections
and fake connections required by timer callbacks (as limited by the
[lua_max_running_timers](#lua_max_running_timers) directive).

A lot of the Lua APIs for Nginx are enabled in the context of the timer
callbacks, like stream/datagram cosockets ([ngx.socket.tcp](#ngxsockettcp) and [ngx.socket.udp](#ngxsocketudp)), shared
memory dictionaries ([ngx.shared.DICT](#ngxshareddict)), user coroutines ([coroutine.*](#coroutinecreate)),
user "light threads" ([ngx.thread.*](#ngxthreadspawn)), [ngx.exit](#ngxexit), [ngx.now](#ngxnow)/[ngx.time](#ngxtime),
[ngx.md5](#ngxmd5)/[ngx.sha1_bin](#ngxsha1_bin), are all allowed. But the subrequest API (like
[ngx.location.capture](#ngxlocationcapture)), the [ngx.req.*](#ngxreqstart_time) API, the downstream output API
(like [ngx.say](#ngxsay), [ngx.print](#ngxprint), and [ngx.flush](#ngxflush)) are explicitly disabled in
this context.

You can pass most of the standard Lua values (nils, booleans, numbers, strings, tables, closures, file handles, and etc) into the timer callback, either explicitly as user arguments or implicitly as upvalues for the callback closure. There are several exceptions, however: you *cannot* pass any thread objects returned by [coroutine.create](#coroutinecreate) and [ngx.thread.spawn](#ngxthreadspawn) or any cosocket objects returned by [ngx.socket.tcp](#ngxsockettcp), [ngx.socket.udp](#ngxsocketudp), and [ngx.req.socket](#ngxreqsocket) because these objects' lifetime is bound to the request context creating them while the timer callback is detached from the creating request's context (by design) and runs in its own (fake) request context. If you try to share the thread or cosocket objects across the boundary of the creating request, then you will get the "no co ctx found" error (for threads) or "bad request" (for cosockets). It is fine, however, to create all these objects inside your timer callback.

This API was first introduced in the `v0.8.0` release.

[Back to TOC](#nginx-api-for-lua)
