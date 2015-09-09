ngx.quote_sql_str
-----------------
**语法:** *quoted_value = ngx.quote_sql_str(raw_value)*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

根据 `MySQL` 转义规则返回一个转义后字符串。

[Back to TOC](#nginx-api-for-lua)

> English source:


**syntax:** *quoted_value = ngx.quote_sql_str(raw_value)*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

Returns a quoted SQL string literal according to the MySQL quoting rules.

