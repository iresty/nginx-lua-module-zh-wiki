Lua 协程 Yielding/Resuming
-------------------------------

无论Lua 5.1 and LuaJIT 2.0/2.1，内建的 `dofile` 和 `require` 当前都是通过绑定 C 函数的方式，如果Lua文件的加载是`dofile` 或 `require` 并调用[ngx.location.capture*](#ngxlocationcapture), [ngx.exec](#ngxexec), [ngx.exit](#ngxexit), 或者Lua中其他 API 函数的 *top-level* 范围调用 yielding ，均会得到"attempt to yield across C-call boundary"的错误信息。为了避免这个情况，把需要调用yielding部分放到你自己的 Lua 函数中，这样在当前文件就不再是 *top-level* 范围。


对于标准 Lua 5.1 解析器的虚拟机唤醒支持是不完善的，[ngx.location.capture](#ngxlocationcapture), [ngx.location.capture_multi](#ngxlocationcapture_multi), [ngx.redirect](#ngxredirect), [ngx.exec](#ngxexec), 和 [ngx.exit](#ngxexit) 方法，在 Lua [pcall()](http://www.lua.org/manual/5.1/manual.html#pdf-pcall) 或 [xpcall()](http://www.lua.org/manual/5.1/manual.html#pdf-xpcall) 中是不能使用的。甚至`for ... in ...` 小节的第一行在标准Lua 5.1解析器中都会报错 `attempt to yield across metamethod/C-call boundary`。 请使用 LuaJIT 2.x，它可以完美支持虚拟机唤醒，避免这些问题。

[Back to TOC](#table-of-contents)

> English source:

Lua Coroutine Yielding/Resuming
-------------------------------
* Because Lua's `dofile` and `require` builtins are currently implemented as C functions in both Lua 5.1 and LuaJIT 2.0/2.1, if the Lua file being loaded by `dofile` or `require` invokes [ngx.location.capture*](#ngxlocationcapture), [ngx.exec](#ngxexec), [ngx.exit](#ngxexit), or other API functions requiring yielding in the *top-level* scope of the Lua file, then the Lua error "attempt to yield across C-call boundary" will be raised. To avoid this, put these calls requiring yielding into your own Lua functions in the Lua file instead of the top-level scope of the file.
* As the standard Lua 5.1 interpreter's VM is not fully resumable, the methods [ngx.location.capture](#ngxlocationcapture), [ngx.location.capture_multi](#ngxlocationcapture_multi), [ngx.redirect](#ngxredirect), [ngx.exec](#ngxexec), and [ngx.exit](#ngxexit) cannot be used within the context of a Lua [pcall()](http://www.lua.org/manual/5.1/manual.html#pdf-pcall) or [xpcall()](http://www.lua.org/manual/5.1/manual.html#pdf-xpcall) or even the first line of the `for ... in ...` statement when the standard Lua 5.1 interpreter is used and the `attempt to yield across metamethod/C-call boundary` error will be produced. Please use LuaJIT 2.x, which supports a fully resumable VM, to avoid this.

[Back to TOC](#table-of-contents)