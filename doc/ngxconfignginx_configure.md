ngx.config.nginx_configure
--------------------------

**语法:** *str = ngx.config.nginx_configure()*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, init_by_lua**

该 API 返回编译 Nginx 时的 `./configure` 命令参数字符串。

该 API 在 `0.9.5` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.config.nginx_configure
--------------------------

**syntax:** *str = ngx.config.nginx_configure()*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, init_by_lua**

This function returns a string for the NGINX `./configure` command's arguments string.

This API was first introduced in the `0.9.5` release.

[Back to TOC](#nginx-api-for-lua)

