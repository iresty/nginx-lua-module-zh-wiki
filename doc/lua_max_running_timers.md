lua_max_running_timers
----------------------

**语法:** *lua_max_running_timers &lt;count&gt;*

**默认:** *lua_max_running_timers 256*

**环境:** *http*

控制允许的"running timers"最大数量。

"running timers" 指的是那些正在执行用户回调函数的 timers 。

当超过这个限制，Nginx 将停止执行新近过期的 timers 回调，并记录一个错误日志 “N lua_max_running_timers are not enough”，这里的 "N" 是这个指令的当前值。

该指令是在`v0.8.0`版本首次引入的。

[返回目录](#directives)

> English source:

lua_max_running_timers
----------------------

**syntax:** *lua_max_running_timers &lt;count&gt;*

**default:** *lua_max_running_timers 256*

**context:** *http*

Controls the maximum number of "running timers" allowed.

Running timers are those timers whose user callback functions are still running.

When exceeding this limit, Nginx will stop running the callbacks of newly expired timers and log an error message "N lua_max_running_timers are not enough" where "N" is the current value of this directive.

This directive was first introduced in the `v0.8.0` release.

[Back to TOC](#directives)
