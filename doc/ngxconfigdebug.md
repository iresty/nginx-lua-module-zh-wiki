ngx.config.debug
----------------
**syntax:** *debug = ngx.config.debug*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, init_by_lua*, init_worker_by_lua**

这个布尔值代表当前 Nginx 是否为调试版本，既，在编译时使用`./configure`的可选项`--with-debug`。

该 API 在 `0.8.7` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.config.debug
----------------
**syntax:** *debug = ngx.config.debug*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, init_by_lua*, init_worker_by_lua**

This boolean field indicates whether the current Nginx is a debug build, i.e., being built by the `./configure` option `--with-debug`.

This field was first introduced in the `0.8.7`.

[Back to TOC](#nginx-api-for-lua)

