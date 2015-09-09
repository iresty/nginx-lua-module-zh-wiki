ngx.crc32_long
---------------
**语法:** *intval = ngx.crc32_long(str)*

**环境:** set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\**

通过一个字符串计算循环冗余校验码。

这个方法最好在字符串较多时调用（比如大于30-60字节），他的结果和 `ngx.crc32_short` 是一样的。

它只是一个被 `nginx` 底层定义的一个函数。<!-- todo 第一版翻译，待调调 -->

这个方法最早出现在版本 `v0.3.1rc8` 中。

```lua

 local args = ngx.decode_args(str, 0)
```

强烈不推荐移除 `max_args` 限制。

这个方法最早出现在版本 `v0.5.0rc29` 中。


> English Source

**syntax:** *intval = ngx.crc32_short(str)*

**context:** set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\**

Calculates the CRC-32 (Cyclic Redundancy Code) digest for the str argument.

This method performs better on relatively long str inputs (i.e., longer than 30 ~ 60 bytes), as compared to `ngx.crc32_short`. The result is exactly the same as `ngx.crc32_short`.

Behind the scene, it is just a thin wrapper around the `ngx_crc32_long` function defined in the Nginx core.

This API was first introduced in the `v0.3.1rc8` release.

[返回目录](#nginx-api-for-lua)