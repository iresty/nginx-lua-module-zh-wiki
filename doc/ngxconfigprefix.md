ngx.config.prefix
-----------------

**语法:** *prefix = ngx.config.prefix()*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, init_by_lua*, init_worker_by_lua**

返回 Nginx 服务的 "prefix" 路径，它可能是由 Nginx 启动时通过可选 `-p` 命令行确定的，也可能是由编译 Nginx 的 `./configure` 脚本中可选的 `--prefix` 命令行参数确定的。

该 API 在 `0.9.2` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.config.prefix
-----------------

**syntax:** *prefix = ngx.config.prefix()*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, init_by_lua*, init_worker_by_lua**

Returns the Nginx server "prefix" path, as determined by the `-p` command-line option when running the nginx executable, or the path specified by the `--prefix` command-line option when building Nginx with the `./configure` script.

This function was first introduced in the `0.9.2`.

[Back to TOC](#nginx-api-for-lua)

