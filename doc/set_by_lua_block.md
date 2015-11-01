set_by_lua_block
----------------

**语法:** *set_by_lua_block $res { lua-script }*

**环境:** *server, server if, location, location if*

**阶段:** *rewrite*

与 [set_by_lua](#set_by_lua) 指令相似，以下情况除外：

1. 该指令在一对括号（`{}`）中直接内嵌 Lua 代码，替代之前 NGINX 的字符串（需要特殊字符转义）
1. 该指令和 [set_by_lua](#set_by_lua) 一样，在 Lua 脚本的后面不支持额外参数

例如：

```nginx

 set_by_lua_block $res { return 32 + math.cos(32) }
 # $res now has the value "32.834223360507" or alike.
```

在 Lua 代码块中无需任何的特殊转义。

该指令在 `v0.9.17` 版本首次引入。

[返回目录C](#directives)

> English source:

set_by_lua_block
----------------

**syntax:** *set_by_lua_block $res { lua-script }*

**context:** *server, server if, location, location if*

**phase:** *rewrite*

Similar to the [set_by_lua](#set_by_lua) directive except that

1. this directive inlines the Lua source directly
inside a pair of curly braces (`{}`) instead of in an NGINX string literal (which requires
special character escaping), and
1. this directive does not support extra arguments after the Lua script as in [set_by_lua](#set_by_lua).

For example,

```nginx

 set_by_lua_block $res { return 32 + math.cos(32) }
 # $res now has the value "32.834223360507" or alike.
```

No special escaping is required in the Lua code block.

This directive was first introduced in the `v0.9.17` release.

[Back to TOC](#directives)

