body_filter_by_lua_block
------------------------

**语法:** *body_filter_by_lua_block { lua-script-str }*

**环境:** *http, server, location, location if*

**阶段:** *output-body-filter*

与 [body_filter_by_lua](#body_filter_by_lua) 指令相似，只不过该指令在一对括号（`{}`）中直接内嵌 Lua 代码，替代之前 NGINX 的字符串（需要特殊字符转义）。

例如：

```nginx

 body_filter_by_lua_block {
     local data, eof = ngx.arg[1], ngx.arg[2]
 }
```

该指令在 `v0.9.17` 版本首次引入。

[返回目录](#directives)

> English source:

body_filter_by_lua_block
------------------------

**syntax:** *body_filter_by_lua_block { lua-script-str }*

**context:** *http, server, location, location if*

**phase:** *output-body-filter*

Similar to the [body_filter_by_lua](#body_filter_by_lua) directive except that this directive inlines
the Lua source directly
inside a pair of curly braces (`{}`) instead of in an NGINX string literal (which requires
special character escaping).

For instance,

```nginx

 body_filter_by_lua_block {
     local data, eof = ngx.arg[1], ngx.arg[2]
 }
```

This directive was first introduced in the `v0.9.17` release.

[Back to TOC](#directives)