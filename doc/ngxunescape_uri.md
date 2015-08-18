ngx.unescape_uri
----------------
**语法:** *newstr = ngx.unescape_uri(str)*

**环境:** *init_by_lua*\**, init_worker_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

将转义过的 URI 内容 `str` 解码。

例如,

```lua

 ngx.say(ngx.unescape_uri("b%20r56+7"))
```

输出


    b r56 7


> English Source

**syntax:** *newstr = ngx.unescape_uri(str)*

**context:** *init_by_lua*\**, init_worker_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

Unescape `str` as an escaped URI component.

For example,

```lua

 ngx.say(ngx.unescape_uri("b%20r56+7"))
```

gives the output


    b r56 7


[返回目录](#nginx-api-for-lua)