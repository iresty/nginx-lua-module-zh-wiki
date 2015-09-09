ngx.encode_args
---------------
**语法:** *str = ngx.encode_args(table)*

**环境:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

根据 URI 编码规则，将 Lua 表编码成一个查询参数字符串。

例如，

```lua

 ngx.encode_args({foo = 3, ["b r"] = "hello world"})
```

生成


    foo=3&b%20r=hello%20world


Lua 表的 key 必须是 Lua 字符串。

支持多值参数。可以使用 Lua 表存储参数值，例如：

```lua

 ngx.encode_args({baz = {32, "hello"}})
```

输出


    baz=32&baz=hello


如果 value 表是空的，效果等同于 `nil` 值。

支持布尔值参数，例如，

```lua

 ngx.encode_args({a = true, b = 1})
```

输出


    a&b=1


如果参数值是 `false`，效果等同于 `nil` 值。

这个方法最早出现在版本 `v0.3.1rc27` 中。


> English Source

**syntax:** *str = ngx.encode_args(table)*

**context:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

Encode the Lua table to a query args string according to the URI encoded rules.

For example,

```lua

 ngx.encode_args({foo = 3, ["b r"] = "hello world"})
```

yields


    foo=3&b%20r=hello%20world


The table keys must be Lua strings.

Multi-value query args are also supported. Just use a Lua table for the argument's value, for example:

```lua

 ngx.encode_args({baz = {32, "hello"}})
```

gives


    baz=32&baz=hello


If the value table is empty and the effect is equivalent to the `nil` value.

Boolean argument values are also supported, for instance,

```lua

 ngx.encode_args({a = true, b = 1})
```

yields


    a&b=1


If the argument value is `false`, then the effect is equivalent to the `nil` value.

This method was first introduced in the `v0.3.1rc27` release.


[返回目录](#nginx-api-for-lua)