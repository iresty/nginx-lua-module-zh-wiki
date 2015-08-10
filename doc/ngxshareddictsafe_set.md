ngx.shared.DICT.safe_set
------------------------
**语法:** *ok, err = ngx.shared.DICT:safe_set(key, value, exptime?, flags?)*

**环境:** *init_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

类似 [set](#ngxshareddictset) 方法，但当共享内存区块存储空间不足时，不覆盖 (最近最少使用的) 有效的项 (非过期项)。此时，它将返回 `nil` 和字符串 "no memory" (内存不足)。
Similar to the [set](#ngxshareddictset) method, but never overrides the (least recently used) unexpired items in the store when running out of storage in the shared memory zone. In this case, it will immediately return `nil` and the string "no memory".

此方法最早出现在 `0.7.18` 版本中。

更多功能请参考 [ngx.shared.DICT](#ngxshareddict)。


> English Source

**syntax:** *ok, err = ngx.shared.DICT:safe_set(key, value, exptime?, flags?)*

**context:** *init_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

Similar to the [set](#ngxshareddictset) method, but never overrides the (least recently used) unexpired items in the store when running out of storage in the shared memory zone. In this case, it will immediately return `nil` and the string "no memory".

This feature was first introduced in the `v0.7.18` release.

See also [ngx.shared.DICT](#ngxshareddict).

[返回目录](#nginx-api-for-lua)