ngx.req.raw_header
------------------
**语法:** *str = ngx.req.raw_header(no_request_line?)*

**环境:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\*

返回 Nginx 服务器接收到的原始 HTTP 协议头。

默认时，请求行和末尾的 `CR LF` 结束符也被包括在内。例如，

```lua

 ngx.print(ngx.req.raw_header())
```

输出结果类似：


    GET /t HTTP/1.1
    Host: localhost
    Connection: close
    Foo: bar



可以通过指定可选的 `no_request_line` 参数为 `true` 来去除结果中的请求行。例如，

```lua

 ngx.print(ngx.req.raw_header(true))
```

输出结果类似：


    Host: localhost
    Connection: close
    Foo: bar



这个方法在 `v0.7.17` 版本中首次引入。


> English Source

**syntax:** *str = ngx.req.raw_header(no_request_line?)*

**context:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\*

Returns the original raw HTTP protocol header received by the Nginx server.

By default, the request line and trailing `CR LF` terminator will also be included. For example,

```lua

 ngx.print(ngx.req.raw_header())
```

gives something like this:


    GET /t HTTP/1.1
    Host: localhost
    Connection: close
    Foo: bar



You can specify the optional
`no_request_line` argument as a `true` value to exclude the request line from the result. For example,

```lua

 ngx.print(ngx.req.raw_header(true))
```

outputs something like this:


    Host: localhost
    Connection: close
    Foo: bar



This method was first introduced in the `v0.7.17` release.

[返回目录](#nginx-api-for-lua)
