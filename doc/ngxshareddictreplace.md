ngx.shared.DICT.replace
-----------------------
**语法:** *success, err, forcible = ngx.shared.DICT:replace(key, value, exptime?, flags?)*

**环境:** *init_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

与 [set](#ngxshareddictset) 方法类似，但仅当存储字典 [ngx.shared.DICT](#ngxshareddict) 中 *存在* 该 key 时执行存储 key-value 对。

如果参数 `key` 在字典中 *不* 存在 (或已经过期)，`success` 返回值为 `false`，同时 `err` 返回 `"not found"` (没找到)。

这个功能最早出现在 `v0.3.1rc22` 版本中。

更多功能请参考 [ngx.shared.DICT](#ngxshareddict)。

> English Source

**syntax:** *success, err, forcible = ngx.shared.DICT:replace(key, value, exptime?, flags?)*

**context:** *init_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

Just like the [set](#ngxshareddictset) method, but only stores the key-value pair into the dictionary [ngx.shared.DICT](#ngxshareddict) if the key *does* exist.

If the `key` argument does *not* exist in the dictionary (or expired already), the `success` return value will be `false` and the `err` return value will be `"not found"`.

This feature was first introduced in the `v0.3.1rc22` release.

See also [ngx.shared.DICT](#ngxshareddict).

[返回目录](#nginx-api-for-lua)