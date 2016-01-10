ngx.req.set_uri_args
--------------------
**语法:** *ngx.req.set_uri_args(args)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;*

用 `args` 参数重写当前请求的 URI 请求参数。`args` 参数可以是一个 Lua 字符串，比如

```lua

 ngx.req.set_uri_args("a=3&b=hello%20world")
```

或一个包含请求参数 key-value 对的 Lua table，例如

```lua

 ngx.req.set_uri_args({ a = 3, b = "hello world" })
```

在第二种情况下，本方法将根据 URI 转义规则转义参数的 key 和 value。

本方法也支持多值参数：

```lua

 ngx.req.set_uri_args({ a = 3, b = {5, 6} })
```

此时请求参数字符串为 `a=3&b=5&b=6`。

这个方法在 `v0.3.1rc13` 版本中首次引入。

更多用法请参考 [ngx.req.set_uri](#ngxreqset_uri)。


> English Source

**syntax:** *ngx.req.set_uri_args(args)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;*

Rewrite the current request's URI query arguments by the `args` argument. The `args` argument can be either a Lua string, as in

```lua

 ngx.req.set_uri_args("a=3&b=hello%20world")
```

or a Lua table holding the query arguments' key-value pairs, as in

```lua

 ngx.req.set_uri_args({ a = 3, b = "hello world" })
```

where in the latter case, this method will escape argument keys and values according to the URI escaping rule.

Multi-value arguments are also supported:

```lua

 ngx.req.set_uri_args({ a = 3, b = {5, 6} })
```

which will result in a query string like `a=3&b=5&b=6`.

This interface was first introduced in the `v0.3.1rc13` release.

See also [ngx.req.set_uri](#ngxreqset_uri).

[返回目录](#nginx-api-for-lua)
