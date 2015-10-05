lua_socket_pool_size
--------------------

**语法:** *lua_socket_pool_size &lt;size&gt;*

**默认:** *lua_socket_pool_size 30*

**环境:** *http, server, location*

指定每个 cosocket 通过远程服务(例如，使用主机+端口配对或 unix socket 文件路径作为标识)关联的连接池的大小限制（每个地址中的连接数）。

每个连接池默认是30个连接。

当连接池中连接数超过限制大小，在连接池中最近最少使用的（空闲）连接将被关闭，给当前连接腾挪空间。

注意，cosocket 连接池是每个 Nginx 工作进程使用的，而不是每个 Nginx 服务实例，所以这里指定的限制也只能在每个独立的 nginx 工作进程上生效。

该指令在`v0.5.0rc1`版本首次引入。

[返回目录](#directives)

> English source:

lua_socket_pool_size
--------------------

**syntax:** *lua_socket_pool_size &lt;size&gt;*

**default:** *lua_socket_pool_size 30*

**context:** *http, server, location*

Specifies the size limit (in terms of connection count) for every cosocket connection pool associated with every remote server (i.e., identified by either the host-port pair or the unix domain socket file path).

Default to 30 connections for every pool.

When the connection pool exceeds the available size limit, the least recently used (idle) connection already in the pool will be closed to make room for the current connection.

Note that the cosocket connection pool is per nginx worker process rather than per nginx server instance, so size limit specified here also applies to every single nginx worker process.

This directive was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#directives)
