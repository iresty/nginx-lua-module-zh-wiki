ngx.worker.pid
--------------

**语法:** *pid = ngx.worker.pid()*

**语法:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, init_by_lua*, init_worker_by_lua**

这个函数返回一个Lua数字，它是当前 Nginx 工作进程的进程 ID （PID）。这个 API 比 `ngx.var.pid` 更有效，[ngx.var.VARIABLE](#ngxvarvariable) API 不能使用的地方（例如 [init_worker_by_lua](#init_worker_by_lua)），该 API 是可以的。

该 API 在 `v0.6.0` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.worker.pid
--------------

**syntax:** *pid = ngx.worker.pid()*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, init_by_lua*, init_worker_by_lua**

This function returns a Lua number for the process ID (PID) of the current Nginx worker process. This API is more efficient than `ngx.var.pid` and can be used in contexts where the [ngx.var.VARIABLE](#ngxvarvariable) API cannot be used (like [init_worker_by_lua](#init_worker_by_lua)).

This API was first introduced in the `0.9.5` release.

[Back to TOC](#nginx-api-for-lua)

