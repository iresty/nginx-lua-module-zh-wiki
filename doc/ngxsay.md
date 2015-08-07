**语法:** *ok, err = ngx.say(...)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

与 [ngx.print](#ngxprint) 相同,同时末尾添加一个回车符。



> English Source

**syntax:** *ok, err = ngx.say(...)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

Just as [ngx.print](#ngxprint) but also emit a trailing newline.