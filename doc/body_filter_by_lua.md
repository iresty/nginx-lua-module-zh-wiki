body_filter_by_lua
------------------

**语法:** *body_filter_by_lua &lt;lua-script-str&gt;*

**环境:** *http, server, location, location if*

**阶段:** *output-body-filter*

使用`<lua-script-str>`指定的 Lua 代码定义一个输出应答体过滤器。

输入数据块是 [ngx.arg](#ngxarg)\[1\]（Lua的字符串形式）,结束标识"eof"是应答体数据最后一位[ngx.arg](#ngxarg)\[2\]（Lua的布尔值形式）。

在这个场景下，结束标识"eof"仅仅是 Nginx chain 缓冲区的`last_buf`（主请求）或`last_in_chain`（子请求）。（在`v0.7.14`版本之前，结束标识"eof"在子请求中是完全不能使用的。）

使用下面 Lua 代码，可以对输出数据流立即终止：

```lua

 return ngx.ERROR
```

这样截断响应体，通常导致结果不完整的，也是无效的响应。

本指令的 Lua 代码可以使用 Lua 字符串或字符串的表重写[ngx.arg](#ngxarg)\[1\]输入数据块内容，从而完成 Nginx 输出体下游过滤数据修改。例如，在输出体转换所有的小写字母，我们可以这样用：

```nginx

 location / {
     proxy_pass http://mybackend;
     body_filter_by_lua 'ngx.arg[1] = string.upper(ngx.arg[1])';
 }
```

当设置`nil`或一个空的 Lua 字符串值给`ngx.arg[1]`，将没有任何数据块下发到 Nginx 下游。

同样，新的结束标识"eof"也可以通过对[ngx.arg](#ngxarg)\[2\]设定一个布尔值。例如：

```nginx

 location /t {
     echo hello world;
     echo hiya globe;

     body_filter_by_lua '
         local chunk = ngx.arg[1]
         if string.match(chunk, "hello") then
             ngx.arg[2] = true  -- new eof
             return
         end

         -- just throw away any remaining chunk data
         ngx.arg[1] = nil
     ';
 }
```

然后 `GET /t` 将返回下面的结果：

    hello world

就是说，当应答体过滤发现一个块包含关键字"hello"，它将立即设置结束标识"eof"为 true ，应答内容被截断尽管后面还有有效数据。

当Lua代码可能改变应答体的长度时，我们必须总是清空响应头中的`Content-Length`（如果有），强制使用流式输出，如：

```nginx

 location /foo {
     # fastcgi_pass/proxy_pass/...

     header_filter_by_lua 'ngx.header.content_length = nil';
     body_filter_by_lua 'ngx.arg[1] = string.len(ngx.arg[1]) .. "\\n"';
 }
```

注意：下面这些API函数在这个环境中是禁用的，这受制于当前 Nginx 输出过滤器的实现：

* 输出API函数类（例如：[ngx.say](#ngxsay) 和 [ngx.send_headers](#ngxsend_headers)）
* 控制API函数类（例如：[ngx.exit](#ngxexit) 和 [ngx.exec](#ngxexec)）
* 子请求函数类（例如：[ngx.location.capture](#ngxlocationcapture) 和 [ngx.location.capture_multi](#ngxlocationcapture_multi)）
* cosocket 函数类（例如：[ngx.socket.tcp](#ngxsockettcp) 和 [ngx.req.socket](#ngxreqsocket)）

Nginx 输出过滤器在一个单独请求中可能被调用多次，因为应答体可能使用块的方式进行投递。所以，本指令中的 Lua 代码在这个单独的HTTP请求生命周期内，同样会执行多次。

该指令在`v0.5.0rc32`版本中首次引入。

[返回目录](#directives)

> English source:

body_filter_by_lua
------------------

**syntax:** *body_filter_by_lua &lt;lua-script-str&gt;*

**context:** *http, server, location, location if*

**phase:** *output-body-filter*

Uses Lua code specified in `<lua-script-str>` to define an output body filter.

The input data chunk is passed via [ngx.arg](#ngxarg)\[1\] (as a Lua string value) and the "eof" flag indicating the end of the response body data stream is passed via [ngx.arg](#ngxarg)\[2\] (as a Lua boolean value).

Behind the scene, the "eof" flag is just the `last_buf` (for main requests) or `last_in_chain` (for subrequests) flag of the Nginx chain link buffers. (Before the `v0.7.14` release, the "eof" flag does not work at all in subrequests.)

The output data stream can be aborted immediately by running the following Lua statement:

```lua

 return ngx.ERROR
```

This will truncate the response body and usually result in incomplete and also invalid responses.

The Lua code can pass its own modified version of the input data chunk to the downstream Nginx output body filters by overriding [ngx.arg](#ngxarg)\[1\] with a Lua string or a Lua table of strings. For example, to transform all the lowercase letters in the response body, we can just write:

```nginx

 location / {
     proxy_pass http://mybackend;
     body_filter_by_lua 'ngx.arg[1] = string.upper(ngx.arg[1])';
 }
```

When setting `nil` or an empty Lua string value to `ngx.arg[1]`, no data chunk will be passed to the downstream Nginx output filters at all.

Likewise, new "eof" flag can also be specified by setting a boolean value to [ngx.arg](#ngxarg)\[2\]. For example,

```nginx

 location /t {
     echo hello world;
     echo hiya globe;

     body_filter_by_lua '
         local chunk = ngx.arg[1]
         if string.match(chunk, "hello") then
             ngx.arg[2] = true  -- new eof
             return
         end

         -- just throw away any remaining chunk data
         ngx.arg[1] = nil
     ';
 }
```

Then `GET /t` will just return the output


    hello world


That is, when the body filter sees a chunk containing the word "hello", then it will set the "eof" flag to true immediately, resulting in truncated but still valid responses.

When the Lua code may change the length of the response body, then it is required to always clear out the `Content-Length` response header (if any) in a header filter to enforce streaming output, as in

```nginx

 location /foo {
     # fastcgi_pass/proxy_pass/...

     header_filter_by_lua 'ngx.header.content_length = nil';
     body_filter_by_lua 'ngx.arg[1] = string.len(ngx.arg[1]) .. "\\n"';
 }
```

Note that the following API functions are currently disabled within this context due to the limitations in NGINX output filter's current implementation:

* Output API functions (e.g., [ngx.say](#ngxsay) and [ngx.send_headers](#ngxsend_headers))
* Control API functions (e.g., [ngx.exit](#ngxexit) and [ngx.exec](#ngxexec))
* Subrequest API functions (e.g., [ngx.location.capture](#ngxlocationcapture) and [ngx.location.capture_multi](#ngxlocationcapture_multi))
* Cosocket API functions (e.g., [ngx.socket.tcp](#ngxsockettcp) and [ngx.req.socket](#ngxreqsocket)).

Nginx output filters may be called multiple times for a single request because response body may be delivered in chunks. Thus, the Lua code specified by in this directive may also run multiple times in the lifetime of a single HTTP request.

This directive was first introduced in the `v0.5.0rc32` release.

[Back to TOC](#directives)
