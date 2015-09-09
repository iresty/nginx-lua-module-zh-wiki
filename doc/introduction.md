介绍
----

`nginx.conf` 文件中各种 `*_by_lua` 和 `*_by_lua_file` 配置指令的作用是提供 Lua API 的网关。下面介绍的这些 Lua API 指令，只能在上述配置指令的环境中，通过用户 Lua 代码调用。

Lua 中使用的 API 以两个标准模块的形式封装：`ngx` 和 `ndk`。这两个模块在 ngx_lua 默认的全局作用域中，在 ngx_lua 指令中总是可用。

这两个模块可以被用在外部 Lua 模块中，例如：

```lua

 local say = ngx.say

 local _M = {}

 function _M.foo(a)
     say(a)
 end

 return _M
```

强烈不推荐使用 [package.seeall](http://www.lua.org/manual/5.1/manual.html#pdf-package.seeall) 标记，因为它有很多不好的副作用。

在外部 Lua 模块中也可以直接 require 这两个模块：

```lua

 local ngx = require "ngx"
 local ndk = require "ndk"
```

自 `v0.2.1rc19` 版开始可以 requrie 这两个模块。

用户代码中的网络 I/O 操作应该使用这些 Nginx Lua API 实现，否则 Nginx 的事件循环可能被阻塞，从而严重影响性能。相对小数据量的磁盘操作可以通过标准的 Lua `io` 库来实现，但大规模的文件读写如果可能应该避免，因为可能会严重阻塞 Nginx 进程。为获得最好性能，强烈建议将所有网络和磁盘 I/O 操作发送到 Nginx 的子请求中 (通过类似 [ngx.location.capture](#ngxlocationcapture) 的方法) 处理。


> Engslish Source

The various `*_by_lua` and `*_by_lua_file` configuration directives serve as gateways to the Lua API within the `nginx.conf` file. The Nginx Lua API described below can only be called within the user Lua code run in the context of these configuration directives.

The API is exposed to Lua in the form of two standard packages `ngx` and `ndk`. These packages are in the default global scope within ngx_lua and are always available within ngx_lua directives.

The packages can be introduced into external Lua modules like this:

```lua

 local say = ngx.say

 local _M = {}

 function _M.foo(a)
     say(a)
 end

 return _M
```

Use of the [package.seeall](http://www.lua.org/manual/5.1/manual.html#pdf-package.seeall) flag is strongly discouraged due to its various bad side-effects.

It is also possible to directly require the packages in external Lua modules:

```lua

 local ngx = require "ngx"
 local ndk = require "ndk"
```

The ability to require these packages was introduced in the `v0.2.1rc19` release.

Network I/O operations in user code should only be done through the Nginx Lua API calls as the Nginx event loop may be blocked and performance drop off dramatically otherwise. Disk operations with relatively small amount of data can be done using the standard Lua `io` library but huge file reading and writing should be avoided wherever possible as they may block the Nginx process significantly. Delegating all network and disk I/O operations to Nginx's subrequests (via the [ngx.location.capture](#ngxlocationcapture) method and similar) is strongly recommended for maximum performance.

[返回目录](#nginx-api-for-lua)