**语法:** *ngx.update_time()*

**环境:** *init_worker_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

强行更新Nginx当前时间缓存。此调用会涉及到一个系统调用，因此会有一些系统开销，所以不要滥用。

这个API最早出现在 `v0.3.1rc32` 版本中。

> English Source

**syntax:** *ngx.update_time()*

**context:** *init_worker_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

Forcibly updates the Nginx current time cache. This call involves a syscall and thus has some overhead, so do not abuse it.

This API was first introduced in `v0.3.1rc32`.

[返回目录](#nginx-api-for-lua)
