Lua Coroutine Yielding/Resuming
-------------------------------

Because Lua's `dofile` and `require` builtins are currently implemented as C functions in both Lua 5.1 and LuaJIT 2.0/2.1, if the Lua file being loaded by `dofile` or `require` invokes [ngx.location.capture*](#ngxlocationcapture), [ngx.exec](#ngxexec), [ngx.exit](#ngxexit), or other API functions requiring yielding in the *top-level* scope of the Lua file, then the Lua error "attempt to yield across C-call boundary" will be raised. To avoid this, put these calls requiring yielding into your own Lua functions in the Lua file instead of the top-level scope of the file.

As the standard Lua 5.1 interpreter's VM is not fully resumable, the methods [ngx.location.capture](#ngxlocationcapture), [ngx.location.capture_multi](#ngxlocationcapture_multi), [ngx.redirect](#ngxredirect), [ngx.exec](#ngxexec), and [ngx.exit](#ngxexit) cannot be used within the context of a Lua [pcall()](http://www.lua.org/manual/5.1/manual.html#pdf-pcall) or [xpcall()](http://www.lua.org/manual/5.1/manual.html#pdf-xpcall) or even the first line of the `for ... in ...` statement when the standard Lua 5.1 interpreter is used and the `attempt to yield across metamethod/C-call boundary` error will be produced. Please use LuaJIT 2.x, which supports a fully resumable VM, to avoid this.

[Back to TOC](#table-of-contents)

> English source:

Lua Coroutine Yielding/Resuming
-------------------------------
* Because Lua's `dofile` and `require` builtins are currently implemented as C functions in both Lua 5.1 and LuaJIT 2.0/2.1, if the Lua file being loaded by `dofile` or `require` invokes [ngx.location.capture*](#ngxlocationcapture), [ngx.exec](#ngxexec), [ngx.exit](#ngxexit), or other API functions requiring yielding in the *top-level* scope of the Lua file, then the Lua error "attempt to yield across C-call boundary" will be raised. To avoid this, put these calls requiring yielding into your own Lua functions in the Lua file instead of the top-level scope of the file.
* As the standard Lua 5.1 interpreter's VM is not fully resumable, the methods [ngx.location.capture](#ngxlocationcapture), [ngx.location.capture_multi](#ngxlocationcapture_multi), [ngx.redirect](#ngxredirect), [ngx.exec](#ngxexec), and [ngx.exit](#ngxexit) cannot be used within the context of a Lua [pcall()](http://www.lua.org/manual/5.1/manual.html#pdf-pcall) or [xpcall()](http://www.lua.org/manual/5.1/manual.html#pdf-xpcall) or even the first line of the `for ... in ...` statement when the standard Lua 5.1 interpreter is used and the `attempt to yield across metamethod/C-call boundary` error will be produced. Please use LuaJIT 2.x, which supports a fully resumable VM, to avoid this.

[Back to TOC](#table-of-contents)