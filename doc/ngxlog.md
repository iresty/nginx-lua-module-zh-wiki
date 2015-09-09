ngx.log
-------
**语法:** *ngx.log(log_level, ...)*

**环境:** *init_by_lua*\**, init_worker_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

将参数拼接起来，按照设定的日志级别记入 error.log。

Lua `nil` 参数将输出 `"nil"` 字符串；Lua 布尔参数将输出 `"true"` 或 `"false"` 字符串；`ngx.null` 常量将输出 `"null"` 字符串。

`log_level` 参数可以使用类似 `ngx.ERR` 和 `ngx.WARN` 的常量。更多信息请参考 [Nginx log level constants](#nginx-log-level-constants)。

在 Nginx 内核中硬编码限制了单条错误信息最长为 `2048` 字节。这个长度包含了最后的换行符和开始的时间戳。如果信息长度超过这个限制，Nginx 将把信息文本截断。这个限制可以通过修改 Nginx 源码中 `src/core/ngx_log.h` 文件中的 `NGX_MAX_ERROR_STR` 宏定义调整。

> English Source

**syntax:** *ngx.log(log_level, ...)*

**context:** *init_by_lua*\**, init_worker_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

Log arguments concatenated to error.log with the given logging level.

Lua `nil` arguments are accepted and result in literal `"nil"` string while Lua booleans result in literal `"true"` or `"false"` string outputs. And the `ngx.null` constant will yield the `"null"` string output.

The `log_level` argument can take constants like `ngx.ERR` and `ngx.WARN`. Check out [Nginx log level constants](#nginx-log-level-constants) for details.

There is a hard coded `2048` byte limitation on error message lengths in the Nginx core. This limit includes trailing newlines and leading time stamps. If the message size exceeds this limit, Nginx will truncate the message text accordingly. This limit can be manually modified by editing the `NGX_MAX_ERROR_STR` macro definition in the `src/core/ngx_log.h` file in the Nginx source tree.

[返回目录](#nginx-api-for-lua)