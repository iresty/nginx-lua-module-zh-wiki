ngx.shared.DICT.add
-------------------
**语法:** *success, err, forcible = ngx.shared.DICT:add(key, value, exptime?, flags?)*

**环境:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

类似 [set](#ngxshareddictset) 方法，但仅当存储字典 [ngx.shared.DICT](#ngxshareddict) 中 *不存在* 该 key 时执行存储 key-value 对。

如果参数 `key` 在字典中已经存在 (且没有过期)，`success` 返回值为 `false`，同时 `err` 返回 `"exist"` (已存在)。

这个功能最早出现在 `v0.3.1rc22` 版本中。

更多功能请参考 [ngx.shared.DICT](#ngxshareddict)。

> English Source

**syntax:** *success, err, forcible = ngx.shared.DICT:add(key, value, exptime?, flags?)*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Just like the [set](#ngxshareddictset) method, but only stores the key-value pair into the dictionary [ngx.shared.DICT](#ngxshareddict) if the key does *not* exist.

If the `key` argument already exists in the dictionary (and not expired for sure), the `success` return value will be `false` and the `err` return value will be `"exists"`.

This feature was first introduced in the `v0.3.1rc22` release.

See also [ngx.shared.DICT](#ngxshareddict).

[返回目录](#nginx-api-for-lua)