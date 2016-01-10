print
-----
**语法:** *print(...)*

**环境:** *init_by_lua&#42;, init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;, ngx.timer.&#42;, balancer_by_lua&#42;, certificate_by_lua&#42;*

将参数值以 `ngx.NOTICE` 日志级别写入 nginx 的 `error.log` 文件。
Writes argument values into the nginx `error.log` file with the `ngx.NOTICE` log level.

等同于

```lua

 ngx.log(ngx.NOTICE, ...)
```

Lua 的 `nil` 值输出 `"nil"` 字符串，Lua 的布尔值输出 `"true"` 或 `"false"` 字符串。`ngx.null` 常量输出为 `"null"` 字符串。

在 Nginx 内核中硬编码限制了单条错误信息最长为 `2048` 字节。这个长度包含了最后的换行符和开始的时间戳。如果信息长度超过这个限制，Nginx 将把信息文本截断。这个限制可以通过修改 Nginx 源码中 `src/core/ngx_log.h` 文件中的 `NGX_MAX_ERROR_STR` 宏定义调整。


> English Source

**syntax:** *print(...)*

**context:** *init_by_lua&#42;, init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;, ngx.timer.&#42;, balancer_by_lua&#42;, certificate_by_lua&#42;*

Writes argument values into the nginx `error.log` file with the `ngx.NOTICE` log level.

It is equivalent to

```lua

 ngx.log(ngx.NOTICE, ...)
```

Lua `nil` arguments are accepted and result in literal `"nil"` strings while Lua booleans result in literal `"true"` or `"false"` strings. And the `ngx.null` constant will yield the `"null"` string output.

There is a hard coded `2048` byte limitation on error message lengths in the Nginx core. This limit includes trailing newlines and leading time stamps. If the message size exceeds this limit, Nginx will truncate the message text accordingly. This limit can be manually modified by editing the `NGX_MAX_ERROR_STR` macro definition in the `src/core/ngx_log.h` file in the Nginx source tree.


[返回目录](#nginx-api-for-lua)