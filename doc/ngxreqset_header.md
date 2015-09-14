ngx.req.set_header
------------------
**语法:** *ngx.req.set_header(header_name, header_value)*

**环境:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\*

将当前请求的名为 `header_name` 的头信息值设置为 `header_value`，如果此头信息名称已经存在，修改其值。

默认时，之后通过 [ngx.location.capture](#ngxlocationcapture) 和 [ngx.location.capture_multi](#ngxlocationcapture_multi) 发起的所有子请求都将继承新的头信息。

下面的例子中将设置 `Content-Type` 头信息：

```lua

 ngx.req.set_header("Content-Type", "text/css")
```

`header_value` 可以是一个值数组，例如：

```lua

 ngx.req.set_header("Foo", {"a", "abc"})
```

将生成两个新的请求头信息：

```bash

 Foo: a
 Foo: abc
```

如果原来已经有名为 `Foo` 的头信息，将被覆盖。

当 `header_value` 参数是 `nil` 时，此请求头将被移除。所以，

```lua

 ngx.req.set_header("X-Foo", nil)
```

等同于

```lua

 ngx.req.clear_header("X-Foo")
```


> English Source

**syntax:** *ngx.req.set_header(header_name, header_value)*

**context:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\*

Set the current request's request header named `header_name` to value `header_value`, overriding any existing ones.

By default, all the subrequests subsequently initiated by [ngx.location.capture](#ngxlocationcapture) and [ngx.location.capture_multi](#ngxlocationcapture_multi) will inherit the new header.

Here is an example of setting the `Content-Type` header:

```lua

 ngx.req.set_header("Content-Type", "text/css")
```

The `header_value` can take an array list of values,
for example,

```lua

 ngx.req.set_header("Foo", {"a", "abc"})
```

will produce two new request headers:

```bash

 Foo: a
 Foo: abc
```

and old `Foo` headers will be overridden if there is any.

When the `header_value` argument is `nil`, the request header will be removed. So

```lua

 ngx.req.set_header("X-Foo", nil)
```

is equivalent to

```lua

 ngx.req.clear_header("X-Foo")
```

[返回目录](#nginx-api-for-lua)