lua_socket_buffer_size
----------------------

**语法:** *lua_socket_buffer_size &lt;size&gt;*

**默认:** *lua_socket_buffer_size 4k/8k*

**环境:** *http, server, location*

指定使用 cosocket 进行读取操作时的缓冲区大小。
Specifies the buffer size used by cosocket reading operations.

这个缓冲区不必为了同时解决所有事情而设置的太大，因为 cosocket 支持100%的非缓存读取和解析。所以即使是`1`字节的缓冲区大小依旧可以在任何地方正常工作，只不过效率比较糟糕。

该指令是在`v0.5.0rc1`版本首次引入。

[返回目录](#directives)

> English source:

lua_socket_buffer_size
----------------------

**syntax:** *lua_socket_buffer_size &lt;size&gt;*

**default:** *lua_socket_buffer_size 4k/8k*

**context:** *http, server, location*

Specifies the buffer size used by cosocket reading operations.

This buffer does not have to be that big to hold everything at the same time because cosocket supports 100% non-buffered reading and parsing. So even `1` byte buffer size should still work everywhere but the performance could be terrible.

This directive was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#directives)

