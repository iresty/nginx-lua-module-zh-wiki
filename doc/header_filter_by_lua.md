header_filter_by_lua
---------------

**语法：** *header_filter_by_lua &lt;lua-script-str&gt;*

**环境：** *http, server, location, location if*

**阶段：** *output-header-filter*

用`<lua-script-str>`中指名的lua代码，来完成应答消息头部的过滤。

注意，下列的接口函数在这个执行阶段是被禁用的：

- 输出类函数（例：ngx.say 和 ngx.send_headers）

- 控制类函数（例：ngx.redirect 和 ngx.exec）

- 子请求相关函数（例：ngx.location.capture和ngx.location.capture_multi）

- 异步套接字类函数（例：ngx.socket.tcp 和 ngx.req.socket）

这里有个使用 Lua 过滤完成覆盖应答头的例子（如果没有则添加）：

```nginx

 location / {
	 proxy_pass http://mybackend;
	 header_filter_by_lua 'ngx.header.Foo = "blah"';
 }

```

该指令在版本 `v0.2.1rc20` 中第一次引入。

[返回目录](#directives)


> English Source

**syntax:** *header_filter_by_lua &lt;lua-script-str&gt;*

**context:** *http, server, location, location if*

**phase:** *output-header-filter*

Uses Lua code specified in `<lua-script-str>` to define an output header filter.

Note that the following API functions are currently disabled within this context:

- Output API functions (e.g., ngx.say and ngx.send_headers)

- Control API functions (e.g., ngx.redirect and ngx.exec)

- Subrequest API functions (e.g., ngx.location.capture and ngx.location.capture_multi)

- Cosocket API functions (e.g., ngx.socket.tcp and ngx.req.socket).

Here is an example of overriding a response header (or adding one if absent) in our Lua header filter:

```nginx

    location / {
        proxy_pass http://mybackend;
        header_filter_by_lua 'ngx.header.Foo = "blah"';
    }

```

This directive was first introduced in the `v0.2.1rc20` release.

[BACK TO TOC](#directives)
