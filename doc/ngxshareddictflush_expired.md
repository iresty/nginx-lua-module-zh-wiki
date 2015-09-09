ngx.shared.DICT.flush_expired
-----------------------------
**语法:** *flushed = ngx.shared.DICT:flush_expired(max_count?)*

**环境:** *init_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

清除字典中已过期的的内容，最多清除可选参数 `max_count` (最大数量) 个。当参数 `max_count` 值为 `0` 或者未指定时，意为无数量限制。返回值为实际清除的数量。

与 [flush_all](#ngxshareddictflush_all) 方法不同，此方法释放删除掉的已过期内容占用的内存。

这个功能最早出现在 `v0.6.3` 版本中。

更多功能请参考 [ngx.shared.DICT.flush_all](#ngxshareddictflush_all) 和 [ngx.shared.DICT](#ngxshareddict)。

> English Source

**syntax:** *flushed = ngx.shared.DICT:flush_expired(max_count?)*

**context:** *init_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

Flushes out the expired items in the dictionary, up to the maximal number specified by the optional `max_count` argument. When the `max_count` argument is given `0` or not given at all, then it means unlimited. Returns the number of items that have actually been flushed.

Unlike the [flush_all](#ngxshareddictflush_all) method, this method actually free up the memory used by the expired items.

This feature was first introduced in the `v0.6.3` release.

See also [ngx.shared.DICT.flush_all](#ngxshareddictflush_all) and [ngx.shared.DICT](#ngxshareddict).

[返回目录](#nginx-api-for-lua)