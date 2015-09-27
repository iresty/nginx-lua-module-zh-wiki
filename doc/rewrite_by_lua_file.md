rewrite_by_lua_file
-------------------

**syntax:** *rewrite_by_lua_file &lt;path-to-lua-script-file&gt;*

**context:** *http, server, location, location if*

**phase:** *rewrite tail*

Equivalent to [rewrite_by_lua](#rewrite_by_lua), except that the file specified by `<path-to-lua-script-file>` contains the Lua code, or, as from the `v0.5.0rc32` release, the [Lua/LuaJIT bytecode](#lualuajit-bytecode-support) to be executed.

Nginx variables can be used in the `<path-to-lua-script-file>` string to provide flexibility. This however carries some risks and is not ordinarily recommended.

When a relative path like `foo/bar.lua` is given, they will be turned into the absolute path relative to the `server prefix` path determined by the `-p PATH` command-line option while starting the Nginx server.

When the Lua code cache is turned on (by default), the user code is loaded once at the first request and cached and the Nginx config must be reloaded each time the Lua source file is modified. The Lua code cache can be temporarily disabled during development by switching [lua_code_cache](#lua_code_cache) `off` in `nginx.conf` to avoid reloading Nginx.

The `rewrite_by_lua_file` code will always run at the end of the `rewrite` request-processing phase unless [rewrite_by_lua_no_postpone](#rewrite_by_lua_no_postpone) is turned on.

Nginx variables are supported in the file path for dynamic dispatch just as in [content_by_lua_file](#content_by_lua_file).

[Back to TOC](#directives)

> English source:

rewrite_by_lua_file
-------------------

**syntax:** *rewrite_by_lua_file &lt;path-to-lua-script-file&gt;*

**context:** *http, server, location, location if*

**phase:** *rewrite tail*

Equivalent to [rewrite_by_lua](#rewrite_by_lua), except that the file specified by `<path-to-lua-script-file>` contains the Lua code, or, as from the `v0.5.0rc32` release, the [Lua/LuaJIT bytecode](#lualuajit-bytecode-support) to be executed.

Nginx variables can be used in the `<path-to-lua-script-file>` string to provide flexibility. This however carries some risks and is not ordinarily recommended.

When a relative path like `foo/bar.lua` is given, they will be turned into the absolute path relative to the `server prefix` path determined by the `-p PATH` command-line option while starting the Nginx server.

When the Lua code cache is turned on (by default), the user code is loaded once at the first request and cached and the Nginx config must be reloaded each time the Lua source file is modified. The Lua code cache can be temporarily disabled during development by switching [lua_code_cache](#lua_code_cache) `off` in `nginx.conf` to avoid reloading Nginx.

The `rewrite_by_lua_file` code will always run at the end of the `rewrite` request-processing phase unless [rewrite_by_lua_no_postpone](#rewrite_by_lua_no_postpone) is turned on.

Nginx variables are supported in the file path for dynamic dispatch just as in [content_by_lua_file](#content_by_lua_file).

[Back to TOC](#directives)
