lua_package_cpath
-----------------

**语法:** *lua_package_cpath &lt;lua-style-cpath-str&gt;*

**默认:** *The content of LUA_CPATH environment variable or Lua's compiled-in defaults.*

**环境:** *http*

设置[set_by_lua](#set_by_lua)，[content_by_lua](#content_by_lua) 和 其他脚本对Lua C模块的查找路径。 cpath路径字符串是标准Luacpath路径格式，`;;` 可被用来代表原始cpath路径。

从`v0.5.0rc29`发行版开始，特殊符号`$prefix` 或 `${prefix}`可用于搜索路径字符串中。`server prefix`的值，通常是由Nginx服务启动时的`-p PATH`命令行决定的。

[返回目录](#directives)

> English source:

lua_package_cpath
-----------------

**syntax:** *lua_package_cpath &lt;lua-style-cpath-str&gt;*

**default:** *The content of LUA_CPATH environment variable or Lua's compiled-in defaults.*

**context:** *http*

Sets the Lua C-module search path used by scripts specified by [set_by_lua](#set_by_lua),
[content_by_lua](#content_by_lua) and others. The cpath string is in standard Lua cpath form, and `;;`
can be used to stand for the original cpath.

As from the `v0.5.0rc29` release, the special notation `$prefix` or `${prefix}` can be used in the search path string to indicate the path of the `server prefix` usually determined by the `-p PATH` command-line option while starting the Nginx server.

[Back to TOC](#directives)


