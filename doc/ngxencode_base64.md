ngx.encode_base64
-----------------
**语法:** *newstr = ngx.encode_base64(str, no_padding?)*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

通过 `base64` 对 `str` 字符串编码。

自 `0.9.16` 版本后，引入了一个布尔值参数 `no_padding` 用来控制是否需要编码数据填充 `等号` 字符串（默认为 `false`，代表需要填充）。 它表明调用者可以在 `base64` 流数据结尾添加填充数据。<!-- 不知道怎么翻译好 hambut -->


> English source:


**syntax:** *newstr = ngx.encode_base64(str, no_padding?)*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

Encodes `str` to a base64 digest.

Since the `0.9.16` release, an optional boolean-typed `no_padding` argument can be specified to control whether the base64 padding should be appended to the resulting digest (default to `false`, i.e., with padding enabled). This enables streaming base64 digest calculation by (data chunks) though it would be the caller's responsibility to append an appropriate padding at the end of data stream.

[Back to TOC](#nginx-api-for-lua)