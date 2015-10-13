ngx.redirect
------------
**语法:** *ngx.redirect(uri, status?)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

发出一个 HTTP `301` 或 `302` 重定向到 `uri`。

可选项 `status` 参数指定 `301` 或 `302` 哪个被使用。 默认使用 `302` （`ngx.HTTP_MOVED_TEMPORARILY`）。

假设当前服务名是 `localhost` 并且监听端口是 1984，这里有个例子：

```lua

 return ngx.redirect("/foo")
```

和下面例子是等价的：

```lua

 return ngx.redirect("/foo", ngx.HTTP_MOVED_TEMPORARILY)
```

重定向到任意外部 URL 也是支持的，例如：

```lua

 return ngx.redirect("http://www.google.com")
```

我们也可以使用数值类型的值当做第二个 `status` 参数：

```lua

 return ngx.redirect("/foo", 301)
```

该方法与使用 `redirect` 修饰的 [rewrite](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite) 标准指令 [ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html) 有些相似，例如，该 `nginx.conf` 片段：

```nginx

 rewrite ^ /foo? redirect;  # nginx config
```

与下面 Lua 代码是等价的：

```lua

 return ngx.redirect('/foo');  -- Lua code
```

当这样

```nginx

 rewrite ^ /foo? permanent;  # nginx config
```

等同于：

```lua

 return ngx.redirect('/foo', ngx.HTTP_MOVED_PERMANENTLY)  -- Lua code
```

也可以指定具体 URI 参数，例如：、

```lua

 return ngx.redirect('/foo?a=3&b=4')
```

注意，此方法的调用终止当前请求的处理，并且它 *必须* 在 [ngx.send_headers](#ngxsend_headers) 或明确有响应体应答（比如 [ngx.print](#ngxprint) 或 [ngx.say](#ngxsay) 之一）之前调用。

该方法调用与 `return` 语句联合使用，是推荐的代码样式，例如，通过`return ngx.redirect(...)`，可使用在 [header_filter_by_lua](#header_filter_by_lua) 之外的环境中加强处理该请求被终止。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.redirect
------------
**syntax:** *ngx.redirect(uri, status?)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

Issue an `HTTP 301` or `302` redirection to `uri`.

The optional `status` parameter specifies whether
`301` or `302` to be used. It is `302` (`ngx.HTTP_MOVED_TEMPORARILY`) by default.

Here is an example assuming the current server name is `localhost` and that it is listening on port 1984:

```lua

 return ngx.redirect("/foo")
```

which is equivalent to

```lua

 return ngx.redirect("/foo", ngx.HTTP_MOVED_TEMPORARILY)
```

Redirecting arbitrary external URLs is also supported, for example:

```lua

 return ngx.redirect("http://www.google.com")
```

We can also use the numerical code directly as the second `status` argument:

```lua

 return ngx.redirect("/foo", 301)
```

This method is similar to the [rewrite](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite) directive with the `redirect` modifier in the standard
[ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html), for example, this `nginx.conf` snippet

```nginx

 rewrite ^ /foo? redirect;  # nginx config
```

is equivalent to the following Lua code

```lua

 return ngx.redirect('/foo');  -- Lua code
```

while

```nginx

 rewrite ^ /foo? permanent;  # nginx config
```

is equivalent to

```lua

 return ngx.redirect('/foo', ngx.HTTP_MOVED_PERMANENTLY)  -- Lua code
```

URI arguments can be specified as well, for example:

```lua

 return ngx.redirect('/foo?a=3&b=4')
```

Note that this method call terminates the processing of the current request and that it *must* be called before [ngx.send_headers](#ngxsend_headers) or explicit response body
outputs by either [ngx.print](#ngxprint) or [ngx.say](#ngxsay).

It is recommended that a coding style that combines this method call with the `return` statement, i.e., `return ngx.redirect(...)` be adopted when this method call is used in contexts other than [header_filter_by_lua](#header_filter_by_lua) to reinforce the fact that the request processing is being terminated.

[Back to TOC](#nginx-api-for-lua)
