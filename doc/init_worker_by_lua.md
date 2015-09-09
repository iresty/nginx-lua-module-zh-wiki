init_worker_by_lua
------------------

**语法:** *init_worker_by_lua &lt;lua-script-str&gt;*

**环境:** *http*

**阶段:** *starting-worker*

开启master进程模式，Nginx工作进程启动时执行指定的Lua代码。关闭master模式，将在[init_by_lua*](#init_by_lua)后直接运行。

这个指令经常被用来创建单进程的反复执行定时器（通过[ngx.timer.at](#ngxtimerat) Lua API创建），可以是后端服务健康检查，也可以是其他定时的日常工作。下面是个例子：

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

这个指令是在`v0.9.5`发行版第一次引入。

[返回目录](#directives)

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

