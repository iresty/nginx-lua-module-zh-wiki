header_filter_by_lua_file
-------------------------

**语法:** *header_filter_by_lua_file &lt;path-to-lua-script-file&gt;*

**环境:** *http, server, location, location if*

**阶段:** *output-header-filter*

除了通过文件`<path-to-lua-script-file>`的内容指定 Lua 代码外，该指令与[header_filter_by_lua](#header_filter_by_lua)是等价的，该指令从`v0.5.0rc32`开始支持[Lua/LuaJIT bytecode](#lualuajit-bytecode-support)的执行。

当给定了一个相对路径如`foo/bar.lua`，它将会被转换成绝对路径，前面增加的部分路径是 Nginx 服务启动时通过命令行选项`-p PATH`决定的`server prefix`。

该指令是在`v0.2.1rc20`版本第一次引入。

[返回目录](#directives)

> English source:

header_filter_by_lua_file
-------------------------

**syntax:** *header_filter_by_lua_file &lt;path-to-lua-script-file&gt;*

**context:** *http, server, location, location if*

**phase:** *output-header-filter*

Equivalent to [header_filter_by_lua](#header_filter_by_lua), except that the file specified by `<path-to-lua-script-file>` contains the Lua code, or as from the `v0.5.0rc32` release, the [Lua/LuaJIT bytecode](#lualuajit-bytecode-support) to be executed.

When a relative path like `foo/bar.lua` is given, they will be turned into the absolute path relative to the `server prefix` path determined by the `-p PATH` command-line option while starting the Nginx server.

This directive was first introduced in the `v0.2.1rc20` release.

[Back to TOC](#directives)