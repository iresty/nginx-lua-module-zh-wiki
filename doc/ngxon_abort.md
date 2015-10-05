ngx.on_abort
------------
**语法:** *ok, err = ngx.on_abort(callback)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

注册一个用户回调函数，当客户端过早关闭当前连接（下游）时自动调用。

如果回掉函数注册成功返回 `1` ，或反之将返回 `nil` 和一个错误描述字符信息。

所有的 [Nginx Lua 的 API ](#nginx-api-for-lua) 都可以在这个回调中使用，因为这个函数是以一个特定的 “轻线程” 方式运行，就像其他使用 [ngx.thread.spawn](#ngxthreadspawn) 创建的 “轻线程”。

该回调函数能自己决定对客户端终止事件做什么处理。例如，它可以简单的不做任何事情从而忽略这个事件
，使得当前 Lua 请求处理可以没有任何打扰的继续执行。当然该回调函数也可以调用 [ngx.exit](#ngxexit) 从而终止所有处理，例如：

```lua

 local function my_cleanup()
     -- custom cleanup work goes here, like cancelling a pending DB transaction

     -- now abort all the "light threads" running in the current request handler
     ngx.exit(499)
 end

 local ok, err = ngx.on_abort(my_cleanup)
 if not ok then
     ngx.log(ngx.ERR, "failed to register the on_abort callback: ", err)
     ngx.exit(500)
 end
```

当 [lua_check_client_abort](#lua_check_client_abort) 被设置为 `off` （这是默认值），这时这个函数调用将永远返回错误信息 “lua_check_client_abort is off” 。

根据当前实现，这个函数在单个请求中能且只能调用一次；随后的调用将收到错误消息 “duplicate call” 。

该 API 是从 `v0.7.4` 版本首次引入。

也可以看看 [lua_check_client_abort](#lua_check_client_abort) 。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.on_abort
------------
**syntax:** *ok, err = ngx.on_abort(callback)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

Registers a user Lua function as the callback which gets called automatically when the client closes the (downstream) connection prematurely.

Returns `1` if the callback is registered successfully or returns `nil` and a string describing the error otherwise.

All the [Nginx API for Lua](#nginx-api-for-lua) can be used in the callback function because the function is run in a special "light thread", just as those "light threads" created by [ngx.thread.spawn](#ngxthreadspawn).

The callback function can decide what to do with the client abortion event all by itself. For example, it can simply ignore the event by doing nothing and the current Lua request handler will continue executing without interruptions. And the callback function can also decide to terminate everything by calling [ngx.exit](#ngxexit), for example,

```lua

 local function my_cleanup()
     -- custom cleanup work goes here, like cancelling a pending DB transaction

     -- now abort all the "light threads" running in the current request handler
     ngx.exit(499)
 end

 local ok, err = ngx.on_abort(my_cleanup)
 if not ok then
     ngx.log(ngx.ERR, "failed to register the on_abort callback: ", err)
     ngx.exit(500)
 end
```

When [lua_check_client_abort](#lua_check_client_abort) is set to `off` (which is the default), then this function call will always return the error message "lua_check_client_abort is off".

According to the current implementation, this function can only be called once in a single request handler; subsequent calls will return the error message "duplicate call".

This API was first introduced in the `v0.7.4` release.

See also [lua_check_client_abort](#lua_check_client_abort).

[Back to TOC](#nginx-api-for-lua)