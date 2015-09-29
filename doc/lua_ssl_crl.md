lua_ssl_crl
-----------

**语法:** *lua_ssl_crl &lt;file&gt;*

**默认:** *no*

**环境:** *http, server, location*

指定一个 PEM 格式吊销证书文件，在[tcpsock:sslhandshake](#tcpsocksslhandshake)方法里验证 SSL/TLS 服务的证书。

该指令是在`v0.9.11`版本首次引入的。

[返回目录](#directives)

> English source:

lua_ssl_crl
-----------

**syntax:** *lua_ssl_crl &lt;file&gt;*

**default:** *no*

**context:** *http, server, location*

Specifies a file with revoked certificates (CRL) in the PEM format used to verify the certificate of the SSL/TLS server in the [tcpsock:sslhandshake](#tcpsocksslhandshake) method.

This directive was first introduced in the `v0.9.11` release.

[Back to TOC](#directives)
