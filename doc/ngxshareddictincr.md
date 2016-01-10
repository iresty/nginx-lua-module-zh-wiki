ngx.shared.DICT.incr
--------------------
**语法:** *newval, err = ngx.shared.DICT:incr(key, value)*

**环境:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

在基于共享内存的字典 [ngx.shared.DICT](#ngxshareddict) 中递增 `key` 的 (数字) 值，步长为 `value`。当操作成功时返回结果数字，否则返回 `nil` 和错误信息字符串。

这个 key 必须已经存储在字典中，否则返回 `nil` 和 `"not found"` (没找到)。

如果 key 的原始值不是一个有效的 Lua 数字，返回 `nil` 和 `"not a number"` (不是数字)。

参数 `value` 可以是任意有效的 Lua 数字，包括负数和浮点数。

这个功能最早出现在 `v0.3.1rc22` 版本中。

更多功能请参考 [ngx.shared.DICT](#ngxshareddict)。

> English Source

**syntax:** *newval, err = ngx.shared.DICT:incr(key, value)*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Increments the (numerical) value for `key` in the shm-based dictionary [ngx.shared.DICT](#ngxshareddict) by the step value `value`. Returns the new resulting number if the operation is successfully completed or `nil` and an error message otherwise.

The key must already exist in the dictionary, otherwise it will return `nil` and `"not found"`.

If the original value is not a valid Lua number in the dictionary, it will return `nil` and `"not a number"`.

The `value` argument can be any valid Lua numbers, like negative numbers or floating-point numbers.

This feature was first introduced in the `v0.3.1rc22` release.

See also [ngx.shared.DICT](#ngxshareddict).

[返回目录](#nginx-api-for-lua)