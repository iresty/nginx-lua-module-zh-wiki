ngx.thread.wait
---------------
**语法:** *ok, res1, res2, ... = ngx.thread.wait(thread1, thread2, ...)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

等待一个或多个子“轻线程”，并等待第一个终止（无论成功或有错误）“轻线程”的返回结果。

参数 `thread1`、`thread2` 等都是之前调用 [ngx.thread.spawn](#ngxthreadspawn) 返回的 Lua 线程对象。

返回值与 [coroutine.resume](#coroutineresume) 是完全一样的，也就是说，第一个返回值是一个布尔值，说明“轻线程”的终止是成功还是异常，随后的返回值是 Lua 函数的返回结果，该 Lua 函数是被用来产生“轻线程”（成功情况下）或错误对象（失败情况下）。

只有直属“父协程”才能等待它的子“轻线程”，否则将会有 Lua 异常抛出。

下面的示范例子，是使用 `ngx.thread.wait` 和 [ngx.location.capture](#ngxlocationcapture) 共同来模拟 [ngx.location.capture_multi](#ngxlocationcapture_multi) ：

```lua

 local capture = ngx.location.capture
 local spawn = ngx.thread.spawn
 local wait = ngx.thread.wait
 local say = ngx.say

 local function fetch(uri)
     return capture(uri)
 end

 local threads = {
     spawn(fetch, "/foo"),
     spawn(fetch, "/bar"),
     spawn(fetch, "/baz")
 }

 for i = 1, #threads do
     local ok, res = wait(threads[i])
     if not ok then
         say(i, ": failed to run: ", res)
     else
         say(i, ": status: ", res.status)
         say(i, ": body: ", res.body)
     end
 end
```

这里它实质实现是“等待所有”模型。

下面的例子是示范“等待任何”模型：

```lua

 function f()
     ngx.sleep(0.2)
     ngx.say("f: hello")
     return "f done"
 end

 function g()
     ngx.sleep(0.1)
     ngx.say("g: hello")
     return "g done"
 end

 local tf, err = ngx.thread.spawn(f)
 if not tf then
     ngx.say("failed to spawn thread f: ", err)
     return
 end

 ngx.say("f thread created: ", coroutine.status(tf))

 local tg, err = ngx.thread.spawn(g)
 if not tg then
     ngx.say("failed to spawn thread g: ", err)
     return
 end

 ngx.say("g thread created: ", coroutine.status(tg))

 ok, res = ngx.thread.wait(tf, tg)
 if not ok then
     ngx.say("failed to wait: ", res)
     return
 end

 ngx.say("res: ", res)

 -- stop the "world", aborting other running threads
 -- 停止这个“世界”，终止其他正在运行的线程
 ngx.exit(ngx.OK)
```

它将生成下面的输出：


    f thread created: running
    g thread created: running
    g: hello
    res: g done

该 API 是从 `v0.7.0` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.thread.wait
---------------
**syntax:** *ok, res1, res2, ... = ngx.thread.wait(thread1, thread2, ...)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Waits on one or more child "light threads" and returns the results of the first "light thread" that terminates (either successfully or with an error).

The arguments `thread1`, `thread2`, and etc are the Lua thread objects returned by earlier calls of [ngx.thread.spawn](#ngxthreadspawn).

The return values have exactly the same meaning as [coroutine.resume](#coroutineresume), that is, the first value returned is a boolean value indicating whether the "light thread" terminates successfully or not, and subsequent values returned are the return values of the user Lua function that was used to spawn the "light thread" (in case of success) or the error object (in case of failure).

Only the direct "parent coroutine" can wait on its child "light thread", otherwise a Lua exception will be raised.

The following example demonstrates the use of `ngx.thread.wait` and [ngx.location.capture](#ngxlocationcapture) to emulate [ngx.location.capture_multi](#ngxlocationcapture_multi):

```lua

 local capture = ngx.location.capture
 local spawn = ngx.thread.spawn
 local wait = ngx.thread.wait
 local say = ngx.say

 local function fetch(uri)
     return capture(uri)
 end

 local threads = {
     spawn(fetch, "/foo"),
     spawn(fetch, "/bar"),
     spawn(fetch, "/baz")
 }

 for i = 1, #threads do
     local ok, res = wait(threads[i])
     if not ok then
         say(i, ": failed to run: ", res)
     else
         say(i, ": status: ", res.status)
         say(i, ": body: ", res.body)
     end
 end
```

Here it essentially implements the "wait all" model.

And below is an example demonstrating the "wait any" model:

```lua

 function f()
     ngx.sleep(0.2)
     ngx.say("f: hello")
     return "f done"
 end

 function g()
     ngx.sleep(0.1)
     ngx.say("g: hello")
     return "g done"
 end

 local tf, err = ngx.thread.spawn(f)
 if not tf then
     ngx.say("failed to spawn thread f: ", err)
     return
 end

 ngx.say("f thread created: ", coroutine.status(tf))

 local tg, err = ngx.thread.spawn(g)
 if not tg then
     ngx.say("failed to spawn thread g: ", err)
     return
 end

 ngx.say("g thread created: ", coroutine.status(tg))

 ok, res = ngx.thread.wait(tf, tg)
 if not ok then
     ngx.say("failed to wait: ", res)
     return
 end

 ngx.say("res: ", res)

 -- stop the "world", aborting other running threads
 ngx.exit(ngx.OK)
```

And it will generate the following output:


    f thread created: running
    g thread created: running
    g: hello
    res: g done


This API was first enabled in the `v0.7.0` release.

[Back to TOC](#nginx-api-for-lua)

ngx.thread.kill
---------------
**syntax:** *ok, err = ngx.thread.kill(thread)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Kills a running "light thread" created by [ngx.thread.spawn](#ngxthreadspawn). Returns a true value when successful or `nil` and a string describing the error otherwise.

According to the current implementation, only the parent coroutine (or "light thread") can kill a thread. Also, a running "light thread" with pending NGINX subrequests (initiated by [ngx.location.capture](#ngxlocationcapture) for example) cannot be killed due to a limitation in the NGINX core.

This API was first enabled in the `v0.9.9` release.

[Back to TOC](#nginx-api-for-lua)