
tcpsock:connect
---------------
**语法:** *ok, err = tcpsock:connect(host, port, options_table?)*

**语法:** *ok, err = tcpsock:connect("unix:/path/to/unix-domain.socket", options_table?)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

尝试以非阻塞的方式，对远端服务或 unix domain socket 文件建立 TCP socket 对象。

在真正解析主机名并连接到后端服务之前，该方法将永远优先在连接池内（都是调用该方法或 [ngx.socket.connect](#ngxsocketconnect) 函数的连接）查找符合条件的空闲连接。

对 `host` 参数 IP 地址和域名在这里都是可以使用的。当使用域名时，该方法将使用 Nginx 内部的动态域名解析器（非阻塞并且需要在 `nginx.conf` 文件中配置 [resolver](http://nginx.org/en/docs/http/ngx_http_core_module.html#resolver) 指令），例如：

```nginx

 resolver 8.8.8.8;  # 使用 Google 的公用域名解析服务器
```

如果域名服务器对这个主机名返回多个 IP 地址，该方法将从中随机挑选一个。

错误情况下，该方法返回 `nil` 及错误描述信息。成功情况下，该方法返回 `1` 。

这里是个连接到 TCP 服务的示例：

```nginx

 location /test {
     resolver 8.8.8.8;

     content_by_lua '
         local sock = ngx.socket.tcp()
         local ok, err = sock:connect("www.google.com", 80)
         if not ok then
             ngx.say("failed to connect to google: ", err)
             return
         end
         ngx.say("successfully connected to google!")
         sock:close()
     ';
 }
```

连接到 Unix Domain Socket 文件也是可能的：

```lua

 local sock = ngx.socket.tcp()
 local ok, err = sock:connect("unix:/tmp/memcached.sock")
 if not ok then
     ngx.say("failed to connect to the memcached unix domain socket: ", err)
     return
 end
```


假设 memcached （或其他服务）正在 Unix Domain Socket 文件 `/tmp/memcached.sock` 监听。

连接操作超时控制，是由 [lua_socket_connect_timeout](#lua_socket_connect_timeout) 配置指令和 [settimeout](#tcpsocksettimeout) 方法设置的。而后者有更高的优先级，例如：

```lua

 local sock = ngx.socket.tcp()
 sock:settimeout(1000)  -- one second timeout
 local ok, err = sock:connect(host, port)
```

调用这个方法 *之前* 调用 [settimeout](#tcpsocksettimeout) 方法设置超时时间，是非常重要的。

对已经连接状态的 socket 对象再次调用该方法，将导致原本的连接首先被关闭。

对于该方法的最后一个参数是个可选的 Lua 表，用来指定各种连接选项：

* `pool`
    对即将被使用的连接池指定一个名字。如果没有指定该参数，连接池的名字将自动生成，使用 `"<host>:<port>"` 或 `"<unix-socket-path>"` 的命名方式。

支持可选的表参数，是在 `v0.5.7` 版本首次引入。

该特性在 `v0.5.0rc1` 版本首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

tcpsock:connect
---------------
**syntax:** *ok, err = tcpsock:connect(host, port, options_table?)*

**syntax:** *ok, err = tcpsock:connect("unix:/path/to/unix-domain.socket", options_table?)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Attempts to connect a TCP socket object to a remote server or to a stream unix domain socket file without blocking.

Before actually resolving the host name and connecting to the remote backend, this method will always look up the connection pool for matched idle connections created by previous calls of this method (or the [ngx.socket.connect](#ngxsocketconnect) function).

Both IP addresses and domain names can be specified as the `host` argument. In case of domain names, this method will use Nginx core's dynamic resolver to parse the domain name without blocking and it is required to configure the [resolver](http://nginx.org/en/docs/http/ngx_http_core_module.html#resolver) directive in the `nginx.conf` file like this:

```nginx

 resolver 8.8.8.8;  # use Google's public DNS nameserver
```

If the nameserver returns multiple IP addresses for the host name, this method will pick up one randomly.

In case of error, the method returns `nil` followed by a string describing the error. In case of success, the method returns `1`.

Here is an example for connecting to a TCP server:

```nginx

 location /test {
     resolver 8.8.8.8;

     content_by_lua '
         local sock = ngx.socket.tcp()
         local ok, err = sock:connect("www.google.com", 80)
         if not ok then
             ngx.say("failed to connect to google: ", err)
             return
         end
         ngx.say("successfully connected to google!")
         sock:close()
     ';
 }
```

Connecting to a Unix Domain Socket file is also possible:

```lua

 local sock = ngx.socket.tcp()
 local ok, err = sock:connect("unix:/tmp/memcached.sock")
 if not ok then
     ngx.say("failed to connect to the memcached unix domain socket: ", err)
     return
 end
```

assuming memcached (or something else) is listening on the unix domain socket file `/tmp/memcached.sock`.

Timeout for the connecting operation is controlled by the [lua_socket_connect_timeout](#lua_socket_connect_timeout) config directive and the [settimeout](#tcpsocksettimeout) method. And the latter takes priority. For example:

```lua

 local sock = ngx.socket.tcp()
 sock:settimeout(1000)  -- one second timeout
 local ok, err = sock:connect(host, port)
```

It is important here to call the [settimeout](#tcpsocksettimeout) method *before* calling this method.

Calling this method on an already connected socket object will cause the original connection to be closed first.

An optional Lua table can be specified as the last argument to this method to specify various connect options:

* `pool`
    specify a custom name for the connection pool being used. If omitted, then the connection pool name will be generated from the string template `"<host>:<port>"` or `"<unix-socket-path>"`.

The support for the options table argument was first introduced in the `v0.5.7` release.

This method was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

