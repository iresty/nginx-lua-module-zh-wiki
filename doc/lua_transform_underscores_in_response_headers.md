lua_transform_underscores_in_response_headers
---------------------------------------------

**语法:** *lua_transform_underscores_in_response_headers on|off*

**默认:** *lua_transform_underscores_in_response_headers on*

**环境:** *http, server, location, location-if*

对于[ngx.header.HEADER](#ngxheaderheader) API 中指定响应头，该指令指定是否将下划线(`_`)转化为连接线(`-`)。

该指令是在`v0.5.0rc32`版本首次引入的。

[返回目录](#directives)

> English source:

lua_transform_underscores_in_response_headers
---------------------------------------------

**syntax:** *lua_transform_underscores_in_response_headers on|off*

**default:** *lua_transform_underscores_in_response_headers on*

**context:** *http, server, location, location-if*

Controls whether to transform underscores (`_`) in the response header names specified in the [ngx.header.HEADER](#ngxheaderheader) API to hypens (`-`).

This directive was first introduced in the `v0.5.0rc32` release.

[Back to TOC](#directives)
