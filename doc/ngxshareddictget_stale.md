**语法:** *value, flags, stale = ngx.shared.DICT:get_stale(key)*

**环境** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

与 [get](#ngxshareddictget) 方法类似，但即使 key 已经过期依然返回值。

返回第三个值，`stale`，来标识该 key 是否已经过期。

需要注意的是，已过期的 key 无法保证存在，所以永远不应该依赖已过期项的可用性。

此方法最早出现在 `0.8.6` 版本中。

更多功能请参考 [ngx.shared.DICT](#ngxshareddict)。


> English Source

**syntax:** *value, flags, stale = ngx.shared.DICT:get_stale(key)*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

Similar to the [get](#ngxshareddictget) method but returns the value even if the key has already expired.

Returns a 3rd value, `stale`, indicating whether the key has expired or not.

Note that the value of an expired key is not guaranteed to be available so one should never rely on the availability of expired items.

This method was first introduced in the `0.8.6` release.

See also [ngx.shared.DICT](#ngxshareddict).
