ngx.shared.DICT.safe_add
------------------------
**语法:** *ok, err = ngx.shared.DICT:safe_add(key, value, exptime?, flags?)*

**环境:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

类似 [add](#ngxshareddictadd) 方法，但当共享内存区块存储空间不足时，不覆盖 (最近最少使用的) 有效的项 (非过期项)。此时，它将返回 `nil` 和字符串 "no memory" (内存不足)。

这个功能最早出现在 `v0.7.18` 版本中。

更多功能请参考 [ngx.shared.DICT](#ngxshareddict)。

> English Source

**syntax:** *ok, err = ngx.shared.DICT:safe_add(key, value, exptime?, flags?)*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Similar to the [add](#ngxshareddictadd) method, but never overrides the (least recently used) unexpired items in the store when running out of storage in the shared memory zone. In this case, it will immediately return `nil` and the string "no memory".

This feature was first introduced in the `v0.7.18` release.

See also [ngx.shared.DICT](#ngxshareddict).

[返回目录](#nginx-api-for-lua)