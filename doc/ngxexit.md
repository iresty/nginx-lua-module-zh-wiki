ngx.exit
--------
**语法:** *ngx.exit(status)*

**环境:** *rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, ngx.timer.*\*

当 `status >= 200` (即 `ngx.HTTP_OK` 及以上) 时，本函数中断当前请求执行并返回状态值给 nginx。

当 `status == 0` (即 `ngx.OK`) 时，本函数退出当前的“处理阶段句柄” (或当使用 [content_by_lua](#content_by_lua) 指令时的“内容句柄”) ，继续执行当前请求的下一个阶段 (如果有)。

`status` 参数可以是 `status` argument can be `ngx.OK`, `ngx.ERROR`, `ngx.HTTP_NOT_FOUND`, `ngx.HTTP_MOVED_TEMPORARILY` 或其它 [HTTP status constants](#http-status-constants)。

要返回一个自定义内容的错误页，使用类似下面的代码：

```lua

 ngx.status = ngx.HTTP_GONE
 ngx.say("This is our own content")
 -- 退出整个请求而不是当前处理阶段
 ngx.exit(ngx.HTTP_OK)
```

实际效果:

```bash

 $ curl -i http://localhost/test
 HTTP/1.1 410 Gone
 Server: nginx/1.0.6
 Date: Thu, 15 Sep 2011 00:51:48 GMT
 Content-Type: text/plain
 Transfer-Encoding: chunked
 Connection: keep-alive

 This is our own content
```

可以直接使用数字作为参数，例如：

```lua

 ngx.exit(501)
```

请注意，虽然此方法接受所有 [HTTP status constants](#http-status-constants) 作为输入，但在 [core constants](#core-constants)  中仅支持 `NGX_OK` 和 `NGX_ERROR` 作为输入。

同时请注意，因为调用此方法会中断当前请求处理，所以建议代码风格是与 `return` 语句一起调用此方法，即，使用 `return ngx.exit(...)` 来强调请求处理过程已经中断。

当使用在 [header_filter_by_lua](#header_filter_by_lua) 环境中时，`ngx.exit()` 是一个异步操作，会立即返回。这个行为在未来版本中可能会改变，所以建议用户一直使用上述与 `returen` 同时使用的代码风格。

> English Source

**syntax:** *ngx.exit(status)*

**context:** *rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, ngx.timer.*\*

When `status >= 200` (i.e., `ngx.HTTP_OK` and above), it will interrupt the execution of the current request and return status code to nginx.

When `status == 0` (i.e., `ngx.OK`), it will only quit the current phase handler (or the content handler if the [content_by_lua](#content_by_lua) directive is used) and continue to run later phases (if any) for the current request.

The `status` argument can be `ngx.OK`, `ngx.ERROR`, `ngx.HTTP_NOT_FOUND`,
`ngx.HTTP_MOVED_TEMPORARILY`, or other [HTTP status constants](#http-status-constants).

To return an error page with custom contents, use code snippets like this:

```lua

 ngx.status = ngx.HTTP_GONE
 ngx.say("This is our own content")
 -- to cause quit the whole request rather than the current phase handler
 ngx.exit(ngx.HTTP_OK)
```

The effect in action:

```bash

 $ curl -i http://localhost/test
 HTTP/1.1 410 Gone
 Server: nginx/1.0.6
 Date: Thu, 15 Sep 2011 00:51:48 GMT
 Content-Type: text/plain
 Transfer-Encoding: chunked
 Connection: keep-alive

 This is our own content
```

Number literals can be used directly as the argument, for instance,

```lua

 ngx.exit(501)
```

Note that while this method accepts all [HTTP status constants](#http-status-constants) as input, it only accepts `NGX_OK` and `NGX_ERROR` of the [core constants](#core-constants).

Also note that this method call terminates the processing of the current request and that it is recommended that a coding style that combines this method call with the `return` statement, i.e., `return ngx.exit(...)` be used to reinforce the fact that the request processing is being terminated.

When being used in the context of [header_filter_by_lua](#header_filter_by_lua), `ngx.exit()` is an asynchronous operation and will return immediately. This behavior may change in future and it is recommended that users always use `return` in combination as suggested above.


[返回目录](#nginx-api-for-lua)