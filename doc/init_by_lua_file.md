init_by_lua_file
----------------

**语法:** *init_by_lua_file &lt;path-to-lua-script-file&gt;*

**环境:** *http*

**阶段:** *loading-config*

与[init_by_lua](#init_by_lua)等价，通过`<path-to-lua-script-file>`指定文件的Lua 代码 或 [Lua/LuaJIT 字节码](#lualuajit-bytecode-support)来执行。

当给定了一个相对路径如`foo/bar.lua`，它将会被转换成绝对路径，前面增加的部分路径是Nginx服务启动时通过命令行选项`-p PATH`决定的`server prefix`。

该指令在`v0.5.5`发行版第一次被引入。

[返回目录](#directives)

> English source:

init_by_lua_file
----------------

**syntax:** *init_by_lua_file &lt;path-to-lua-script-file&gt;*

**context:** *http*

**phase:** *loading-config*

Equivalent to [init_by_lua](#init_by_lua), except that the file specified by `<path-to-lua-script-file>` contains the Lua code or [Lua/LuaJIT bytecode](#lualuajit-bytecode-support) to be executed.

When a relative path like `foo/bar.lua` is given, they will be turned into the absolute path relative to the `server prefix` path determined by the `-p PATH` command-line option while starting the Nginx server.

This directive was first introduced in the `v0.5.5` release.

[Back to TOC](#directives)
