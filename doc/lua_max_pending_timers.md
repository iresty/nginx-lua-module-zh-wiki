lua_max_pending_timers
----------------------

**语法:** *lua_max_pending_timers &lt;count&gt;*

**默认:** *lua_max_pending_timers 1024*

**环境:** *http*

控制允许使用的`pending timers`最大数量。

`pending timers` 指的是还没有过期的 `timers` 。

当超过这个限制， [ngx.timer.at](#ngxtimerat) 调用将立即返回 `nil` 和 错误信息 “too many pending timers”。

该指令是在`v0.8.0`版本首次引入的。

[返回目录](#directives)

> English source:

lua_max_pending_timers
----------------------

**syntax:** *lua_max_pending_timers &lt;count&gt;*

**default:** *lua_max_pending_timers 1024*

**context:** *http*

Controls the maximum number of pending timers allowed.

Pending timers are those timers that have not expired yet.

When exceeding this limit, the [ngx.timer.at](#ngxtimerat) call will immediately return `nil` and the error string "too many pending timers".

This directive was first introduced in the `v0.8.0` release.

[Back to TOC](#directives)
