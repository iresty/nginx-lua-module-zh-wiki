ngx.req.start_time
------------------
**语法:** *secs = ngx.req.start_time()*

**环境:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\*

返回当前请求创建时的时间戳，格式为浮点数，其中小数部分代表毫秒值。

以下用 Lua 代码模拟计算了 `$request_time` 变量值 (由 [ngx_http_log_module](http://nginx.org/en/docs/http/ngx_http_log_module.html) 模块生成)

```lua

 local request_time = ngx.now() - ngx.req.start_time()
```

这个函数在 `v0.7.7` 版本中首次引入。

更多使用方法请参考 [ngx.now](#ngxnow) 和 [ngx.update_time](#ngxupdate_time)。


> English Source

**syntax:** *secs = ngx.req.start_time()*

**context:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\*

Returns a floating-point number representing the timestamp (including milliseconds as the decimal part) when the current request was created.

The following example emulates the `$request_time` variable value (provided by [ngx_http_log_module](http://nginx.org/en/docs/http/ngx_http_log_module.html)) in pure Lua:

```lua

 local request_time = ngx.now() - ngx.req.start_time()
```

This function was first introduced in the `v0.7.7` release.

See also [ngx.now](#ngxnow) and [ngx.update_time](#ngxupdate_time).

[返回目录](#nginx-api-for-lua)
