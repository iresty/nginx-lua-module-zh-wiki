特别转义序列

PCRE 的转义符号例如 `\d`,`\s` 以及 `\w` 等需要特别注意，因为在字符串语义中，反斜线字符 `\` 会被 Lua 语言解析器和 Nginx 配置文件解析器在执行前同时处理掉，所以以下代码片段将无法按预期运行：
```nginx
 # nginx.conf
 ? location /test {
 ?     content_by_lua '
 ?         local regex = "\d+"  -- 这里是错的!!
 ?         local m = ngx.re.match("hello, 1234", regex)
 ?         if m then ngx.say(m[0]) else ngx.say("not matched!") end
 ?     ';
 ? }
 # 结果为 "not matched!"
```

为避免这个问题，需要双重转义反斜线符号：

```nginx
 # nginx.conf
 location /test {
     content_by_lua '
         local regex = "\\\\d+"
         local m = ngx.re.match("hello, 1234", regex)
         if m then ngx.say(m[0]) else ngx.say("not matched!") end
     ';
 }
 # 结果为 "1234"
```

这里的 `\\\\d+`，先被 Nginx 配置文件解析器处理成 `\\d+` ，再被 Lua 语言解析器处理成 `\d+`，之后才被执行。

或者，正则表达式模板可以使用 Lua 字符串"长括号"语义写出，其语法形式为 `[[...]]`，在这种情况下，反斜线仅需为 Nginx 配置文件解析器转义一次。

```nginx
 # nginx.conf
 location /test {
     content_by_lua '
         local regex = [[\\d+]]
         local m = ngx.re.match("hello, 1234", regex)
         if m then ngx.say(m[0]) else ngx.say("not matched!") end
     ';
 }
 # 结果为 to "1234"
 ```

这里，`[[\\d+]]` 被 Nginx 配置文件解析器处理成 `[[\d+]]`，符合预期。

注意，当正则表达式模板中包括 `[...]` 序列时，Lua 语言中“更长的长括号”形式 `[=[...]=]` 是必要的。如果需要，可以将`[=[...]=]` 作为默认形式。

```nginx
 # nginx.conf
 location /test {
     content_by_lua '
         local regex = [=[[0-9]+]=]
         local m = ngx.re.match("hello, 1234", regex)
         if m then ngx.say(m[0]) else ngx.say("not matched!") end
     ';
 }
 # 结果为 "1234"
```

还有一种转义 PCRE 序列的方法是把 Lua 代码放到外部脚本文件中，通过各种 `*_by_lua_file` 指令执行。在这种方法中，反斜线仅被 Lua 语言解析器处理，因此只需要转义一次。

```lua
 -- test.lua
 local regex = "\\d+"
 local m = ngx.re.match("hello, 1234", regex)
 if m then ngx.say(m[0]) else ngx.say("not matched!") end
 -- 结果为 "1234"
```

在外部脚本文件中，PCRE 序列如果使用“长括号”形式 Lua 字符串，则无需修改。

```lua
 -- test.lua
 local regex = [[\d+]]
 local m = ngx.re.match("hello, 1234", regex)
 if m then ngx.say(m[0]) else ngx.say("not matched!") end
 -- 结果为 "1234"
```


> English source:

PCRE sequences such as `\d`, `\s`, or `\w`, require special attention because in string literals, the backslash character, `\`, is stripped out by both the Lua language parser and by the Nginx config file parser before processing. So the following snippet will not work as expected:

```nginx

 # nginx.conf
 ? location /test {
 ?     content_by_lua '
 ?         local regex = "\d+"  -- THIS IS WRONG!!
 ?         local m = ngx.re.match("hello, 1234", regex)
 ?         if m then ngx.say(m[0]) else ngx.say("not matched!") end
 ?     ';
 ? }
 # evaluates to "not matched!"
```

To avoid this, *double* escape the backslash:

```nginx

 # nginx.conf
 location /test {
     content_by_lua '
         local regex = "\\\\d+"
         local m = ngx.re.match("hello, 1234", regex)
         if m then ngx.say(m[0]) else ngx.say("not matched!") end
     ';
 }
 # evaluates to "1234"
```

Here, `\\\\d+` is stripped down to `\\d+` by the Nginx config file parser and this is further stripped down to `\d+` by the Lua language parser before running.

Alternatively, the regex pattern can be presented as a long-bracketed Lua string literal by encasing it in "long brackets", `[[...]]`, in which case backslashes have to only be escaped once for the Nginx config file parser. 

```nginx

 # nginx.conf
 location /test {
     content_by_lua '
         local regex = [[\\d+]]
         local m = ngx.re.match("hello, 1234", regex)
         if m then ngx.say(m[0]) else ngx.say("not matched!") end
     ';
 }
 # evaluates to "1234"
```

Here, `[[\\d+]]` is stripped down to `[[\d+]]` by the Nginx config file parser and this is processed correctly.

Note that a longer from of the long bracket, `[=[...]=]`, may be required if the regex pattern contains `[...]` sequences. 
The `[=[...]=]` form may be used as the default form if desired.

```nginx

 # nginx.conf
 location /test {
     content_by_lua '
         local regex = [=[[0-9]+]=]
         local m = ngx.re.match("hello, 1234", regex)
         if m then ngx.say(m[0]) else ngx.say("not matched!") end
     ';
 }
 # evaluates to "1234"
```

An alternative approach to escaping PCRE sequences is to ensure that Lua code is placed in external script files and executed using the various `*_by_lua_file` directives. 
With this approach, the backslashes are only stripped by the Lua language parser and therefore only need to be escaped once each.

```lua

 -- test.lua
 local regex = "\\d+"
 local m = ngx.re.match("hello, 1234", regex)
 if m then ngx.say(m[0]) else ngx.say("not matched!") end
 -- evaluates to "1234"
```

Within external script files, PCRE sequences presented as long-bracketed Lua string literals do not require modification. 
 
```lua

 -- test.lua
 local regex = [[\d+]]
 local m = ngx.re.match("hello, 1234", regex)
 if m then ngx.say(m[0]) else ngx.say("not matched!") end
 -- evaluates to "1234"
 ```
