header_filter_by_lua_block
--------------------------

**语法:** *header_filter_by_lua_block { lua-script }*

**环境:** *http, server, location, location if*

**阶段:** *output-header-filter*

与 [header_filter_by_lua](#header_filter_by_lua) 指令相似，只不过该指令在一对括号（`{}`）中直接内嵌 Lua 代码，替代之前 NGINX 的字符串（需要特殊字符转义）。

例如：

```nginx

 header_filter_by_lua_block {
     ngx.header["content-length"] = nil
 }
```

该指令在 `v0.9.17` 版本首次引入。

[返回目录](#directives)

> English source:

header_filter_by_lua_block
--------------------------

**syntax:** *header_filter_by_lua_block { lua-script }*

**context:** *http, server, location, location if*

**phase:** *output-header-filter*

Similar to the [header_filter_by_lua](#header_filter_by_lua) directive except that this directive inlines
the Lua source directly
inside a pair of curly braces (`{}`) instead of in an NGINX string literal (which requires
special character escaping).

For instance,

```nginx

 header_filter_by_lua_block {
     ngx.header["content-length"] = nil
 }
```

This directive was first introduced in the `v0.9.17` release.

[Back to TOC](#directives)
