不支持与SSI混合
-----------------------------

在同样的Nginx请求混合SSI 与 ngx_lua 是完全不被支持的，只使用ngx_lua即可。使用SSI你可以做的任何事情，使用ngx_lua可以更好的完成，并且效能更棒。

[返回目录](#table-of-contents)

> English source:

Mixing with SSI Not Supported
-----------------------------

Mixing SSI with ngx_lua in the same Nginx request is not supported at all. Just use ngx_lua exclusively. Everything you can do with SSI can be done atop ngx_lua anyway and it can be more efficient when using ngx_lua.

[Back to TOC](#table-of-contents)