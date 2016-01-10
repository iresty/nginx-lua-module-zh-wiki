ngx.header.HEADER
-----------------
**语法:** *ngx.header.HEADER = VALUE*

**语法:** *value = ngx.header.HEADER*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;*

修改、添加、或清除当前请求待发送的 `HEADER` 响应头信息。

头名称中的下划线 (`_`) 将被默认替换为连字符 (`-`)。可以通过 [lua_transform_underscores_in_response_headers](#lua_transform_underscores_in_response_headers) 指令关闭这个替换。

头名称大小写不敏感。

```lua

 -- 与 ngx.header["Content-Type"] = 'text/plain' 相同
 ngx.header.content_type = 'text/plain';

 ngx.header["X-My-Header"] = 'blah blah';
```

多值头信息可以按以下方法设置：

```lua

 ngx.header['Set-Cookie'] = {'a=32; path=/', 'b=4; path=/'}
```

在响应头中将输出：

```bash

 Set-Cookie: a=32; path=/
 Set-Cookie: b=4; path=/
```

只接受 Lua 表输入 (对于类似 `Content-Type` 之类的只接受一个值的标准头信息，只有 Lua 表中的最后一个元素有效)。

```lua

 ngx.header.content_type = {'a', 'b'}
```

等同于

```lua

 ngx.header.content_type = 'b'
```

将一个头信息的值设为 `nil` 将从响应头中移除该输出：

```lua

 ngx.header["X-My-Header"] = nil;
```

赋值一个空 Lua 表效果相同：

```lua

 ngx.header["X-My-Header"] = {};
```

在输出响应头 (不管是显示的通过 [ngx.send_headers](#ngxsend_headers) 或隐式的通过类似 [ngx.print](#ngxprint) 指令) 以后设置 `ngx.header.HEADER` 将抛出 Lua 异常。

读取 `ngx.header.HEADER` 将返回响应头中名为 `HEADER` 的头信息的值。

读取时，头名称中的下划线 (`_`) 也会被替换成连字符 (`-`)，并且大小写不敏感。如果该头信息不存在，将返回 `nil`。

这个 API 在 [header_filter_by_lua](#header_filter_by_lua) 和 [header_filter_by_lua_file](#header_filter_by_lua_file) 环境中非常有用，例如：

```nginx

 location /test {
     set $footer '';

     proxy_pass http://some-backend;

     header_filter_by_lua '
         if ngx.header["X-My-Header"] == "blah" then
             ngx.var.footer = "some value"
         end
     ';

     echo_after_body $footer;
 }
```

对于多值头信息，所有值将被按顺序放入一个 Lua 表中，例如，响应头


    Foo: bar
    Foo: baz


在读取 `ngx.header.Foo` 时返回的结果为

```lua

 {"bar", "baz"}
```

需要注意的是，`ngx.header` 不是一个标准 Lua 表，不能通过 Lua 的 `ipairs` 函数进行迭代查询。

读取 *请求* 头信息，请使用 [ngx.req.get_headers](#ngxreqget_headers) 函数。


> English Source

**syntax:** *ngx.header.HEADER = VALUE*

**syntax:** *value = ngx.header.HEADER*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;*

Set, add to, or clear the current request's `HEADER` response header that is to be sent.

Underscores (`_`) in the header names will be replaced by hyphens (`-`) by default. This transformation can be turned off via the [lua_transform_underscores_in_response_headers](#lua_transform_underscores_in_response_headers) directive.

The header names are matched case-insensitively.

```lua

 -- equivalent to ngx.header["Content-Type"] = 'text/plain'
 ngx.header.content_type = 'text/plain';

 ngx.header["X-My-Header"] = 'blah blah';
```

Multi-value headers can be set this way:

```lua

 ngx.header['Set-Cookie'] = {'a=32; path=/', 'b=4; path=/'}
```

will yield

```bash

 Set-Cookie: a=32; path=/
 Set-Cookie: b=4; path=/
```

in the response headers. 

Only Lua tables are accepted (Only the last element in the table will take effect for standard headers such as `Content-Type` that only accept a single value).

```lua

 ngx.header.content_type = {'a', 'b'}
```

is equivalent to

```lua

 ngx.header.content_type = 'b'
```

Setting a slot to `nil` effectively removes it from the response headers:

```lua

 ngx.header["X-My-Header"] = nil;
```

The same applies to assigning an empty table:

```lua

 ngx.header["X-My-Header"] = {};
```

Setting `ngx.header.HEADER` after sending out response headers (either explicitly with [ngx.send_headers](#ngxsend_headers) or implicitly with [ngx.print](#ngxprint) and similar) will throw out a Lua exception.

Reading `ngx.header.HEADER` will return the value of the response header named `HEADER`. 

Underscores (`_`) in the header names will also be replaced by dashes (`-`) and the header names will be matched case-insensitively. If the response header is not present at all, `nil` will be returned.

This is particularly useful in the context of [header_filter_by_lua](#header_filter_by_lua) and [header_filter_by_lua_file](#header_filter_by_lua_file), for example,

```nginx

 location /test {
     set $footer '';

     proxy_pass http://some-backend;

     header_filter_by_lua '
         if ngx.header["X-My-Header"] == "blah" then
             ngx.var.footer = "some value"
         end
     ';

     echo_after_body $footer;
 }
```

For multi-value headers, all of the values of header will be collected in order and returned as a Lua table. For example, response headers


    Foo: bar
    Foo: baz


will result in

```lua

 {"bar", "baz"}
```

to be returned when reading `ngx.header.Foo`.

Note that `ngx.header` is not a normal Lua table and as such, it is not possible to iterate through it using the Lua `ipairs` function.

For reading *request* headers, use the [ngx.req.get_headers](#ngxreqget_headers) function instead.

[返回目录](#nginx-api-for-lua)