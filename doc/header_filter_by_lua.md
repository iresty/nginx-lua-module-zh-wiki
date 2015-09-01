header_filter_by_lua
---------------

**语法:** *header_filter_by_lua &lt;lua-script-str&gt;*

**环境:** *http, server, location, location if*

**nginx执行阶段:** *output-header-filter*

用`<lua-script-str>`中的lua代码，来设置应答消息的头部信息。

注意，下列的接口函数在这个执行阶段是失效的：

- 输出类函数（例：ngx.say和ngx.send_headers）

- 控制类函数(例：ngx.redirect和ngx.exec)

- 子请求相关函数 （例：ngx.location.capture和ngx.location.capture_multi）

- 套接字类函数（例：ngx.socket.tcp和ngx.req.socket）

下面这个例子展示的是，在header_filter_by_lua阶段中,会将应答消息头部中的某个的字段覆盖掉（如果没有这个字段，则是将其添加进头部）

```nginx

 location / {
	 proxy_pass http://mybackend;
	 header_filter_by_lua 'ngx.header.Foo = "blah"';
 }

```

这个指令最早出现在版本 `v0.2.1rc20 中。

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
```nginx

This directive was first introduced in the v0.2.1rc20 release.

[BACK TO TOC](#directives)
