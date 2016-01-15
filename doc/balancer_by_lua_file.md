balancer_by_lua_file
--------------------

**语法:** *balancer_by_lua_file &lt;path-to-lua-script-file&gt;*

**环境:** *upstream*

**阶段:** *content*

除了通过文件 `<path-to-lua-script-file>` 的内容指定 Lua 代码外，该指令与 [balancer_by_lua_block](#balancer_by_lua_block) 是等价的，该指令从 `v0.5.0rc32` 开始支持 [Lua/LuaJIT bytecode](#lualuajit-bytecode-support) 的执行。

当给定了一个相对路径如 `foo/bar.lua`，它将会被转换成绝对路径，前面增加的部分路径是 Nginx 服务启动时通过命令行选项 `-p PATH` 决定的 `server prefix`。

该指令在 `v0.10.0` 版本首次引入。

[返回目录](#directives)

> English source:

balancer_by_lua_file
--------------------

**syntax:** *balancer_by_lua_file &lt;path-to-lua-script-file&gt;*

**context:** *upstream*

**phase:** *content*

Equivalent to [balancer_by_lua_block](#balancer_by_lua_block), except that the file specified by `<path-to-lua-script-file>` contains the Lua code, or, as from the `v0.5.0rc32` release, the [Lua/LuaJIT bytecode](#lualuajit-bytecode-support) to be executed.

When a relative path like `foo/bar.lua` is given, they will be turned into the absolute path relative to the `server prefix` path determined by the `-p PATH` command-line option while starting the Nginx server.

This directive was first introduced in the `v0.10.0` release.

[Back to TOC](#directives)