init_worker_by_lua_file
-----------------------

**语法:** *init_worker_by_lua_file &lt;lua-file-path&gt;*

**环境:** *http*

**阶段:** *starting-worker*

Similar to [init_worker_by_lua](#init_worker_by_lua), but accepts the file path to a Lua source file or Lua bytecode file.

与[init_worker_by_lua](#init_worker_by_lua)等价，通过`<lua-file-path>`指定的Lua 或 [Lua/LuaJIT 字节码](#lualuajit-bytecode-support)文件来执行。

该指令是在`v0.9.5`发行版第一次引入。

[返回目录](#directives)

> English source:

init_worker_by_lua_file
-----------------------

**syntax:** *init_worker_by_lua_file &lt;lua-file-path&gt;*

**context:** *http*

**phase:** *starting-worker*

Similar to [init_worker_by_lua](#init_worker_by_lua), but accepts the file path to a Lua source file or Lua bytecode file.

This directive was first introduced in the `v0.9.5` release.

[Back to TOC](#directives)

