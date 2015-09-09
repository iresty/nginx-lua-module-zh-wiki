ngx.sha1_bin
------------
**语法:** *digest = ngx.sha1_bin(str)*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

通过 `SHA-1` 计算 `str` 字符串返回二进制的数据。

在安装 `Nginx` 时 这个函数需要 `SHA-1` 的支持。（这通常说明应该在安装 `Nginx` 时一起安装 `OpenSSL` 库）。

这个方法在 `v0.5.0rc6` 版本中首次引入。

[Back to TOC](#nginx-api-for-lua)

> English source:


**syntax:** *digest = ngx.sha1_bin(str)*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

Returns the binary form of the SHA-1 digest of the `str` argument.

This function requires SHA-1 support in the Nginx build. (This usually just means OpenSSL should be installed while building Nginx).

This function was first introduced in the `v0.5.0rc6`.

