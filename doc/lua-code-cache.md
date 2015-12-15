lua_code_cache
---------------
**语法**: *lua_code_cache on | off*

**默认**: *lua_code_cache on*

**环境**: *http, server, location, location if*

打开或者关闭 *_by_lua_file 指令（类似 set_by_lua_file 和 content_by_lua_file）
中指定的 Lua 代码，以及 Lua 模块的 Lua 代码缓存。
。

当它关闭时，每个请求都将会运行在一个单独的 Lua 虚拟机中，在 v0.9.3 release 版本中引入。所以在 set_by_lua_file, content_by_lua_file, access_by_lua_file 中引用 lua 文件时不会被缓存并且模块每次会重新加载。有了这个选项开发者很容易通过编辑文件并重新请求的方法进行测试。


然而需要注意的是，当你在编辑直接写在 nginx 配置中的 lua 代码时，它并不是像 set_by_lua,content_by_lua, access_by_lua, rewrite_by_lua 指令一样通过保存就能更新。 只有 nginx 的配置解释器才能正确解析 nginx 的配置文件，所以此时唯一的办法是向 nginx 发送重新加载配置文件的指令或者重启 nginx 进程的指令。

```shell
kill -HUP pid
nginx -s reload
```

即使代码缓存指定打开时，在 *_by_lua_file 指令中使用 dofile 或 loadfile 函数时内容不会被缓存（除非你自己直接缓存结果）。通常你可以在  init_by_lua 或 init_by_lua_file 指令中加载这些文件或通过真正的 lua 模块去实现他们。


现在 ngx_lua 模块的还不支持 Apache mod_lua 的可用统计模块。


不推荐在生产环境中关闭 lua 代码缓存，请确保它只在开发环境中使用，他对整体性能有非常明显的影响。举个例子，输出“你好世界”在没有开启 lua 代码缓存时可以降低一个量级。


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
