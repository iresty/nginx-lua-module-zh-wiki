ngx.send_headers
----------------
**语法:** *ok, err = ngx.send_headers()*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

指名把应答头发送出去。

自从 `v0.8.3` 版本开始，成功情况该函数返回 `1` ，否则返回 `nil` 和错误字符描述信息。

注意，在内容通过 [ngx.say](#ngxsay)、[ngx.print](#ngxprint) 输出或当 [content_by_lua](#content_by_lua) 存在时，ngx_lua 将自动发送头部内容，所以通常情况不需要手动发送应答头。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.send_headers
----------------
**syntax:** *ok, err = ngx.send_headers()*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

Explicitly send out the response headers.

Since `v0.8.3` this function returns `1` on success, or returns `nil` and a string describing the error otherwise.

Note that there is normally no need to manually send out response headers as ngx_lua will automatically send headers out
before content is output with [ngx.say](#ngxsay) or [ngx.print](#ngxprint) or when [content_by_lua](#content_by_lua) exits normally.

[Back to TOC](#nginx-api-for-lua)

