
udpsock:send
------------
**语法:** *ok, err = udpsock:send(data)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

对当前 UDP 或 unix 域数据报 socket 对象发送数据。

成功情况下，返回 `1`。其他情况，返回 `nil` 和错误描述信息。

输入参数 `data` 可以是 Lua 字符串，也可以是包含字符串的（嵌套）Lua 表。对于输入参数是表的情况，该方法将逐一拷贝所有的字符串对象到底层的 Nginx socket 发送缓冲区，这是比 Lua 层面完成字符串拼接更好的优化方案。

该特性是在 `v0.5.7` 版本首次引入的。

[返回目录](#nginx-api-for-lua)

> English source:

udpsock:send
------------
**syntax:** *ok, err = udpsock:send(data)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Sends data on the current UDP or datagram unix domain socket object.

In case of success, it returns `1`. Otherwise, it returns `nil` and a string describing the error.

The input argument `data` can either be a Lua string or a (nested) Lua table holding string fragments. In case of table arguments, this method will copy all the string elements piece by piece to the underlying Nginx socket send buffers, which is usually optimal than doing string concatenation operations on the Lua land.

This feature was first introduced in the `v0.5.7` release.

[Back to TOC](#nginx-api-for-lua)