C 宏定义配置
----------------------
通过 OpenResty 或者 NGINX 内核方式构建该模块，你可以定义下面的 C 宏定义作为可选项提供给 C 编译器：

* `NGX_LUA_USE_ASSERT`
    声明后，将在ngx_lua C代码中开启断言。推荐用在调试或者测试版本中。启用后，它会引入额外一些（小的）运行时开销。在`v0.9.10`版本中首次引入此选项。

* `NGX_LUA_ABORT_AT_PANIC`
    当 Lua/LuaJIT 虚拟机出现panic错误时，ngx_lua默认会让当前的工作进程优雅退出。通过指定这个宏定义，ngx_lua将立即终止当前的 NGINX 工作进程（通常会生成一个core dump文件）。这个选项主要用来调试虚拟机的panic错误。在`v0.9.8`版本中首次引入此选项。
* `NGX_LUA_NO_FFI_API`
    去除 NGINX 中FFI-based Lua API需要的的纯 C 函数(例如 [lua-resty-core](https://github.com/openresty/lua-resty-core#readme) 所需要的)。开启这个宏可以让 NGINX 二进制代码更小。

在 NGINX 或者 OpenResty 启用一个或多个宏定义，只用传给`./configure`脚本几个额外的C编译选项。例如：

```
#    ./configure --with-cc-opt="-DNGX_LUA_USE_ASSERT -DNGX_LUA_ABORT_AT_PANIC"
```

[返回目录](#table-of-contents)
<!-- review by WenMing -->
> English source:

C Macro Configurations
----------------------

While building this module either via OpenResty or with the NGINX core, you can define the following C macros via the C compiler options:

* `NGX_LUA_USE_ASSERT`
    When defined, will enable assertions in the ngx_lua C code base. Recommended for debugging or testing builds. It can introduce some (small) runtime overhead when enabled. This macro was first introduced in the `v0.9.10` release.
* `NGX_LUA_ABORT_AT_PANIC`
    When the Lua/LuaJIT VM panics, ngx_lua will instruct the current nginx worker process to quit gracefully by default. By specifying this C macro, ngx_lua will abort the current nginx worker process (which usually result in a core dump file) immediately. This option is useful for debugging VM panics. This option was first introduced in the `v0.9.8` release.
* `NGX_LUA_NO_FFI_API`
    Excludes pure C API functions for FFI-based Lua API for NGINX (as required by [lua-resty-core](https://github.com/openresty/lua-resty-core#readme), for example). Enabling this macro can make the resulting binary code size smaller.

To enable one or more of these macros, just pass extra C compiler options to the `./configure` script of either NGINX or OpenResty. For instance,


    ./configure --with-cc-opt="-DNGX_LUA_USE_ASSERT -DNGX_LUA_ABORT_AT_PANIC"


[Back to TOC](#table-of-contents)
