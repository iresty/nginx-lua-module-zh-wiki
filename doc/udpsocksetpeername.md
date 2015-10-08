
udpsock:setpeername
-------------------
**语法:** *ok, err = udpsock:setpeername(host, port)*

**语法:** *ok, err = udpsock:setpeername("unix:/path/to/unix-domain.socket")*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

尝试对远端服务或 unix 域数据报 socket 文件建立 UDP socket 对象。因为数据报协议实际上 *连接少* ，该方法并没有真正建立一条连接，但为了后续读/写操作只是设置了远程端点的名称。

对 `host` 参数 IP 地址和域名在这里都是可以使用的。当使用域名时，该方法将使用 Nginx 内部的动态域名解析器（非阻塞并且需要在 `nginx.conf` 文件中配置 [resolver](http://nginx.org/en/docs/http/ngx_http_core_module.html#resolver) 指令），例如：

```nginx

 resolver 8.8.8.8;  # use Google's public DNS nameserver
```

如果域名服务器对这个主机名返回多个 IP 地址，该方法将从中随机挑选一个。

失败情况下，该方法返回 `nil` 和 错误字符描述信息。成功情况下，该方法返回 `1` 。

这里是个连接到 UDP （memcached） 服务的示例：

```nginx

 location /test {
     resolver 8.8.8.8;

     content_by_lua '
         local sock = ngx.socket.udp()
         local ok, err = sock:setpeername("my.memcached.server.domain", 11211)
         if not ok then
             ngx.say("failed to connect to memcached: ", err)
             return
         end
         ngx.say("successfully connected to memcached!")
         sock:close()
     ';
 }
```

自 `v0.7.18` 版本以来，在 Linux 平台连接到 unix 域数据报 socket 文件也是可能的：

```lua

 local sock = ngx.socket.udp()
 local ok, err = sock:setpeername("unix:/tmp/some-datagram-service.sock")
 if not ok then
     ngx.say("failed to connect to the datagram unix domain socket: ", err)
     return
 end
```

假设数据报服务在 unix domain socket 文件 `/tmp/some-datagram-service.sock` 上监听，并且客户端 socket 将在 Linux 上使用 "autobind" 。

对已经连接的 socket 对象调用该方法将导致原始连接将先被关闭。

该特性是在 `v0.5.7` 版本首次引入的。

[返回目录](#nginx-api-for-lua)

> English source:

udpsock:setpeername
-------------------
**syntax:** *ok, err = udpsock:setpeername(host, port)*

**syntax:** *ok, err = udpsock:setpeername("unix:/path/to/unix-domain.socket")*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Attempts to connect a UDP socket object to a remote server or to a datagram unix domain socket file. Because the datagram protocol is actually connection-less, this method does not really establish a "connection", but only just set the name of the remote peer for subsequent read/write operations.

Both IP addresses and domain names can be specified as the `host` argument. In case of domain names, this method will use Nginx core's dynamic resolver to parse the domain name without blocking and it is required to configure the [resolver](http://nginx.org/en/docs/http/ngx_http_core_module.html#resolver) directive in the `nginx.conf` file like this:

```nginx

 resolver 8.8.8.8;  # use Google's public DNS nameserver
```

If the nameserver returns multiple IP addresses for the host name, this method will pick up one randomly.

In case of error, the method returns `nil` followed by a string describing the error. In case of success, the method returns `1`.

Here is an example for connecting to a UDP (memcached) server:

```nginx

 location /test {
     resolver 8.8.8.8;

     content_by_lua '
         local sock = ngx.socket.udp()
         local ok, err = sock:setpeername("my.memcached.server.domain", 11211)
         if not ok then
             ngx.say("failed to connect to memcached: ", err)
             return
         end
         ngx.say("successfully connected to memcached!")
         sock:close()
     ';
 }
```

Since the `v0.7.18` release, connecting to a datagram unix domain socket file is also possible on Linux:

```lua

 local sock = ngx.socket.udp()
 local ok, err = sock:setpeername("unix:/tmp/some-datagram-service.sock")
 if not ok then
     ngx.say("failed to connect to the datagram unix domain socket: ", err)
     return
 end
```

assuming the datagram service is listening on the unix domain socket file `/tmp/some-datagram-service.sock` and the client socket will use the "autobind" feature on Linux.

Calling this method on an already connected socket object will cause the original connection to be closed first.

This method was first introduced in the `v0.5.7` release.

[Back to TOC](#nginx-api-for-lua)

