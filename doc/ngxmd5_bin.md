ngx.md5_bin
-----------
**语法:** *digest = ngx.md5_bin(str)*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

通过计算 `str` 字符串返回二进制的数据。

如果需要返回纯文本数据请看 [ngx.md5](ngxmd5.md) 方法。

[Back to TOC](#nginx-api-for-lua)
> English source:

**syntax:** *digest = ngx.md5_bin(str)*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

Returns the binary form of the MD5 digest of the `str` argument.

See [ngx.md5](#ngxmd5) if the hexadecimal form of the MD5 digest is required.

