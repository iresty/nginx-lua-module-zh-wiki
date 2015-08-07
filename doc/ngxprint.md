**语法:** *ok, err = ngx.print(...)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

将输入参数合并发送给 HTTP 客户端 (作为 HTTP 响应体)。如果此时还没有发送响应头信息，本函数将先发送 HTTP 响应头，再输出响应体。

自版本 `v0.8.3` 起，本函数当成功时返回 `1`，失败时返回 `nil` 以及一个描述错误的字符串。

Lua 的 `nil` 值输出 `"nil"` 字符串，Lua 的布尔值输出 `"true"` 或 `"false"` 字符串。

输入允许字符串嵌套数组，数组中所有元素相按顺序输出。

```lua

 local table = {
     "hello, ",
     {"world: ", true, " or ", false,
         {": ", nil}}
 }
 ngx.print(table)
```

将输出

```bash

 hello, world: true or false: nil
```

非数组表(哈希表)参数将导致抛出 Lua 异常。

`ngx.null` 常量输出为 `"null"` 字符串。

本函数为异步调用，将立即返回，不会等待所有数据被写入系统发送缓冲区。要以同步模式运行，请在调用 `ngx.print` 之后调用 `ngx.flush(true)`。这种方式在流式输出时非常有用。更多细节请参考 [ngx.flush](#ngxflush)。

请注意，`ngx.print` 和 [ngx.say](#ngxsay) 都会调用 Nginx body 输出过滤器，这种操作非常“昂贵”。所以，在“热”循环中使用这两个函数要非常小心；可以通过 Lua 进行缓存以节约调用。



> English Source

**syntax:** *ok, err = ngx.print(...)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

Emits arguments concatenated to the HTTP client (as response body). If response headers have not been sent, this function will send headers out first and then output body data.

Since `v0.8.3` this function returns `1` on success, or returns `nil` and a string describing the error otherwise.

Lua `nil` values will output `"nil"` strings and Lua boolean values will output `"true"` and `"false"` literal strings respectively.

Nested arrays of strings are permitted and the elements in the arrays will be sent one by one:

```lua

 local table = {
     "hello, ",
     {"world: ", true, " or ", false,
         {": ", nil}}
 }
 ngx.print(table)
```

will yield the output

```bash

 hello, world: true or false: nil
```

Non-array table arguments will cause a Lua exception to be thrown.

The `ngx.null` constant will yield the `"null"` string output.

This is an asynchronous call and will return immediately without waiting for all the data to be written into the system send buffer. To run in synchronous mode, call `ngx.flush(true)` after calling `ngx.print`. This can be particularly useful for streaming output. See [ngx.flush](#ngxflush) for more details.

Please note that both `ngx.print` and [ngx.say](#ngxsay) will always invoke the whole Nginx output body filter chain, which is an expensive operation. So be careful when calling either of these two in a tight loop; buffer the data yourself in Lua and save the calls.
