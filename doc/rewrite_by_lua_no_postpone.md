rewrite_by_lua_no_postpone
--------------------------

**语法:** *rewrite_by_lua_no_postpone on|off*

**默认:** *rewrite_by_lua_no_postpone off*

**环境:** *http*

控制是否禁用[rewrite_by_lua](#rewrite_by_lua) 和 [rewrite_by_lua_file](#rewrite_by_lua_file) 指令在`rewrite`阶段的延迟执行。该指令的默认值是 `off` ，在`rewrite`阶段的 Lua 代码将被延迟到最后执行。

该指令是在`v0.5.0rc29`版本首次引入的。

[返回目录](#directives)

> English source:

rewrite_by_lua_no_postpone
--------------------------

**syntax:** *rewrite_by_lua_no_postpone on|off*

**default:** *rewrite_by_lua_no_postpone off*

**context:** *http*

Controls whether or not to disable postponing [rewrite_by_lua](#rewrite_by_lua) and [rewrite_by_lua_file](#rewrite_by_lua_file) directives to run at the end of the `rewrite` request-processing phase. By default, this directive is turned off and the Lua code is postponed to run at the end of the `rewrite` phase.

This directive was first introduced in the `v0.5.0rc29` release.

[Back to TOC](#directives)
