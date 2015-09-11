ngx.req.get_headers
-------------------
**语法:** *headers = ngx.req.get_headers(max_headers?, raw?)*

**环境:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua, log_by_lua*\*

返回一个 Lua table，包含当前请求的所有请求头信息。

```lua

 local h = ngx.req.get_headers()
 for k, v in pairs(h) do
     ...
 end
```

读某一个头信息：

```lua

 ngx.say("Host: ", ngx.req.get_headers()["Host"])
```

请注意，[ngx.var.HEADER](#ngxvarvariable) API 使用 nginx 内核 [$http_HEADER](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_http) 变量，在读取单个请求头信息时更加适用。

如果请求头中多次出现某一字段，例如：

```bash

 Foo: foo
 Foo: bar
 Foo: baz
```

`ngx.req.get_headers()["Foo"]` 的值会是一个 Lua (数组) table，类似：

```lua

 {"foo", "bar", "baz"}
```

请注意，为防止拒绝服务式攻击 (denial of service attacks)，默认最多解析前 100 个请求头信息 (包括同名的)，更多的头信息将直接忽略。

可选的 `max_headers` 函数参数可以用来修改这个限制：

```lua

 local headers = ngx.req.get_headers(10)
```

这个参数可以被设置为 0 以移除此限制，此时将解析所有接收到的请求头信息：

```lua

 local headers = ngx.req.get_headers(0)
```

强烈不推荐移除 `max_headers` 限制。

自 `0.6.9` 版本开始，默认情况下，返回的 Lua table 中的所有头名称被转换成纯小写字母形式，除非设置 `raw` 参数为 `true` (默认是 `false`)。

同时，默认情况下，Lua table 中将被加入 `__index` 元方法 ([metamethod](http://www.lua.org/pil/13.4.1.html))， 用来在查询失败时，将 key 转换为全小写字母、且下划线变为连字符后，重新搜索。例如，如果请求头信息中有 `My-Foo-Header` 字段，下面的调用都将正确取出值：

```lua

 ngx.say(headers.my_foo_header)
 ngx.say(headers["My-Foo-Header"])
 ngx.say(headers["my-foo-header"])
```

当 `raw` 参数设置为 `true` 时，`__index` 元方法不会被加入。


> English Source

**syntax:** *headers = ngx.req.get_headers(max_headers?, raw?)*

**context:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua, log_by_lua*\*

Returns a Lua table holding all the current request headers.

```lua

 local h = ngx.req.get_headers()
 for k, v in pairs(h) do
     ...
 end
```

To read an individual header:

```lua

 ngx.say("Host: ", ngx.req.get_headers()["Host"])
```

Note that the [ngx.var.HEADER](#ngxvarvariable) API call, which uses core [$http_HEADER](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_http_) variables, may be more preferable for reading individual request headers.

For multiple instances of request headers such as:

```bash

 Foo: foo
 Foo: bar
 Foo: baz
```

the value of `ngx.req.get_headers()["Foo"]` will be a Lua (array) table such as:

```lua

 {"foo", "bar", "baz"}
```

Note that a maximum of 100 request headers are parsed by default (including those with the same name) and that additional request headers are silently discarded to guard against potential denial of service attacks.  

However, the optional `max_headers` function argument can be used to override this limit:

```lua

 local headers = ngx.req.get_headers(10)
```

This argument can be set to zero to remove the limit and to process all request headers received:

```lua

 local headers = ngx.req.get_headers(0)
```

Removing the `max_headers` cap is strongly discouraged.

Since the `0.6.9` release, all the header names in the Lua table returned are converted to the pure lower-case form by default, unless the `raw` argument is set to `true` (default to `false`).

Also, by default, an `__index` metamethod is added to the resulting Lua table and will normalize the keys to a pure lowercase form with all underscores converted to dashes in case of a lookup miss. For example, if a request header `My-Foo-Header` is present, then the following invocations will all pick up the value of this header correctly:

```lua

 ngx.say(headers.my_foo_header)
 ngx.say(headers["My-Foo-Header"])
 ngx.say(headers["my-foo-header"])
```

The `__index` metamethod will not be added when the `raw` argument is set to `true`.

[返回目录](#nginx-api-for-lua)
