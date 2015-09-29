lua_ssl_protocols
-----------------

**语法:** *lua_ssl_protocols \[SSLv2\] \[SSLv3\] \[TLSv1\] [TLSv1.1] [TLSv1.2]*

**默认:** *lua_ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2*

**环境:** *http, server, location*

在 [tcpsock:sslhandshake](#tcpsocksslhandshake) 方法中开启请求 SSL/TLS 服务的传输协议列表。

该指令是在`v0.9.11`版本首次引入的。

[返回目录](#directives)

> English source:

lua_ssl_protocols
-----------------

**syntax:** *lua_ssl_protocols \[SSLv2\] \[SSLv3\] \[TLSv1\] [TLSv1.1] [TLSv1.2]*

**default:** *lua_ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2*

**context:** *http, server, location*

Enables the specified protocols for requests to a SSL/TLS server in the [tcpsock:sslhandshake](#tcpsocksslhandshake) method.

This directive was first introduced in the `v0.9.11` release.

[Back to TOC](#directives)