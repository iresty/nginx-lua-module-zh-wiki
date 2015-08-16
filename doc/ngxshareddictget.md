ngx.shared.DICT.get
-------------------
**语法:** *value, flags = ngx.shared.DICT:get(key)*

**环境:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

从 [ngx.shared.DICT](#ngxshareddict) 字典中获取名为 `key` 的键 (key) 值。如果此 key 不存在或已过期，返回 `nil`。

当发生错误时，将返回 `nil` 和一个描述错误的字符串。

返回值将保持写入字典时的原始数据类型，例如，Lua 布尔型、数字型、或字符串型。

此方法的第一个参数必须是该字典对象本身，例如，

```lua

 local cats = ngx.shared.cats
 local value, flags = cats.get(cats, "Marry")
```

或使用 Lua 的“语法糖”形式来调用方法：

```lua

 local cats = ngx.shared.cats
 local value, flags = cats:get("Marry")
```

这两种形式完全等价。

如果用户标志 (`flags`) 是 `0` (默认值)，将不会返回 `flags` 值。

这个功能最早出现在 `v0.3.1rc22` 版本中。

更多功能请参考 [ngx.shared.DICT](#ngxshareddict)。


> English Source

**syntax:** *value, flags = ngx.shared.DICT:get(key)*

**context:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

Retrieving the value in the dictionary [ngx.shared.DICT](#ngxshareddict) for the key `key`. If the key does not exist or has been expired, then `nil` will be returned.

In case of errors, `nil` and a string describing the error will be returned.

The value returned will have the original data type when they were inserted into the dictionary, for example, Lua booleans, numbers, or strings.

The first argument to this method must be the dictionary object itself, for example,

```lua

 local cats = ngx.shared.cats
 local value, flags = cats.get(cats, "Marry")
```

or use Lua's syntactic sugar for method calls:

```lua

 local cats = ngx.shared.cats
 local value, flags = cats:get("Marry")
```

These two forms are fundamentally equivalent.

If the user flags is `0` (the default), then no flags value will be returned.

This feature was first introduced in the `v0.3.1rc22` release.

See also [ngx.shared.DICT](#ngxshareddict).

[返回目录](#nginx-api-for-lua)