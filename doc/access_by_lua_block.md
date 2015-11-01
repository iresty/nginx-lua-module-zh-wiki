access_by_lua_block
-------------------

**语法:** *access_by_lua_block { lua-script }*

**环境:** *http, server, location, location if*

**阶段:** *access tail*

与 [access_by_lua](#access_by_lua) 指令相似，只不过该指令在一对括号（`{}`）中直接内嵌 Lua 代码，替代之前 NGINX 的字符串（需要特殊字符转义）。

For instance,

```nginx

 access_by_lua_block {
     do_something("hello, world!\nhiya\n")
 }
```

该指令在 `v0.9.17` 版本首次引入。

[返回目录](#directives)

> English source:

access_by_lua_block
-------------------

**syntax:** *access_by_lua_block { lua-script }*

**context:** *http, server, location, location if*

**phase:** *access tail*

Similar to the [access_by_lua](#access_by_lua) directive except that this directive inlines
the Lua source directly
inside a pair of curly braces (`{}`) instead of in an NGINX string literal (which requires
special character escaping).

For instance,

```nginx

 access_by_lua_block {
     do_something("hello, world!\nhiya\n")
 }
```

This directive was first introduced in the `v0.9.17` release.

[Back to TOC](#directives)

