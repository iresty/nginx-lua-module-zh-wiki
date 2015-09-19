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
