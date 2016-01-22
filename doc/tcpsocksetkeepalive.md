
tcpsock:setkeepalive
--------------------
**语法:** *ok, err = tcpsock:setkeepalive(timeout?, size?)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

把当前 socket 连接立即放到内建的 cosocket 连接池中，维持活动状态直到被其他 [connect](#tcpsockconnect) 方法调用请求，或者达到自身绑定的最大空闲时间后连接过期。

第一个可选参数参数 `timeout` ，可以用来指定当前连接的最大空闲时间（单位毫秒）。如果没指定该参数，将使用配置在 [lua_socket_keepalive_timeout](#lua_socket_keepalive_timeout) 指令的值作为默认值使用。如果给定的值是 `0` ，那么超时时间是没有限制的。

第二个参数 `size`，可以用来指定当前服务（例如，当前主机+端口配对或 unix socket 文件路径作为标识）在连接池中允许存放的最大连接数。注意，连接池大小一旦创建后，是不能被修改的。如果没指定该参数，将使用配置在 [lua_socket_pool_size](#lua_socket_pool_size) 指令的值作为默认值使用。

当连接池中连接数超过限制大小，在连接池中最近最少使用的（空闲）连接将被关闭，给当前连接腾挪空间。

注意，cosocket 连接池是每个 Nginx 工作进程使用的，而不是每个 Nginx 服务实例，所以这里指定的限制也只能在每个独立的 nginx 工作进程上生效。

连接池中空闲连接的任何异常事件都将会被监控，例如连接终止、在线收到非预期数据，在这种情况下有问题的连接，将被关闭并从池子中移除。

成功情况，该方法返回 `1`；否则，将返回 `nil` 和错误描述字符信息。

对于当前连接，当系统接收缓冲区有未读取完的数据，这时该方法将返回 "connection in dubious state" 的错误信息（作为第二个返回值），因为前一个请求留下了一些未读取数据给下一个请求，这种连接被重用是不安全的。

该方法也可以让当前 cosocket 对象进入 `closed` 状态，所以这里不再需要事后手工调用 [close](#tcpsockclose) 方法。

该特性是在 `v0.5.0rc1` 版本首次引入的。

[返回目录](#nginx-api-for-lua)

> English source:

tcpsock:setkeepalive
--------------------
**syntax:** *ok, err = tcpsock:setkeepalive(timeout?, size?)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Puts the current socket's connection immediately into the cosocket built-in connection pool and keep it alive until other [connect](#tcpsockconnect) method calls request it or the associated maximal idle timeout is expired.

The first optional argument, `timeout`, can be used to specify the maximal idle timeout (in milliseconds) for the current connection. If omitted, the default setting in the [lua_socket_keepalive_timeout](#lua_socket_keepalive_timeout) config directive will be used. If the `0` value is given, then the timeout interval is unlimited.

The second optional argument, `size`, can be used to specify the maximal number of connections allowed in the connection pool for the current server (i.e., the current host-port pair or the unix domain socket file path). Note that the size of the connection pool cannot be changed once the pool is created. When this argument is omitted, the default setting in the [lua_socket_pool_size](#lua_socket_pool_size) config directive will be used.

When the connection pool exceeds the available size limit, the least recently used (idle) connection already in the pool will be closed to make room for the current connection.

Note that the cosocket connection pool is per Nginx worker process rather than per Nginx server instance, so the size limit specified here also applies to every single Nginx worker process.

Idle connections in the pool will be monitored for any exceptional events like connection abortion or unexpected incoming data on the line, in which cases the connection in question will be closed and removed from the pool.

In case of success, this method returns `1`; otherwise, it returns `nil` and a string describing the error.

When the system receive buffer for the current connection has unread data, then this method will return the "connection in dubious state" error message (as the second return value) because the previous session has unread data left behind for the next session and the connection is not safe to be reused.

This method also makes the current cosocket object enter the "closed" state, so there is no need to manually call the [close](#tcpsockclose) method on it afterwards.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

