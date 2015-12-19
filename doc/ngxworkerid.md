ngx.worker.id
-------------

**语法:** *count = ngx.worker.id()*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, init_by_lua**

返回当前 Nginx 工作进程的一个顺序数字（从 0 开始）。

所以，如果工作进程总数是 `N`，那么该方法将返回 0 和 `N - 1` （包含）的一个数字。

该方法只对 NGINX 1.9.1+ 版本返回有意义的值。更早版本的 nginx，将总是返回 `nil` 。 

同样可以看看 [ngx.worker.count](#ngxworkercount)。

该 API 从 `v0.9.20` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.worker.id
-------------

**syntax:** *count = ngx.worker.id()*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, init_by_lua**

Returns the ordinal number of the current Nginx worker processes (starting from number 0).

So if the total number of workers is `N`, then this method may return a number between 0
and `N - 1` (inclusive).

This function returns meaningful values only for NGINX 1.9.1+. With earlier versions of NGINX, it
always returns `nil`.

See also [ngx.worker.count](#ngxworkercount).

This API was first introduced in the `0.9.20` release.

[Back to TOC](#nginx-api-for-lua)