tcpsock:getreusedtimes
----------------------
**语法:** *count, err = tcpsock:getreusedtimes()*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

该方法返回当前连接的使用次数（调用成功）。失败时，返回“nil”和错误描述信息。

如果当前连接不是从内建连接池中获取的，该方法总是返回 `0` ，也就是说，该连接还没有被使用过。如果连接来自连接池，那么返回值永远都是非零。所以这个方法可以用来确认当前连接是否来自池子。

该特性是在 `v0.5.0rc1` 版本首次引入的。

[返回目录](#nginx-api-for-lua)

> English source:

tcpsock:getreusedtimes
----------------------
**syntax:** *count, err = tcpsock:getreusedtimes()*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

This method returns the (successfully) reused times for the current connection. In case of error, it returns `nil` and a string describing the error.

If the current connection does not come from the built-in connection pool, then this method always returns `0`, that is, the connection has never been reused (yet). If the connection comes from the connection pool, then the return value is always non-zero. So this method can also be used to determine if the current connection comes from the pool.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

