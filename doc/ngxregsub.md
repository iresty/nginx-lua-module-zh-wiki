ngx.re.gsub
-----------
**语法:** *newstr, n, err = ngx.re.gsub(subject, regex, replace, options?)*

**环境:** *init_worker_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

就象 [ngx.re.sub](#ngxresub), 但执行全局替换。

下面是例子:

```lua

 local newstr, n, err = ngx.re.gsub("hello, world", "([a-z])[a-z]+", "[$0,$1]", "i")
 if newstr then
     -- newstr == "[hello,h], [world,w]"
     -- n == 2
 else
     ngx.log(ngx.ERR, "error: ", err)
     return
 end
```

```lua

 local func = function (m)
     return "[" .. m[0] .. "," .. m[1] .. "]"
 end
 local newstr, n, err = ngx.re.gsub("hello, world", "([a-z])[a-z]+", func, "i")
     -- newstr == "[hello,h], [world,w]"
     -- n == 2
```

这个方法需要在 Nginx 中启用 PCRE 库。 ([Known Issue With Special Escaping Sequences](#special-escaping-sequences)).

这个功能最早出现在 `v0.2.1rc15` 版本中。



> English Source

**syntax:** *newstr, n, err = ngx.re.gsub(subject, regex, replace, options?)*

**context:** *init_worker_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

Just like [ngx.re.sub](#ngxresub), but does global substitution.

Here is some examples:

```lua

 local newstr, n, err = ngx.re.gsub("hello, world", "([a-z])[a-z]+", "[$0,$1]", "i")
 if newstr then
     -- newstr == "[hello,h], [world,w]"
     -- n == 2
 else
     ngx.log(ngx.ERR, "error: ", err)
     return
 end
```

```lua

 local func = function (m)
     return "[" .. m[0] .. "," .. m[1] .. "]"
 end
 local newstr, n, err = ngx.re.gsub("hello, world", "([a-z])[a-z]+", func, "i")
     -- newstr == "[hello,h], [world,w]"
     -- n == 2
```

This method requires the PCRE library enabled in Nginx.  ([Known Issue With Special Escaping Sequences](#special-escaping-sequences)).

This feature was first introduced in the `v0.2.1rc15` release.

[返回目录](#nginx-api-for-lua)