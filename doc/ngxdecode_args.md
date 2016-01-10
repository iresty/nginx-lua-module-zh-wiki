ngx.decode_args
---------------
**语法:** *table = ngx.decode_args(str, max_args?)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

将 URI 编码的查询字符串解码为 Lua 表。本函数是 [ngx.encode_args](#ngxencode_args) 的逆函数。

可选的参数 `max_args` 可以用来指定从 `str` 中最多解析的参数个数。默认时，最多解析 100 个请求参数 (包括同名的)。为避免潜在的拒绝服务式攻击 (denial of services, DOS)，超过 `max_args` 数量上限的 URI 参数被丢弃，

这个参数可以被设成 0 以去掉解析参数数量上限：

```lua

 local args = ngx.decode_args(str, 0)
```

强烈不推荐移除 `max_args` 限制。

这个方法最早出现在版本 `v0.5.0rc29` 中。


> English Source

**syntax:** *table = ngx.decode_args(str, max_args?)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Decodes a URI encoded query-string into a Lua table. This is the inverse function of [ngx.encode_args](#ngxencode_args).

The optional `max_args` argument can be used to specify the maximum number of arguments parsed from the `str` argument. By default, a maximum of 100 request arguments are parsed (including those with the same name) and that additional URI arguments are silently discarded to guard against potential denial of service attacks.

This argument can be set to zero to remove the limit and to process all request arguments received:

```lua

 local args = ngx.decode_args(str, 0)
```

Removing the `max_args` cap is strongly discouraged.

This method was introduced in the `v0.5.0rc29`.


[返回目录](#nginx-api-for-lua)