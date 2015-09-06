init_worker_by_lua
------------------

**语法:** *init_worker_by_lua &lt;lua-script-str&gt;*

**作用域:** *http*

**阶段:** *starting-worker*

Runs the specified Lua code upon every Nginx worker process's startup when the master process is enabled. When the master process is disabled, this hook will just run after [init_by_lua*](#init_by_lua).

This hook is often used to create per-worker reoccurring timers (via the [ngx.timer.at](#ngxtimerat) Lua API), either for backend healthcheck or other timed routine work. Below is an example,

```nginx

 init_worker_by_lua '
     local delay = 3  -- in seconds
     local new_timer = ngx.timer.at
     local log = ngx.log
     local ERR = ngx.ERR
     local check

     check = function(premature)
         if not premature then
             -- do the health check or other routine work
             local ok, err = new_timer(delay, check)
             if not ok then
                 log(ERR, "failed to create timer: ", err)
                 return
             end
         end
     end

     local ok, err = new_timer(delay, check)
     if not ok then
         log(ERR, "failed to create timer: ", err)
         return
     end
 ';
```

This directive was first introduced in the `v0.9.5` release.

[Back to TOC](#directives)

> English source:

init_worker_by_lua
------------------

**syntax:** *init_worker_by_lua &lt;lua-script-str&gt;*

**context:** *http*

**phase:** *starting-worker*

Runs the specified Lua code upon every Nginx worker process's startup when the master process is enabled. When the master process is disabled, this hook will just run after [init_by_lua*](#init_by_lua).

This hook is often used to create per-worker reoccurring timers (via the [ngx.timer.at](#ngxtimerat) Lua API), either for backend healthcheck or other timed routine work. Below is an example,

```nginx

 init_worker_by_lua '
     local delay = 3  -- in seconds
     local new_timer = ngx.timer.at
     local log = ngx.log
     local ERR = ngx.ERR
     local check

     check = function(premature)
         if not premature then
             -- do the health check or other routine work
             local ok, err = new_timer(delay, check)
             if not ok then
                 log(ERR, "failed to create timer: ", err)
                 return
             end
         end
     end

     local ok, err = new_timer(delay, check)
     if not ok then
         log(ERR, "failed to create timer: ", err)
         return
     end
 ';
```

This directive was first introduced in the `v0.9.5` release.

[Back to TOC](#directives)

