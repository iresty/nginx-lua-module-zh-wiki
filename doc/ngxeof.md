ngx.eof
-------
**语法:** *ok, err = ngx.eof()*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

明确指定响应输出流的末尾。在 HTTP 1.1 分块编码输出模式下，它会触发 Nginx 内核发送“最后一块”。

当禁用下游连接的 HTTP 1.1 保持连接功能后，用户程序可以通过调用此方法，使下游 HTTP 客户端主动关闭连接。这招可以用来执行后台任务，而无需 HTTP 客户端等待连接关闭，例如：

```nginx

 location = /async {
     keepalive_timeout 0;
     content_by_lua '
         ngx.say("got the task!")
         ngx.eof()  -- 下游 HTTP 客户端将在这里断开连接
         -- 在这里访问 MySQL, PostgreSQL, Redis, Memcached 等 ...
     ';
 }
```

但是，如果用户程序创建子请求通过 Nginx 上游模块访问其他 location 时，需要配置上游模块忽略客户端连接中断 (如果不是默认)。例如，默认时，基本模块 [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) 在客户端关闭连接后，立刻中断主请求和子请求，所以在 [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) 配置的 location 块中打开 [proxy_ignore_client_abort](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ignore_client_abort) 开关非常重要：

```nginx

 proxy_ignore_client_abort on;
```

一个执行后台任务的方法是使用 [ngx.timer.at](#ngxtimerat) API。

自版本 `v0.8.3`开始，此函数成功时返回 `1`，失败时返回 `nil` 和错误信息串。


> English Source

**syntax:** *ok, err = ngx.eof()*

**context:** **context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

Explicitly specify the end of the response output stream. In the case of HTTP 1.1 chunked encoded output, it will just trigger the Nginx core to send out the "last chunk".

When you disable the HTTP 1.1 keep-alive feature for your downstream connections, you can rely on descent HTTP clients to close the connection actively for you when you call this method. This trick can be used do back-ground jobs without letting the HTTP clients to wait on the connection, as in the following example:

```nginx

 location = /async {
     keepalive_timeout 0;
     content_by_lua '
         ngx.say("got the task!")
         ngx.eof()  -- a descent HTTP client will close the connection at this point
         -- access MySQL, PostgreSQL, Redis, Memcached, and etc here...
     ';
 }
```

But if you create subrequests to access other locations configured by Nginx upstream modules, then you should configure those upstream modules to ignore client connection abortions if they are not by default. For example, by default the standard [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) will terminate both the subrequest and the main request as soon as the client closes the connection, so it is important to turn on the [proxy_ignore_client_abort](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ignore_client_abort) directive in your location block configured by [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html):

```nginx

 proxy_ignore_client_abort on;
```

A better way to do background jobs is to use the [ngx.timer.at](#ngxtimerat) API.

Since `v0.8.3` this function returns `1` on success, or returns `nil` and a string describing the error otherwise.


[返回目录](#nginx-api-for-lua)