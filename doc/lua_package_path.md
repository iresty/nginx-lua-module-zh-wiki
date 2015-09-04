lua_package_path
----------------

**语法:** *lua_package_path &lt;lua-style-path-str&gt;*

**默认:** *The content of LUA_PATH environ variable or Lua's compiled-in defaults.*

**环境:** *http*

Sets the Lua module search path used by scripts specified by [set_by_lua](#set_by_lua),
[content_by_lua](#content_by_lua) and others. The path string is in standard Lua path form, and `;;`
can be used to stand for the original search paths.

As from the `v0.5.0rc29` release, the special notation `$prefix` or `${prefix}` can be used in the search path string to indicate the path of the `server prefix` usually determined by the `-p PATH` command-line option while starting the Nginx server.

[Back to TOC](#directives)

> English source:

lua_package_path
----------------

**syntax:** *lua_package_path &lt;lua-style-path-str&gt;*

**default:** *The content of LUA_PATH environ variable or Lua's compiled-in defaults.*

**context:** *http*

Sets the Lua module search path used by scripts specified by [set_by_lua](#set_by_lua),
[content_by_lua](#content_by_lua) and others. The path string is in standard Lua path form, and `;;`
can be used to stand for the original search paths.

As from the `v0.5.0rc29` release, the special notation `$prefix` or `${prefix}` can be used in the search path string to indicate the path of the `server prefix` usually determined by the `-p PATH` command-line option while starting the Nginx server.

[Back to TOC](#directives)