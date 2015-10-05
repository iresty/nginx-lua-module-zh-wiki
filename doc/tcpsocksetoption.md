
tcpsock:setoption
-----------------
**语法:** *tcpsock:setoption(option, value?)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

该函数是为兼容 [LuaSocket](http://w3.impa.br/~diego/software/luasocket/tcp.html) API，目前没做任何事情。它的功能将在将来实现。

该特性是在 `v0.5.0rc1` 版本首次引入的。

[返回目录](#nginx-api-for-lua)

> English source:

tcpsock:setoption
-----------------
**syntax:** *tcpsock:setoption(option, value?)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

This function is added for [LuaSocket](http://w3.impa.br/~diego/software/luasocket/tcp.html) API compatibility and does nothing for now. Its functionality will be implemented in future.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)