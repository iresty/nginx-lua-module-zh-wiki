content_by_lua_file
-------------------

**语法:** *content_by_lua_file &lt;path-to-lua-script-file&gt;*

**环境:** *location, location if*

**阶段:** *content*

除了通过文件`<path-to-lua-script-file>`的内容指定 Lua 代码外，该指令与[content_by_lua](#content_by_lua)是等价的，该指令从`v0.5.0rc32`开始支持[Lua/LuaJIT bytecode](#lualuajit-bytecode-support)的执行。

在`<path-to-lua-script-file>`中可以使用 Nginx 的内置变量用来提高灵活性，然而这带有一定的风险，通常并不推荐使用。

当给定了一个相对路径如`foo/bar.lua`，它将会被转换成绝对路径，前面增加的部分路径是 Nginx 服务启动时通过命令行选项`-p PATH`决定的`server prefix`。

当 Lua 代码缓存开启（默认），用户代码在第一次请求时完成加载（只有一次）并缓存，当 Lua 文件被修改时，每次都要对 Nginx 配置进行重新加载。Lua 代码缓存是可以暂时被禁用，通过开关[lua_code_cache](#lua_code_cache)在`nginx.conf`中设置为`off`，这样就可以避免反复重新加载 Nginx。

支持通过 Nginx 变量完成动态调度文件路径，例如：

```nginx

 # 注意: nginx 变量必须要小心过滤，否则它将带来严重的安全风险！
 location ~ ^/app/([-_a-zA-Z0-9/]+) {
     set $path $1;
     content_by_lua_file /path/to/lua/app/root/$path.lua;
 }
```

一定要非常小心恶意用户的输入，并始终仔细验证或过滤掉用户提供的路径项。

[返回目录](#directives)

> English source:

content_by_lua_file
-------------------

**syntax:** *content_by_lua_file &lt;path-to-lua-script-file&gt;*

**context:** *location, location if*

**phase:** *content*

Equivalent to [content_by_lua](#content_by_lua), except that the file specified by `<path-to-lua-script-file>` contains the Lua code, or, as from the `v0.5.0rc32` release, the [Lua/LuaJIT bytecode](#lualuajit-bytecode-support) to be executed.

Nginx variables can be used in the `<path-to-lua-script-file>` string to provide flexibility. This however carries some risks and is not ordinarily recommended.

When a relative path like `foo/bar.lua` is given, they will be turned into the absolute path relative to the `server prefix` path determined by the `-p PATH` command-line option while starting the Nginx server.

When the Lua code cache is turned on (by default), the user code is loaded once at the first request and cached
and the Nginx config must be reloaded each time the Lua source file is modified.
The Lua code cache can be temporarily disabled during development by
switching [lua_code_cache](#lua_code_cache) `off` in `nginx.conf` to avoid reloading Nginx.

Nginx variables are supported in the file path for dynamic dispatch, for example:

```nginx

 # WARNING: contents in nginx var must be carefully filtered,
 # otherwise there'll be great security risk!
 location ~ ^/app/([-_a-zA-Z0-9/]+) {
     set $path $1;
     content_by_lua_file /path/to/lua/app/root/$path.lua;
 }
```

But be very careful about malicious user inputs and always carefully validate or filter out the user-supplied path components.

[Back to TOC](#directives)
