ngx.worker.count
----------------

**语法:** *count = ngx.worker.count()*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, init_by_lua**

返回当前 Nginx 工作进程数的数量（既：在 `nginx.conf` 配置中，使用 [worker_processes](http://nginx.org/en/docs/ngx_core_module.html#worker_processes) 指令配置的值）。

该 API 从 `v0.9.20` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.worker.count
----------------

**syntax:** *count = ngx.worker.count()*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, init_by_lua**

Returns the total number of the Nginx worker processes (i.e., the value configured
by the [worker_processes](http://nginx.org/en/docs/ngx_core_module.html#worker_processes)
directive in `nginx.conf`).

This API was first introduced in the `0.9.20` release.

[Back to TOC](#nginx-api-for-lua)
