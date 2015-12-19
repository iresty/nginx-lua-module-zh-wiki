access_by_lua_no_postpone
-------------------------

**语法:** *access_by_lua_no_postpone on|off*

**默认:** *access_by_lua_no_postpone off*

**环境:** *http*

控制是否禁用 [access_by_lua](#access_by_lua)* 指令在 `access` 请求处理阶段末尾的推迟执行。默认的，该指令是 `off` 并且 `access` 阶段的 Lua 代码是被延迟到末尾执行。

该指令在 `v0.9.20` 版本首次引入。

[返回目录](#directives)

> English source:

access_by_lua_no_postpone
-------------------------

**syntax:** *access_by_lua_no_postpone on|off*

**default:** *access_by_lua_no_postpone off*

**context:** *http*

Controls whether or not to disable postponing [access_by_lua](#access_by_lua)* directives to run at the end of the `access` request-processing phase. By default, this directive is turned off and the Lua code is postponed to run at the end of the `access` phase.

This directive was first introduced in the `v0.9.20` release.

[Back to TOC](#directives)


