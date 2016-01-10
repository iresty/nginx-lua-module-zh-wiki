ngx.unescape_uri
----------------
**语法:** *newstr = ngx.unescape_uri(str)*

**环境:** *init_by_lua&#42;, init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

将转义过的 URI 内容 `str` 解码。

例如,

```lua

 ngx.say(ngx.unescape_uri("b%20r56+7"))
```

输出


    b r56 7


> English Source

**syntax:** *newstr = ngx.unescape_uri(str)*

**context:** *init_by_lua&#42;, init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Unescape `str` as an escaped URI component.

For example,

```lua

 ngx.say(ngx.unescape_uri("b%20r56+7"))
```

gives the output


    b r56 7


[返回目录](#nginx-api-for-lua)