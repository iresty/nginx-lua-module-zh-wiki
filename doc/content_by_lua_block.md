content_by_lua_block
--------------------

**语法:** *content_by_lua_block { lua-script }*

**环境:** *location, location if*

**阶段:** *content*

与 [content_by_lua](#content_by_lua) 指令相似，只不过该指令在一对括号（`{}`）中直接内嵌 Lua 代码，替代之前 NGINX 的字符串（需要特殊字符转义）。

例如：

```nginx

 content_by_lua_block {
     ngx.say("I need no extra escaping here, for example: \r\nblah")
 }
```

该指令在 `v0.9.17` 版本首次引入。

[返回目录](#directives)

> English source:

content_by_lua_block
--------------------

**syntax:** *content_by_lua_block { lua-script }*

**context:** *location, location if*

**phase:** *content*

Similar to the [content_by_lua](#content_by_lua) directive except that this directive inlines
the Lua source directly
inside a pair of curly braces (`{}`) instead of in an NGINX string literal (which requires
special character escaping).

For instance,

```nginx

 content_by_lua_block {
     ngx.say("I need no extra escaping here, for example: \r\nblah")
 }
```

This directive was first introduced in the `v0.9.17` release.

[Back to TOC](#directives)
