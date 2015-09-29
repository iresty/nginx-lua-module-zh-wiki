lua_ssl_ciphers
---------------

**语法:** *lua_ssl_ciphers &lt;ciphers&gt;*

**默认:** *lua_ssl_ciphers DEFAULT*

**环境:** *http, server, location*

指定在[tcpsock:sslhandshake](#tcpsocksslhandshake) 方法中请求 SSL/TLS 服务 的加密方式。 其中参数 ciphers 是 OpenSSL 库里面指定的格式。

可以使用 “openssl ciphers” 来查看完整的加密方式列表。

该指令是在`v0.9.11`版本首次引入的。

[返回目录](#directives)

> English source:

lua_ssl_ciphers
---------------

**syntax:** *lua_ssl_ciphers &lt;ciphers&gt;*

**default:** *lua_ssl_ciphers DEFAULT*

**context:** *http, server, location*

Specifies the enabled ciphers for requests to a SSL/TLS server in the [tcpsock:sslhandshake](#tcpsocksslhandshake) method. The ciphers are specified in the format understood by the OpenSSL library.

The full list can be viewed using the “openssl ciphers” command.

This directive was first introduced in the `v0.9.11` release.

[Back to TOC](#directives)