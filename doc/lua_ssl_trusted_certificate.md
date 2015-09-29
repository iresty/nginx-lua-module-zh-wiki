lua_ssl_trusted_certificate
---------------------------

**语法:** *lua_ssl_trusted_certificate &lt;file&gt;*

**默认:** *no*

**环境:** *http, server, location*

指定一个 PEM 格式信任 CA 证书文件，在[tcpsock:sslhandshake](#tcpsocksslhandshake)方法里验证 SSL/TLS 服务的证书。

该指令是在`v0.9.11`版本首次引入的。

也可以看看 [lua_ssl_verify_depth](#lua_ssl_verify_depth).

[返回目录](#directives)

> English source:

lua_ssl_trusted_certificate
---------------------------

**syntax:** *lua_ssl_trusted_certificate &lt;file&gt;*

**default:** *no*

**context:** *http, server, location*

Specifies a file path with trusted CA certificates in the PEM format used to verify the certificate of the SSL/TLS server in the [tcpsock:sslhandshake](#tcpsocksslhandshake) method.

This directive was first introduced in the `v0.9.11` release.

See also [lua_ssl_verify_depth](#lua_ssl_verify_depth).

[Back to TOC](#directives)
