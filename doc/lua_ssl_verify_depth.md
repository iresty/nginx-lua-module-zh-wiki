lua_ssl_verify_depth
--------------------

**语法:** *lua_ssl_verify_depth &lt;number&gt;*

**默认:** *lua_ssl_verify_depth 1*

**环境:** *http, server, location*

设置服务端证书链的验证深度。

该指令是在`v0.9.11`版本首次引入的。

也可以看看 [lua_ssl_trusted_certificate](#lua_ssl_trusted_certificate)。

[返回目录](#directives)

> English source:

lua_ssl_verify_depth
--------------------

**syntax:** *lua_ssl_verify_depth &lt;number&gt;*

**default:** *lua_ssl_verify_depth 1*

**context:** *http, server, location*

Sets the verification depth in the server certificates chain.

This directive was first introduced in the `v0.9.11` release.

See also [lua_ssl_trusted_certificate](#lua_ssl_trusted_certificate).

[Back to TOC](#directives)
