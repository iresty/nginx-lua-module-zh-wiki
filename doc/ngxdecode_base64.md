ngx.decode_base64
-----------------
**语法:** *newstr = ngx.decode_base64(str)*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

通过 `base64` 解码 `str` 字符串得到未编码过的字符串。如果 `str` 字符串没有被正常解码将会返回 `nil`。

> English source:


**syntax:** *newstr = ngx.decode_base64(str)*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

Decodes the `str` argument as a base64 digest to the raw form. Returns `nil` if `str` is not well formed.

[Back to TOC](#nginx-api-for-lua)