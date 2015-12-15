lua_use_default_type
------------------

**语法:**: *lua_use_default_type on | off*

**默认**: *lua_use_default_type on*

**环境**: *http, server, location, location if*

指定响应头中 Content-Type 的默认值，是否使用 default_type 指令中指明的 MIME 类型。
如果你的 Lua 请求处理程序不想要默认的 Content-Type 响应头，可以关闭这个指令。

默认情况下该指令被打开。

这个指令在 v0.9.1 release 版本中首次引入。

> English source:

lua_use_default_type
syntax: lua_use_default_type on | off

default: lua_use_default_type on

context: http, server, location, location if

Specifies whether to use the MIME type specified by the default_type directive for the default value of the Content-Type response header. If you do not want a default Content-Type response header for your Lua request handlers, then turn this directive off.

This directive is turned on by default.

This directive was first introduced in the v0.9.1 release.
