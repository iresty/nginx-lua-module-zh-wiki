lua_code_cache
---------------
**语法**: *lua_code_cache on | off*

**默认**: *lua_code_cache on*

**环境**: *http, server, location, location if*

打开或者关闭 *_by_lua_file 指令（类似 set_by_lua_file 和 content_by_lua_file）
中指定的 Lua 代码，以及 Lua 模块的 Lua 代码缓存。

当缓存关闭时，每个 ngx_lua 处理的请求都将会运行在一个单独的 Lua 虚拟机实例中，从 0.9.3 版本开始。
所以在 set_by_lua_file, content_by_lua_file, access_by_lua_file 中引用的 Lua 文件不会被缓存，
并且所有使用的 Lua 模块都会从头开始加载。有了这个选项，开发者很容易通过编辑文件并重新请求的方法进行测试。

然而需要注意的是，当你编辑写在 nginx.conf 中内嵌的 Lua 代码时，
比如通过 set_by_lua, content_by_lua, access_by_lua, rewrite_by_lua 这些指令
写在 nginx.conf 里面的 Lua 代码，缓存不会被更新。
因为只有 Nginx 的配置文件解释器才能正确解析 nginx.conf，
所以重新加载配置文件的唯一办法是发送 HUP 信号或者重启 Nginx。

```shell
kill -HUP pid
nginx -s reload
```

即使代码缓存打开了，在 *_by_lua_file 中使用 dofile 或 loadfile
函数时内容不会被缓存（除非你自己缓存结果）。
通常你可以在 init_by_lua 或 init_by_lua_file 指令中加载所有这些文件，
或者让这些 Lua 文件变成真正的 Lua 模块，通过 require 来加载。

现在 ngx_lua 模块还不支持 Apache mod_lua 模块中可用的 stat 模式（stat mode在 TODO 列表中）。

不推荐在生产环境中关闭 lua 代码缓存，请确保它只在开发环境中使用，他对整体性能有非常明显的影响。
举个例子，输出“你好世界”在没有开启 lua 代码缓存时可以降低一个量级。


> English source:
lua_code_cache
syntax: lua_code_cache on | off

default: lua_code_cache on

context: http, server, location, location if

Enables or disables the Lua code cache for Lua code in *_by_lua_file directives (like set_by_lua_file and content_by_lua_file) and Lua modules.

When turning off, every request served by ngx_lua will run in a separate Lua VM instance, starting from the 0.9.3 release. So the Lua files referenced in set_by_lua_file, content_by_lua_file, access_by_lua_file, and etc will not be cached and all Lua modules used will be loaded from scratch. With this in place, developers can adopt an edit-and-refresh approach.

Please note however, that Lua code written inlined within nginx.conf such as those specified by set_by_lua, content_by_lua, access_by_lua, and rewrite_by_lua will not be updated when you edit the inlined Lua code in your nginx.conf file because only the Nginx config file parser can correctly parse the nginx.conf file and the only way is to reload the config file by sending a HUP signal or just to restart Nginx.

Even when the code cache is enabled, Lua files which are loaded by dofile or loadfile in *_by_lua_file cannot be cached (unless you cache the results yourself). Usually you can either use the init_by_lua or init_by_lua_file directives to load all such files or just make these Lua files true Lua modules and load them via require.

The ngx_lua module does not support the stat mode available with the Apache mod_lua module (yet).

Disabling the Lua code cache is strongly discouraged for production use and should only be used during development as it has a significant negative impact on overall performance. For example, the performance a "hello world" Lua example can drop by an order of magnitude after disabling the Lua code cache.
