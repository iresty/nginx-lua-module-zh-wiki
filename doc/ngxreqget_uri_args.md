ngx.req.get_uri_args
--------------------
**语法:** *args = ngx.req.get_uri_args(max_args?)*

**环境:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua, log_by_lua*\*

返回一个 Lua table，包含当前请求的所有 URL 查询参数。

```nginx

 location = /test {
     content_by_lua '
         local args = ngx.req.get_uri_args()
         for key, val in pairs(args) do
             if type(val) == "table" then
                 ngx.say(key, ": ", table.concat(val, ", "))
             else
                 ngx.say(key, ": ", val)
             end
         end
     ';
 }
```

访问 `GET /test?foo=bar&bar=baz&bar=blah` 将输出：

```bash

 foo: bar
 bar: baz, blah
```

多次出现同一个参数 key 时，将生成一个 Lua table，按顺序保存其所有 value。

key 和 value 将根据 URI 编码规则进行解码。访问上面的配置文件，`GET /test?a%20b=1%61+2` 将输出：

```bash

 a b: 1a 2
```

不包含 `=<value>` 部分的参数被视为布尔值参数。`GET /test?foo&bar` 输出：

```bash

 foo: true
 bar: true
```

换句话说，它们将被赋值 Lua 布尔值 `true`。但是，它们与空字符串值参数不同，如 `GET /test?foo=&bar=` 将输出：

```bash

 foo:
 bar:
```

没有 key 的参数将被忽略。例如 `GET /test?=hello&=world` 将没有任何输出。

支持通过 nginx 变量 `$args` (或在 Lua 中访问 `ngx.var.args`) 动态更新查询参数：

```lua

 ngx.var.args = "a=3&b=42"
 local args = ngx.req.get_uri_args()
```

这个例子里的 `args` table 将一直是：

```lua

 {a = 3, b = 42}
```

而无论实际请求查询串是什么内容。

请注意，为防止拒绝服务式攻击 (denial of service attacks)，默认最多解析前 100 个请求参数 (包括同名的)，更多的参数将直接忽略。

可选的 `max_args` 函数参数可以用来修改这个限制：

```lua

 local args = ngx.req.get_uri_args(10)
```

这个参数可以被设置为 0 以移除此限制，此时将解析所有接收到的请求参数：

```lua

 local args = ngx.req.get_uri_args(0)
```

强烈不推荐移除 `max_args` 限制。


> English Source

**syntax:** *args = ngx.req.get_uri_args(max_args?)*

**context:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua, log_by_lua*\*

Returns a Lua table holding all the current request URL query arguments.

```nginx

 location = /test {
     content_by_lua '
         local args = ngx.req.get_uri_args()
         for key, val in pairs(args) do
             if type(val) == "table" then
                 ngx.say(key, ": ", table.concat(val, ", "))
             else
                 ngx.say(key, ": ", val)
             end
         end
     ';
 }
```

Then `GET /test?foo=bar&bar=baz&bar=blah` will yield the response body

```bash

 foo: bar
 bar: baz, blah
```

Multiple occurrences of an argument key will result in a table value holding all the values for that key in order.

Keys and values are unescaped according to URI escaping rules. In the settings above, `GET /test?a%20b=1%61+2` will yield:

```bash

 a b: 1a 2
```

Arguments without the `=<value>` parts are treated as boolean arguments. `GET /test?foo&bar` will yield:

```bash

 foo: true
 bar: true
```

That is, they will take Lua boolean values `true`. However, they are different from arguments taking empty string values. `GET /test?foo=&bar=` will give something like

```bash

 foo:
 bar:
```

Empty key arguments are discarded. `GET /test?=hello&=world` will yield an empty output for instance.

Updating query arguments via the nginx variable `$args` (or `ngx.var.args` in Lua) at runtime is also supported:

```lua

 ngx.var.args = "a=3&b=42"
 local args = ngx.req.get_uri_args()
```

Here the `args` table will always look like

```lua

 {a = 3, b = 42}
```

regardless of the actual request query string.

Note that a maximum of 100 request arguments are parsed by default (including those with the same name) and that additional request arguments are silently discarded to guard against potential denial of service attacks.

However, the optional `max_args` function argument can be used to override this limit:

```lua

 local args = ngx.req.get_uri_args(10)
```

This argument can be set to zero to remove the limit and to process all request arguments received:

```lua

 local args = ngx.req.get_uri_args(0)
```

Removing the `max_args` cap is strongly discouraged.

[返回目录](#nginx-api-for-lua)
