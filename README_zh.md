Name
====

ngx_http_lua_module - 嵌入强有力的 Lua 到 Nginx HTTP 服务中。

*该模块不是随着 Nginx 源码进行分发。* 更多请看 [安装说明](#installation)。

Table of Contents
=================

* [Name](#name)
* [Status](#status)
* [Version](#version)
* [Synopsis](#synopsis)
* [Description](#description)
* [Typical Uses](#typical-uses)
* [Nginx Compatibility](#nginx-compatibility)
* [Installation](#installation)
    * [C Macro Configurations](#c-macro-configurations)
    * [Installation on Ubuntu 11.10](#installation-on-ubuntu-1110)
* [Community](#community)
    * [English Mailing List](#english-mailing-list)
    * [Chinese Mailing List](#chinese-mailing-list)
* [Code Repository](#code-repository)
* [Bugs and Patches](#bugs-and-patches)
* [Lua/LuaJIT bytecode support](#lualuajit-bytecode-support)
* [System Environment Variable Support](#system-environment-variable-support)
* [HTTP 1.0 support](#http-10-support)
* [Statically Linking Pure Lua Modules](#statically-linking-pure-lua-modules)
* [Data Sharing within an Nginx Worker](#data-sharing-within-an-nginx-worker)
* [Known Issues](#known-issues)
    * [TCP socket connect operation issues](#tcp-socket-connect-operation-issues)
    * [Lua Coroutine Yielding/Resuming](#lua-coroutine-yieldingresuming)
    * [Lua Variable Scope](#lua-variable-scope)
    * [Locations Configured by Subrequest Directives of Other Modules](#locations-configured-by-subrequest-directives-of-other-modules)
    * [Cosockets Not Available Everywhere](#cosockets-not-available-everywhere)
    * [Special Escaping Sequences](#special-escaping-sequences)
    * [Mixing with SSI Not Supported](#mixing-with-ssi-not-supported)
    * [SPDY Mode Not Fully Supported](#spdy-mode-not-fully-supported)
    * [Missing data on short circuited requests](#missing-data-on-short-circuited-requests)
* [TODO](#todo)
* [Changes](#changes)
* [Test Suite](#test-suite)
* [Copyright and License](#copyright-and-license)
* [See Also](#see-also)
* [Directives](#directives)
* [Nginx API for Lua](#nginx-api-for-lua)
* [Obsolete Sections](#obsolete-sections)
    * [Special PCRE Sequences](#special-pcre-sequences)

Status
======

生产版本可用

Version
=======

该文档描述的 ngx_lua [v0.9.20](https://github.com/openresty/lua-nginx-module/tags) 是2015年12月19号发布。

Synopsis
========

```nginx
 # 设置纯 Lua 扩展库的搜寻路径(';;' 是默认路径):
 lua_package_path '/foo/bar/?.lua;/blah/?.lua;;';

 # 设置 C 编写的 Lua 扩展模块的搜寻路径(也可以用 ';;'):
 lua_package_cpath '/bar/baz/?.so;/blah/blah/?.so;;';

 server {
     location /lua_content {
         # 通过 default_type 设置默认的 MIME 类型：
         default_type 'text/plain';

         content_by_lua_block {
             ngx.say('Hello,world!')
         }
     }

     location /nginx_var {
         # 通过 default_type 设置默认的 MIME 类型：
         default_type 'text/plain';

         # 试试访问 /nginx_var?a=hello,world
         content_by_lua_block {
             ngx.say(ngx.var.arg_a)
         }
     }

     location /request_body {
          client_max_body_size 50k;
          client_body_buffer_size 50k;

          content_by_lua_block {
             ngx.req.read_body()  -- explicitly read the req body
             local data = ngx.req.get_body_data()
             if data then
                 ngx.say("body data:")
                 ngx.print(data)
                 return
             end

             -- body may get buffered in a temp file:
             local file = ngx.req.get_body_file()
             if file then
                 ngx.say("body is in file ", file)
             else
                 ngx.say("no body found")
             end
         }
     }

     # 在子请求中直接发起 Lua 非阻塞 I/O 调用
     # （其实，更好的方式是使用 cosockets）
     location /lua {
         # 通过 default_type 设置默认的 MIME 类型：
         default_type 'text/plain';

         content_by_lua_block {
             local res = ngx.location.capture("/some_other_location")
             if res then
                 ngx.say("status: ", res.status)
                 ngx.say("body:")
                 ngx.print(res.body)
             end';
         }
     }

     location = /foo {
         rewrite_by_lua_block {
             res = ngx.location.capture("/memc",
                 { args = { cmd = "incr", key = ngx.var.uri } }
             )
         }

         proxy_pass http://blah.blah.com;
     }

     location = /mixed {
         rewrite_by_lua_file /path/to/rewrite.lua;
         access_by_lua_file /path/to/access.lua;
         content_by_lua_file /path/to/content.lua;
     }

     # 在代码中使用 NGINX 变量
     # 注意： NGINX 变量的内容一定要做仔细的过滤，否则会有很大的安全风险
     location ~ ^/app/([-_a-zA-Z0-9/]+) {
         set $path $1;
         content_by_lua_file /path/to/lua/app/root/$path.lua;
     }

     location / {
        lua_need_request_body on;

        client_max_body_size 100k;
        client_body_buffer_size 100k;

        access_by_lua_block {
            -- 检测客户端 IP 地址是否在我们的黑名单中
            if ngx.var.remote_addr == "132.5.72.3" then
                ngx.exit(ngx.HTTP_FORBIDDEN)
            end

            -- 检测客户端 URI 数据是否包含敏感词汇
            if ngx.var.uri and
                   string.match(ngx.var.request_body, "evil")
            then
                return ngx.redirect("/terms_of_use.html")
            end

            -- tests passed
        }

        # proxy_pass/fastcgi_pass/etc settings
     }
 }
```

[返回目录](#table-of-contents)

Description
===========

该模块通过标准 Lua5.1 解释器或 [LuaJIT 2.0/2.1](http://luajit.org/luajit.html)，把 Lua 嵌入到 NGINX 里面，
并利用 NGINX 子请求，把强大的 Lua 线程（Lua协程）混合到 NGINX 的事件模型中。

与 [Apache's mod_lua](https://httpd.apache.org/docs/trunk/mod/mod_lua.html)、[Lighttpd's mod_magnet](http://redmine.lighttpd.net/wiki/1/Docs:ModMagnet) 不同的是, 该模块中的 Lua 代码在网络上是 100% 非阻塞的。
包括该模块的 [Nginx API for Lua](#nginx-api-for-lua)，
上游请求服务如：MySQL、PostgreSQL、Memcached、Redis或upstream HTTP web 服务等，都是100%非阻塞的。

至少下面这些 Lua 库、NGINX 模块是可以与 ngx_lua 模块配合使用的：

* [lua-resty-memcached](https://github.com/openresty/lua-resty-memcached)
* [lua-resty-mysql](https://github.com/openresty/lua-resty-mysql)
* [lua-resty-redis](https://github.com/openresty/lua-resty-redis)
* [lua-resty-dns](https://github.com/openresty/lua-resty-dns)
* [lua-resty-upload](https://github.com/openresty/lua-resty-upload)
* [lua-resty-websocket](https://github.com/openresty/lua-resty-websocket)
* [lua-resty-lock](https://github.com/openresty/lua-resty-lock)
* [lua-resty-logger-socket](https://github.com/cloudflare/lua-resty-logger-socket)
* [lua-resty-lrucache](https://github.com/openresty/lua-resty-lrucache)
* [lua-resty-string](https://github.com/openresty/lua-resty-string)
* [ngx_memc](http://github.com/openresty/memc-nginx-module)
* [ngx_postgres](https://github.com/FRiCKLE/ngx_postgres)
* [ngx_redis2](http://github.com/openresty/redis2-nginx-module)
* [ngx_redis](http://wiki.nginx.org/HttpRedisModule)
* [ngx_proxy](http://nginx.org/en/docs/http/ngx_http_proxy_module.html)
* [ngx_fastcgi](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html)

几乎所有的 NGINX 模块都可以通过 [ngx.location.capture](#ngxlocationcapture) 或 [ngx.location.capture_multi](#ngxlocationcapture_multi)
 与 ngx_lua 模块完成调用，但推荐使用类似 `lua-resty-*` 库，而不是子请求访问 NGINX 上游模块，因为前者更加灵活并且内存效率更高。

在单个 NGINX worker 中，标准 Lua 或 LuaJIT 的实例在所有请求中是共享使用的，但每个请求上下文是通过轻量的 Lua 协程做到隔离的。

在 NGINX worker 进程中加载 Lua 模块，坚持小内存的使用，甚至在重负载下依然如此。

[返回目录](#table-of-contents)

Typical Uses
============

列举部分：

* 在 Lua 中揉和和处理各种不同的 NGINX 上游输出（proxy, drizzle, postgres, redis, memcached等）
* 在请求真正到达上游服务之前，Lua 可以随心所欲的做复杂的访问控制和安全检测
* 随心所欲的操控响应头里面的信息（通过 Lua）
* 从外部存储服务（比如 redis, memcached, mysql, postgresql）中获取后端信息，并用这些信息来实时选择哪一个后端来完成业务访问
* 在内容 handler 中随意编写复杂的 web 应用，使用同步但依然非阻塞的方式，访问后端数据库和其他存储
* 在 rewrite 阶段，通过 Lua 完成非常复杂的 URL dispatch
* 用 Lua 可以为 NGINX 子请求和任意location，实现高级缓存机制

本模块会把你带入一个拥有无限可能的服务端开发新世界，你可以把 NGINX 的各种功能进行自由拼接，
更重要的是，开发门槛并不高，这一切都是用强大轻巧的 Lua 语言来操控。

本模块的脚本有充分的灵活性，并且性能和原生 C 语言编程相比毫不逊色，无论是 CPU 时间还是内存占用方面。
当然这个需要你使用 LuaJIT 2.x。

其他脚本语言的实现通常很难达到类似性能。

Lua state（Lua VM instance）会被共享给单个 nginx worker 内所有的请求，从而达到最小化内存消耗。

[返回目录](#table-of-contents)

Nginx Compatibility
===================

最新模块版本和 NGINX 的兼容列表：

* 1.9.x (最后测试: 1.9.7)
* 1.8.x
* 1.7.x (最后测试: 1.7.10)
* 1.6.x

比 Nginx 1.6.0 更老的版本 *不* 再支持。

[返回目录](#table-of-contents)

Installation
============

强烈推荐使用[ngx_openresty](http://openresty.org)，它包含了 NGINX, ngx_lua, LuaJIT 2.0/2.1 (或者可选的标准 Lua 5.1解释器)，
还包含很多强劲、好用的 NGINX 模块。
使用一个简单的命令就可以完成基础安装：`./configure --with-luajit && make && make install`。

当然，ngx_lua 也可以手动的编译到 NGINX 中：

1. 安装LuaJIT 2.0 或 2.1 (推荐) 或 Lua 5.1 (Lua 5.2 暂时还*不支持*)。
LuaJIT可从 [The LuaJIT project website](http://luajit.org/download.html) 获取，
Lua 5.1可从 [Lua project website](http://www.lua.org/) 获取。
2. 下载最新版本的 ngx_devel_kit (NDK)开发模块 [这里](https://github.com/simpl/ngx_devel_kit/tags) 。
3. 下载最新版本的 ngx_lua [这里](https://github.com/openresty/lua-nginx-module/tags) 。
4. 下载最新版本的 NGINX [这里](http://nginx.org/) (查看 [Nginx 兼容列表](#nginx-compatibility))。

源码编译本模块：

```bash

 wget 'http://nginx.org/download/nginx-1.9.7.tar.gz'
 tar -xzvf nginx-1.9.7.tar.gz
 cd nginx-1.9.7/

 # tell nginx's build system where to find LuaJIT 2.0:
 export LUAJIT_LIB=/path/to/luajit/lib
 export LUAJIT_INC=/path/to/luajit/include/luajit-2.0

 # tell nginx's build system where to find LuaJIT 2.1:
 export LUAJIT_LIB=/path/to/luajit/lib
 export LUAJIT_INC=/path/to/luajit/include/luajit-2.1

 # or tell where to find Lua if using Lua instead:
 #export LUA_LIB=/path/to/lua/lib
 #export LUA_INC=/path/to/lua/include

 # Here we assume Nginx is to be installed under /opt/nginx/.
 ./configure --prefix=/opt/nginx \
         --with-ld-opt="-Wl,-rpath,/path/to/luajit-or-lua/lib" \
         --add-module=/path/to/ngx_devel_kit \
         --add-module=/path/to/lua-nginx-module

 make -j2
 make install
```

[返回目录](#table-of-contents)

C Macro Configurations
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

Installation on Ubuntu 11.10
----------------------------

注意：这里推荐使用 LuaJIT 2.0 或 LuaJIT 2.1 替换掉标准 Lua 5.1 解释器。

如果不得不使用标准的 Lua 5.1 解释器，在 Ubuntu 上使用这个命令完成安装：

```bash

 apt-get install -y lua5.1 liblua5.1-0 liblua5.1-0-dev
```

应该可以正确被安装，除了一个小"纠结"：

`liblua.so`库在 liblua5.1 包中已经发生改变，只能使用`liblua5.1.so`，并且需要被链接到`/usr/lib`，这样才可以在 configure 执行阶段被找到。

```bash

 ln -s /usr/lib/x86_64-linux-gnu/liblua5.1.so /usr/lib/liblua.so
```

[返回目录](#table-of-contents)

Community
=========

英文邮件列表
--------------------

英文邮件列表： [openresty-en](https://groups.google.com/group/openresty-en) 。

[返回目录](#table-of-contents)

中文邮件列表
--------------------

中文邮件列表： [openresty](https://groups.google.com/group/openresty) 。

[返回目录](#table-of-contents)

Code Repository
===============

本项目代码放在github上 [openresty/lua-nginx-module](https://github.com/openresty/lua-nginx-module)。

[返回目录](#table-of-contents)

Bugs and Patches
================

提交bug报告、想法或补丁，可按照下面两个方式：

1. 创建一个ticket [GitHub Issue Tracker](https://github.com/openresty/lua-nginx-module/issues)
1. 或者发到这里 [OpenResty community](#community).

[返回目录](#table-of-contents)

Lua/LuaJIT bytecode support
===========================

从`v0.5.0rc32` release 开始，所有 `*_by_lua_file` 的配置指令（比如 [content_by_lua_file](#content_by_lua_file)）
都支持直接加载 Lua 5.1 和 LuaJIT 2.0/2.1 的二进制字节码文件。

请注意，LuaJIT 2.0/2.1 生成的二进制格式与标准 Lua 5.1 解析器是不兼容的。
所以如果使用在 ngx_lua 下使用LuaJIT 2.0/2.1，那么 LuaJIT 兼容的二进制文件必须是下面这样生成的：

```bash

 /path/to/luajit/bin/luajit -b /path/to/input_file.lua /path/to/output_file.luac
```

`-bg` 选项是在 LuaJIT 字节码文件中包含调试信息。

```bash

 /path/to/luajit/bin/luajit -bg /path/to/input_file.lua /path/to/output_file.luac
```

对于`-b` 选项，请参考官方 LuaJIT 文档获取更多细节：

<http://luajit.org/running.html#opt_b>

同样的，由 LuaJIT 2.1 生成的字节码文件对于 LuaJIT 2.0 也是 *不* 兼容的，反之亦然。
第一次对于 LuaJIT 2.1 版本的字节支持，是在 v0.9.3 上完成的。

近似的，如果使用标准 Lua 5.1 解释器，Lua 的兼容字节码文件必须用 **luac** 命令行来生成：

```bash

 luac -o /path/to/output_file.luac /path/to/input_file.lua
```

与 LuaJIT 不太一样， Lua 5.1 的字节码文件是默认包含调试信息的。这里可以使用 `-s` 选项去掉调试信息：

```bash

 luac -s -o /path/to/output_file.luac /path/to/input_file.lua
```

在使用 LuaJIT 2.0/2.1 的 ngx_lua 实例中试图加载标准 Lua 5.1 字节码文件（反之亦然），将会在 error.log 中记录一条类似的错误信息：

>> [error] 13909#0: *1 failed to load Lua inlined code: bad byte-code header in /path/to/test_file.luac


使用 Lua`require` 和 `dofile` 这类原语加载字节码文件，应该总能按照预期工作。

[返回目录](#table-of-contents)

System Environment Variable Support
===================================

如果你想在 Lua 中通过标准 Lua API [os.getenv](http://www.lua.org/manual/5.1/manual.html#pdf-os.getenv)
来访问系统环境变量，例如`foo`，
那么你需要在你的 nginx.conf 中，通过[ env 指令](http://nginx.org/en/docs/ngx_core_module.html#env)，把这个环境变量列出来。
例如：

```nginx

 env foo;
```

[返回目录](#table-of-contents)

HTTP 1.0 support
================

HTTP 1.0 协议不支持分块输出，当响应体不为空时，需要在响应头中明确指定 `Content-Length`，以支持 HTTP 1.0 长连接。所以当把 [lua_http10_buffering](#lua_http10_buffering) 设置为 `on` 输出 HTTP 1.0 响应时，ngx_lua 将缓存 [ngx.say](#ngxsay) 和 [ngx.print](#ngxprint) 的所有输出，同时延迟发送响应头直到接收到所有输出内容。这时 ngx_lua 可以计算响应体的总长度，并为 HTTP 1.0 客户端创建一个正确的 `Content-Length` 响应头。如果在正在执行的 Lua 代码中设置 `Content-Length` 响应头，这种缓冲模式将被禁用，即使已经将 [lua_http10_buffering](#lua_http10_buffering) 指令设置为 `on`。

对于大型流式响应输出，禁用 [lua_http10_buffering](#lua_http10_buffering) 以最小化内存占用非常重要。

请注意，一些常见的 HTTP 性能测试工具，例如 `ab` 和 `http_load` 默认发送 HTTP 1.0 请求。要强制 `curl` 发送 HTTP 1.0 请求，使用 `-0` 选项。

[返回目录](#table-of-contents)

Statically Linking Pure Lua Modules
===================================

当使用 LuaJIT 2.x 时，可以把一个纯 Lua 字节码模块，静态链接到可运行的 Nginx 中。

首先你要使用`luajit` 的可执行程序， 把 `.lua` 的 Lua 模块编译成 `.o` 的目标文件（包含导出的字节码数据），
然后链接这些 `.o` 文件到你的 Nginx 构造环境。

用下面这个小例子来印证一下。这里我们的 `.lua` 文件使用 `foo.lua`：

```lua

 -- foo.lua
 local _M = {}

 function _M.go()
     print("Hello from foo")
 end

 return _M
```

我们把 `.lua` 文件编译成 `foo.o` 文件：

    /path/to/luajit/bin/luajit -bg foo.lua foo.o

这里重要的是 `.lua` 文件名，
它决定了这个模块在业务 Lua 中是如何使用的。
文件名 `foo.o` 除了扩展名是 `.o` 以外其他都不重要（只是用来告诉 `luajit` 使用什么格式输出）。
如果你想从输出的字节码中去掉 Lua 调试信息，
你可以用 `-b` 选项替代原有的 `-bg` 。

然后在构建 Nginx 或者 OpenResty时，
传给 `./configure` 脚本 `--with-ld-opt="foo.o"` 选项：

```bash

 ./configure --with-ld-opt="/path/to/foo.o" ...
```

最后，你可以在运行在 ngx_lua 中的任意 Lua 代码中调用：

```lua

 local foo = require "foo"
 foo.go()
```

并且，这段代码再也不会依赖外部的 `foo.lua` 文件，
因为它已经被编译到了 `nginx` 程序中。

在调用`require`时， 如果你想 Lua 模块名中使用点号，如下所示：

```lua

 local foo = require "resty.foo"
```

那么在你使用 `luajit` 命令行工具把他编译成 `.o` 文件之前，
你需要把 `foo.lua` 文件重命名为 `resty_foo.lua`。

把 `.lua` 文件编译成 `.o` 文件，和构建 NGINX + ngx_lua，
这两个过程中，使用完全相同版本的 LuaJIT 是至关重要的。

这是因为 LuaJIT 字节码格式在不同版本之间可能是不兼容的。
当字节码文件出现了不兼容情况，你将看到一行 Lua 运行时错误信息：没找到 Lua 模块。

当你拥有多个 `.lua` 文件需要链接，
你可以一次指明所有的 `.o` 文件，并赋给 `--with-ld-opt` 选项，参考：

```bash

 ./configure --with-ld-opt="/path/to/foo.o /path/to/bar.o" ...
```

如果你有非常多的 `.o` 文件，把这些文件的名字写到命令行中都不太可行，
这种情况下，对你的 `.o` 文件可以构建一个静态库（或者归档），参考：

```bash

 ar rcus libmyluafiles.a *.o
```

然后你就可以把 `myluafiles` 链接到你的 nginx 可执行程序中：

```bash

 ./configure \
     --with-ld-opt="-L/path/to/lib -Wl,--whole-archive -lmyluafiles -Wl,--no-whole-archive"
```

`/path/to/lib` 目录中应包含 `libmyluafiles.a` 文件。
应当指出的是，这里要添加链接选项 `--whole-archive`，
否则我们的归档将被跳过，因为在我们的归档没有导出 nginx 执行需要的函数符号。

[返回目录](#table-of-contents)

Data Sharing within an Nginx Worker
===================================

在同一个nginx worker进程处理的所有请求中共享数据，
需要将共享数据封装进一个 Lua 模块中，并使用 Lua 语言内置的 `require` 方法加载该模块，
之后就可以在 Lua 中操作共享数据了。
这种方法之所以可行，是因为(在同一个nginx worker中)加载模块的操作仅被执行一次，
所有的协程都会共享同一份拷贝(包括代码和数据)。
但是请注意，Lua的全局变量(注意，不是模块级变量)将因为“每个请求一个协程”的隔离要求而不被保持。

下面是一个完整的例子：

```lua
 -- mydata.lua
 local _M = {}

 local data = {
     dog = 3,
     cat = 4,
     pig = 5,
 }

 function _M.get_age(name)
     return data[name]
 end

 return _M
```

然后通过 nginx.conf 访问：

```nginx
 location /lua {
     content_by_lua '
         local mydata = require "mydata"
         ngx.say(mydata.get_age("dog"))
     ';
 }
```

例子中的 `mydata` 模块将只在第一个请求到达 `/lua` 的时候被加载运行，
之后同一个nginx worker进程处理的所有请求，都将使用此模块已经加载的实例，并共享实例中的数据，
直到 nginx 主进程接到 `HUP` 信号强制重新进行加载。
这种数据共享技术是基于本模块(ngx_lua)的高性能 Lua 应用的基础。

注意，这种数据共享方式是基于worker而不是基于服务器的。
也就是说，当 NGINX 主进程下面有多个 worker 进程时，数据共享不能跨越这些 worker 之间的进程边界。

一般来说，仅推荐使用这种方式共享只读数据。
当计算过程中没有非阻塞性I/O操作时(包括[ngx.sleep](#ngxsleep))，
你也可以在nginx worker进程内所有并发请求中共享可改变的数据。
只要你不把控制权交还给 nginx 事件循环以及 ngx_lua 的轻量级线程调度器(包括隐含的)，它们之间就不会有任何竞争。
因此，当你决定在worker中共享可变数据时，一定要非常小心。
错误的优化经常会导致在高负载时产生竞争，这种bug非常难以发现。

如果需要在服务器级别共享数据，请使用以下方法：

 1. 使用本模块提供的[ngx.shared.DICT](#ngxshareddict) API
 2. 使用单服务器单nginx worker进程(当使用多CPU或者多核CPU的服务器时不推荐)
 3. 使用类似 `memcached`, `redis`, `MySQL` 或 `PostgreSQL` 等数据共享机制。
 与本模块相关的[ngx_openresty软件包](http://openresty.org)包含了一系列相关的 NGINX 模块以及 Lua 库，
 提供与这些数据存储机制的交互界面。

[返回目录](#table-of-contents)

Known Issues
============

[返回目录](#table-of-contents)

TCP socket 连接操作遗留问题
-----------------------------------

[tcpsock:connect](#tcpsockconnect)方法，返回`success`但实际上连接故障，例如出现`Connection Refused`错误。

然而，后面尝试对cosocket对象的任何操作都将失败，并返回由失效连接操作所产生实际的错误状态消息。

这个问题是由于在Nginx的事件模型的局限性，似乎只影响Mac OS X系统。

[返回目录](#table-of-contents)

Lua 协程 Yielding/Resuming
-------------------------------

无论Lua 5.1 and LuaJIT 2.0/2.1，内建的 `dofile` 和 `require` 当前都是通过绑定 C 函数的方式，如果Lua文件的加载是`dofile` 或 `require` 并调用[ngx.location.capture*](#ngxlocationcapture), [ngx.exec](#ngxexec), [ngx.exit](#ngxexit), 或者Lua中其他 API 函数的 *top-level* 范围调用 yielding ，均会得到"attempt to yield across C-call boundary"的错误信息。为了避免这个情况，把需要调用yielding部分放到你自己的 Lua 函数中，这样在当前文件就不再是 *top-level* 范围。


对于标准 Lua 5.1 解析器的虚拟机唤醒支持是不完善的，[ngx.location.capture](#ngxlocationcapture), [ngx.location.capture_multi](#ngxlocationcapture_multi), [ngx.redirect](#ngxredirect), [ngx.exec](#ngxexec), 和 [ngx.exit](#ngxexit) 方法，在 Lua [pcall()](http://www.lua.org/manual/5.1/manual.html#pdf-pcall) 或 [xpcall()](http://www.lua.org/manual/5.1/manual.html#pdf-xpcall) 中是不能使用的。甚至`for ... in ...` 小节的第一行在标准Lua 5.1解析器中都会报错 `attempt to yield across metamethod/C-call boundary`。 请使用 LuaJIT 2.x，它可以完美支持虚拟机唤醒，避免这些问题。

[返回目录](#table-of-contents)

Lua Variable Scope
------------------

在代码中导入模块时应注意一些细节，推介使用如下格式：

```
 local xxx = require('xxx')
```

而非：

```
require('xxx')
```

理由如下：从设计上讲，全局环境的生命周期和一个Nginx的请求的生命周期是相同的。为了做到会话隔离，每个请求都有自己的Lua全局变量环境。Lua模块在第一次请求打到服务器上的时候被加载起来，通过`package.loaded`表内建的`require()`完成缓存，为后续代码复用。并且一些Lua模块内的`module()`存在边际问题，对加载完成的模块设置成全局表变量，但是这个全局变量在请求处理最后将被清空，并且每个后续请求都拥有自己（干净）的全局空间。所以它将因为访问`nil`值收到Lua异常。

一般来说，在ngx_lua的上下文中使用Lua全局变量真的不是什么好主意：  

1. 滥用全局变量的副作用会对并发场景产生副作用，比如当使用者把这些变量看作是本地变量的时候；
1. Lua的全局变量需要向上查找一个全局环境（只是一个Lua表），代价比较高；
1. 一些Lua的全局变量引用只是拼写错误，这会导致出错很难排查。

所以，我们极力推介在使用变量的时候总是使用local来定义以限定起生效范围是有理由的。

使用工具(lua-releng tool)[https://github.com/openresty/nginx-devel-utils/blob/master/lua-releng]查找你的Lua源文件：

    $ lua-releng     
    Checking use of Lua global variables in file lib/foo/bar.lua ...  
        1       [1489]  SETGLOBAL       7 -1    ; contains
        55      [1506]  GETGLOBAL       7 -3    ; setvar
        3       [1545]  GETGLOBAL       3 -4    ; varexpand

上述输出说明文件`lib/foo/bar.lua`的1489行写入一个名为`contains`的全局变量，1506行读取一个名为`setvar`的全局变量，1545行读取一个名为`varexpand`的全局变量，

这个工具能保证Lua模块中的局部变量全部是用local关键字定义过的，否则将会抛出一个运行时库。这样能阻止类似变量这样的资源的竞争。理由请参考(Data Sharing within an Nginx Worker)[http://wiki.nginx.org/HttpLuaModule#Data_Sharing_within_an_Nginx_Worker]

[返回目录](#table-of-contents)

Locations Configured by Subrequest Directives of Other Modules
--------------------------------------------------------------
[ngx.location.capture](#ngxlocationcapture) 和 [ngx.location.capture_multi](#ngxlocationcapture_multi) 指令无法抓取包含以下指令的 location： [add_before_body](http://nginx.org/en/docs/http/ngx_http_addition_module.html#add_before_body), [add_after_body](http://nginx.org/en/docs/http/ngx_http_addition_module.html#add_after_body), [auth_request](http://nginx.org/en/docs/http/ngx_http_auth_request_module.html#auth_request), [echo_location](http://github.com/openresty/echo-nginx-module#echo_location), [echo_location_async](http://github.com/openresty/echo-nginx-module#echo_location_async), [echo_subrequest](http://github.com/openresty/echo-nginx-module#echo_subrequest), 或 [echo_subrequest_async](http://github.com/openresty/echo-nginx-module#echo_subrequest_async) 。

```nginx

 location /foo {
     content_by_lua '
         res = ngx.location.capture("/bar")
     ';
 }
 location /bar {
     echo_location /blah;
 }
 location /blah {
     echo "Success!";
 }
```

```nginx

 $ curl -i http://example.com/foo
```

将不会如预期一样的工作。

[返回目录](#nginx-api-for-lua)

Cosockets Not Available Everywhere
----------------------------------

归咎于 `nginx` 内核的各种限制规则，cosocket API 在这些场景中是被禁的：
[set_by_lua*](#set_by_lua)， [log_by_lua*](#log_by_lua)， [header_filter_by_lua*](#header_filter_by_lua)， 和 [body_filter_by_lua](#body_filter_by_lua)。

cosocket在[init_by_lua*](#init_by_lua) 和 [init_worker_by_lua*](#init_worker_by_lua)小节中也是被禁的，但我们后面将会添加这些环境的支持，因为在nginx内核上是没有这个限制的（或者这个限制是可以被绕过的）。

这里有个绕路方法，前提是原始场景 *不* 需要等待cosocket结果。就是说，通过[ngx.timer.at](#ngxtimerat) API 创建一个零延迟的`timer`，在`timer`中完成cosocket的处理结果，用这种异步的方式进行协作。

[返回目录](#table-of-contents)

Special Escaping Sequences
--------------------------

**注意** 自引入 `*_by_lua_block {}` 配置指令，我们将不再被该问题折磨。

PCRE 的转义符号例如 `\d`,`\s` 以及 `\w` 等需要特别注意，因为在字符串语义中，反斜线字符 `\` 会被 Lua 语言解析器和 Nginx 配置文件解析器在执行前同时处理掉，所以以下代码片段将无法按预期运行：
```nginx
 # nginx.conf
 ? location /test {
 ?     content_by_lua '
 ?         local regex = "\d+"  -- 这里是错的!!
 ?         local m = ngx.re.match("hello, 1234", regex)
 ?         if m then ngx.say(m[0]) else ngx.say("not matched!") end
 ?     ';
 ? }
 # 结果为 "not matched!"
```

为避免这个问题，需要双重转义反斜线符号：

```nginx
 # nginx.conf
 location /test {
     content_by_lua '
         local regex = "\\\\d+"
         local m = ngx.re.match("hello, 1234", regex)
         if m then ngx.say(m[0]) else ngx.say("not matched!") end
     ';
 }
 # 结果为 "1234"
```

这里的 `\\\\d+`，先被 Nginx 配置文件解析器处理成 `\\d+` ，再被 Lua 语言解析器处理成 `\d+`，之后才被执行。

或者，正则表达式模板可以使用 Lua 字符串"长括号"语义写出，其语法形式为 `[[...]]`，在这种情况下，反斜线仅需为 Nginx 配置文件解析器转义一次。

```nginx
 # nginx.conf
 location /test {
     content_by_lua '
         local regex = [[\\d+]]
         local m = ngx.re.match("hello, 1234", regex)
         if m then ngx.say(m[0]) else ngx.say("not matched!") end
     ';
 }
 # 结果为 to "1234"
 ```

这里，`[[\\d+]]` 被 Nginx 配置文件解析器处理成 `[[\d+]]`，符合预期。

注意，当正则表达式模板中包括 `[...]` 序列时，Lua 语言中“更长的长括号”形式 `[=[...]=]` 是必要的。如果需要，可以将`[=[...]=]` 作为默认形式。

```nginx
 # nginx.conf
 location /test {
     content_by_lua '
         local regex = [=[[0-9]+]=]
         local m = ngx.re.match("hello, 1234", regex)
         if m then ngx.say(m[0]) else ngx.say("not matched!") end
     ';
 }
 # 结果为 "1234"
```

还有一种转义 PCRE 序列的方法是把 Lua 代码放到外部脚本文件中，通过各种 `*_by_lua_file` 指令执行。在这种方法中，反斜线仅被 Lua 语言解析器处理，因此只需要转义一次。

```lua
 -- test.lua
 local regex = "\\d+"
 local m = ngx.re.match("hello, 1234", regex)
 if m then ngx.say(m[0]) else ngx.say("not matched!") end
 -- 结果为 "1234"
```

在外部脚本文件中，PCRE 序列如果使用“长括号”形式 Lua 字符串，则无需修改。

```lua
 -- test.lua
 local regex = [[\d+]]
 local m = ngx.re.match("hello, 1234", regex)
 if m then ngx.say(m[0]) else ngx.say("not matched!") end
 -- 结果为 "1234"
```

[返回目录](#table-of-contents)

Mixing with SSI Not Supported
-----------------------------

在同样的Nginx请求混合SSI 与 ngx_lua 是完全不被支持的，只使用ngx_lua即可。使用SSI你可以做的任何事情，使用ngx_lua可以更好的完成，并且效能更棒。

[返回目录](#table-of-contents)

SPDY Mode Not Fully Supported
-----------------------------

一些ngx_lua提供的Lua APIs在Nginx的`SPDY`模式下确定不能工作：ngx.location.capture](#ngxlocationcapture)， [ngx.location.capture_multi](#ngxlocationcapture_multi)，和[ngx.req.socket](#ngxreqsocket)。

[返回目录](#table-of-contents)

Missing data on short circuited requests
----------------------------------------

Nginx提前销毁一个请求的可能（至少）：

* 400 (Bad Request)                 -- 错误请求
* 405 (Not Allowed)                 -- 不允许
* 408 (Request Timeout)             -- 请求超时
* 414 (Request URI Too Large)       -- 请求URI过大
* 494 (Request Headers Too Large)   -- 请求Headers过大
* 499 (Client Closed Request)       -- 客户端关闭请求
* 500 (Internal Server Error)       -- 内部服务错误
* 501 (Not Implemented)             -- 未实现

这意味着正常执行阶段被跳过，如重写或访问阶段。这也意味着，后面的所有阶段，例如
[log_by_lua](#log_by_lua)，将无法获得在这个阶段存放的普通信息。

[返回目录](#table-of-contents)

TODO
====

* cosocket：实现LuaSocket非连接的UDP API。
* 实现普通的TCP服务替代HTTP服务，并支持Lua代码。例如：

```lua

 tcp {
     server {
         listen 11212;
         handler_by_lua '
             -- custom Lua code implementing the special TCP server...
         ';
     }
 }
```

* 实现普通的UDP服务替代HTTP服务，并支持Lua代码。例如：

```lua

 udp {
     server {
         listen 1953;
         handler_by_lua '
             -- custom Lua code implementing the special UDP server...
         ';
     }
 }
```

* shm：实现一个"shared queue API"，对[shared dict](#lua_shared_dict) 补充API。
* cosocket：在[init_by_lua*](#init_by_lua)添加支持。
* cosocket：对于流式cosocket，实现bind()方法。
* cosocket：基于池子的后端并发连接控制，当后端并发超过它的连接池限制，实现自动排队的`connect`。
* cosocket：查看合并aviramc's的[patch](https://github.com/openresty/lua-nginx-module/pull/290)，添加`bsdrecv`方法。
* 添加新的API函数完成标准`add_header`配置功能。
* [ngx.re](#ngxrematch) API: 当没有成功匹配结果时，返回结果使用`false`替换`nil`，这样我们可以避免数组的"holes"问题。
* 查看、合并 Jader H. Silva 的 补丁 `ngx.re.split()` 。
* 查看、合并 Jader H. Silva 的 补丁 `ngx.re.split()`.
* 查看、合并 vadim-pavlov 的 补丁，给[ngx.location.capture](#ngxlocationcapture)添加 `extra_headers` 选项
* 使用 `ngx_hash_t` 去优化内建的 header 查找，涉及[ngx.req.set_header](#ngxreqset_header), [ngx.header.HEADER](#ngxheaderheader) 等。
* cosocket连接池溢出，支持配置选项定义不同策略
* 添加新的小节，当nginx关闭时执行一段代码。
* 添加`ignore_resp_headers`, `ignore_resp_body`, 和 `ignore_resp`选项给[ngx.location.capture](#ngxlocationcapture)、[ngx.location.capture_multi](#ngxlocationcapture_multi)，对于用户提升微小性能。
* add automatic Lua code time slicing support by yielding and resuming the Lua VM actively via Lua's debug hooks.
<!-- todo 没懂，后面看看怎么翻译  -->
* 添加`stat`类似[mod_lua](https://httpd.apache.org/docs/trunk/mod/mod_lua.html)。

[返回目录](#table-of-contents)

Changes
=======

该模块每个发行版本的变更，都记录在ngx_openresty绑定的变更日志中：

<http://openresty.org/#Changes>

[返回目录](#table-of-contents)

Test Suite
==========

为了运行测试套件，依赖下面这些条件：

* Nginx version >= 1.4.2

* Perl modules:
    * Test::Nginx: <https://github.com/openresty/test-nginx>

* Nginx 模块:
    * [ngx_devel_kit](https://github.com/simpl/ngx_devel_kit)
    * [ngx_set_misc](https://github.com/openresty/set-misc-nginx-module)
    * [ngx_auth_request](http://mdounin.ru/files/ngx_http_auth_request_module-0.2.tar.gz) (this is not needed if you're using Nginx 1.5.4+.
    * [ngx_echo](https://github.com/openresty/echo-nginx-module)
    * [ngx_memc](https://github.com/openresty/memc-nginx-module)
    * [ngx_srcache](https://github.com/openresty/srcache-nginx-module)
    * ngx_lua (i.e., this module)
    * [ngx_lua_upstream](https://github.com/openresty/lua-upstream-nginx-module)
    * [ngx_headers_more](https://github.com/openresty/headers-more-nginx-module)
    * [ngx_drizzle](https://github.com/openresty/drizzle-nginx-module)
    * [ngx_rds_json](https://github.com/openresty/rds-json-nginx-module)
    * [ngx_coolkit](https://github.com/FRiCKLE/ngx_coolkit)
    * [ngx_redis2](https://github.com/openresty/redis2-nginx-module)

`configure`时添加的这些模块顺序是非常重要的。因为在filter链中不同的过滤模块位置决定最终输出。正确的添加顺序如上所示。

* 第三方 Lua 库:
    * [lua-cjson](http://www.kyne.com.au/~mark/software/lua-cjson.php)

* 应用:
    * mysql: 创建数据库 'ngx_test', 对用户'ngx_test'赋予所有权限，密码也是'ngx_test'。
    * memcached: 监听默认端口，11211.
    * redis: 监听默认端口, 6379.

查看[developer build script](https://github.com/openresty/lua-nginx-module/blob/master/util/build2.sh)内容，在搭建测试环境时确定更多细节。

在默认的测试模式下启动测试套件：

    cd /path/to/lua-nginx-module
    export PATH=/path/to/your/nginx/sbin:$PATH
    prove -I/path/to/test-nginx/lib -r t

运行指定的测试文件：

    cd /path/to/lua-nginx-module
    export PATH=/path/to/your/nginx/sbin:$PATH
    prove -I/path/to/test-nginx/lib t/002-content.t t/003-errors.t

在一个特别的测试文件中，运行指定的测试块，对你需要进行块测试部分添加一行`--- ONLY`信息，并使用`prove`工具运行这个`.t`文件。

此外，还有其他各种测试方式，基于mockeagain， valgrind 等。参考[Test::Nginx documentation](http://search.cpan.org/perldoc?Test::Nginx)，有更多不同高级测试方式的资料。也可以看看在Amazon EC2的Nginx集群测试报告 <http://qa.openresty.org.> 。

[返回目录](#table-of-contents)

Copyright and License
=====================

该模块是根据BSD许可证授权。

Copyright (C) 2009-2015, by Xiaozhe Wang (chaoslawful) <chaoslawful@gmail.com>.

Copyright (C) 2009-2015, by Yichun "agentzh" Zhang (章亦春) <agentzh@gmail.com>, CloudFlare Inc.

版权所有。

在源代码和二进制形式的二次发行和使用，无论修改与否，允许的前提是满足以下条件：

* 二次发行源代码必须保留以上版权声明、条件列表和下面的免责声明。
* 二次发行二进制形式必须复制上述版权声明、此条件列表和文档和或在分发提供的其他材料中的下列免责声明。

```
THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```

[返回目录](#table-of-contents)

See Also
========

* [lua-resty-memcached](https://github.com/openresty/lua-resty-memcached) 基于ngx_lua cosocket的库。
* [lua-resty-redis](https://github.com/openresty/lua-resty-redis) 基于ngx_lua cosocket的库。
* [lua-resty-mysql](https://github.com/openresty/lua-resty-mysql) 基于ngx_lua cosocket的库。
* [lua-resty-upload](https://github.com/openresty/lua-resty-upload) 基于ngx_lua cosocket的库。
* [lua-resty-dns](https://github.com/openresty/lua-resty-dns) 基于ngx_lua cosocket的库。
* [lua-resty-websocket](https://github.com/openresty/lua-resty-websocket) 提供WebSocket的客户端、服务端，基于ngx_lua cosocket的库。
* [lua-resty-string](https://github.com/openresty/lua-resty-string) 基于 [LuaJIT FFI](http://luajit.org/ext_ffi.html) 的库。
* [lua-resty-lock](https://github.com/openresty/lua-resty-lock) 一个简单非阻塞lock API库。
* [lua-resty-cookie](https://github.com/cloudflare/lua-resty-cookie) HTTP cookie 的库。
* [基于URI参数，发起不同的MySQL查询](http://openresty.org/#RoutingMySQLQueriesBasedOnURIArgs)
* [基于Redsi和Lua的动态路由](http://openresty.org/#DynamicRoutingBasedOnRedis)
* [使用LuaRocks与ngx_lua](http://openresty.org/#UsingLuaRocks)
* [ngx_lua的介绍信息](https://github.com/openresty/lua-nginx-module/wiki/Introduction)
* [ngx_devel_kit](https://github.com/simpl/ngx_devel_kit)
* [echo-nginx-module](http://github.com/openresty/echo-nginx-module)
* [drizzle-nginx-module](http://github.com/openresty/drizzle-nginx-module)
* [postgres-nginx-module](https://github.com/FRiCKLE/ngx_postgres)
* [memc-nginx-module](http://github.com/openresty/memc-nginx-module)
* [ngx_openresty 捆绑包](http://openresty.org)
* [Nginx Systemtap 工具箱](https://github.com/openresty/nginx-systemtap-toolkit)

[返回目录](#table-of-contents)

Directives
==========

* [lua_use_default_type](#lua_use_default_type)
* [lua_code_cache](#lua_code_cache)
* [lua_regex_cache_max_entries](#lua_regex_cache_max_entries)
* [lua_regex_match_limit](#lua_regex_match_limit)
* [lua_package_path](#lua_package_path)
* [lua_package_cpath](#lua_package_cpath)
* [init_by_lua](#init_by_lua)
* [init_by_lua_block](#init_by_lua_block)
* [init_by_lua_file](#init_by_lua_file)
* [init_worker_by_lua](#init_worker_by_lua)
* [init_worker_by_lua_block](#init_worker_by_lua_block)
* [init_worker_by_lua_file](#init_worker_by_lua_file)
* [set_by_lua](#set_by_lua)
* [set_by_lua_block](#set_by_lua_block)
* [set_by_lua_file](#set_by_lua_file)
* [content_by_lua](#content_by_lua)
* [content_by_lua_block](#content_by_lua_block)
* [content_by_lua_file](#content_by_lua_file)
* [rewrite_by_lua](#rewrite_by_lua)
* [rewrite_by_lua_block](#rewrite_by_lua_block)
* [rewrite_by_lua_file](#rewrite_by_lua_file)
* [access_by_lua](#access_by_lua)
* [access_by_lua_block](#access_by_lua_block)
* [access_by_lua_file](#access_by_lua_file)
* [header_filter_by_lua](#header_filter_by_lua)
* [header_filter_by_lua_block](#header_filter_by_lua_block)
* [header_filter_by_lua_file](#header_filter_by_lua_file)
* [body_filter_by_lua](#body_filter_by_lua)
* [body_filter_by_lua_block](#body_filter_by_lua_block)
* [body_filter_by_lua_file](#body_filter_by_lua_file)
* [log_by_lua](#log_by_lua)
* [log_by_lua_block](#log_by_lua_block)
* [log_by_lua_file](#log_by_lua_file)
* [balancer_by_lua_block](#balancer_by_lua_block)
* [balancer_by_lua_file](#balancer_by_lua_file)
* [lua_need_request_body](#lua_need_request_body)
* [ssl_certificate_by_lua_block](#ssl_certificate_by_lua_block)
* [ssl_certificate_by_lua_file](#ssl_certificate_by_lua_file)
* [lua_shared_dict](#lua_shared_dict)
* [lua_socket_connect_timeout](#lua_socket_connect_timeout)
* [lua_socket_send_timeout](#lua_socket_send_timeout)
* [lua_socket_send_lowat](#lua_socket_send_lowat)
* [lua_socket_read_timeout](#lua_socket_read_timeout)
* [lua_socket_buffer_size](#lua_socket_buffer_size)
* [lua_socket_pool_size](#lua_socket_pool_size)
* [lua_socket_keepalive_timeout](#lua_socket_keepalive_timeout)
* [lua_socket_log_errors](#lua_socket_log_errors)
* [lua_ssl_ciphers](#lua_ssl_ciphers)
* [lua_ssl_crl](#lua_ssl_crl)
* [lua_ssl_protocols](#lua_ssl_protocols)
* [lua_ssl_trusted_certificate](#lua_ssl_trusted_certificate)
* [lua_ssl_verify_depth](#lua_ssl_verify_depth)
* [lua_http10_buffering](#lua_http10_buffering)
* [rewrite_by_lua_no_postpone](#rewrite_by_lua_no_postpone)
* [access_by_lua_no_postpone](#access_by_lua_no_postpone)
* [lua_transform_underscores_in_response_headers](#lua_transform_underscores_in_response_headers)
* [lua_check_client_abort](#lua_check_client_abort)
* [lua_max_pending_timers](#lua_max_pending_timers)
* [lua_max_running_timers](#lua_max_running_timers)

[返回目录](#table-of-contents)

lua_use_default_type
--------------------
**语法:**: *lua_use_default_type on | off*

**默认**: *lua_use_default_type on*

**环境**: *http, server, location, location if*

指定响应头中 Content-Type 的默认值，是否使用 default_type 指令中指明的 MIME 类型。
如果你的 Lua 请求处理程序不想要默认的 Content-Type 响应头，可以关闭这个指令。

默认情况下该指令被打开。

这个指令在 0.9.1 版本中首次引入。

[返回目录](#table-of-contents)

lua_code_cache
--------------
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

[返回目录](#directives)

lua_regex_cache_max_entries
---------------------------
**语法:** *lua_regex_cache_max_entries &lt;num&gt;*

**默认:** *lua_regex_cache_max_entries 1024*

**环境:** *http*

在工作进程级别，指定正则表达式编译缓存允许的最大数目。

正则表达式被用于[ngx.re.match](#ngxrematch)， [ngx.re.gmatch](#ngxregmatch)， [ngx.re.sub](#ngxresub)， 和 [ngx.re.gsub](#ngxregsub)，如果使用`o` (既，编译一次的标识)正则选项，将会被缓存。

允许的默认数量为1024，当达到此限制，新的正则表达式将不会被缓存（就像没指定`o`选项一样），将会有且仅只有一个告警信息在 `error.log` 文件中：

    2011/08/27 23:18:26 [warn] 31997#0: *1 lua exceeding regex cache max entries (1024), ...

对于部分正则表达式（字符串的各种替换，如 [ngx.re.sub](#ngxresub) 和 [ngx.re.gsub](#ngxregsub)），不要使用`o`选项，这类正则每次都不一样，缓存无法被利用。这样我们可以避免撞上最大数的限制。

[返回目录](#directives)

lua_regex_match_limit
---------------------
**语法:** *lua_regex_match_limit &lt;num&gt;*

**默认:** *lua_regex_match_limit 0*

**环境:** *http*

指定执行[ngx.re API](#ngxrematch)时使用PCRE库的"匹配限制"。引述PCRE手册，“the limit ... has the effect of limiting the amount of backtracking that can take place.”。

当触发了这个限制，在Lua代码的[ngx.re API](#ngxrematch)函数，将返回错误信息"pcre_exec() failed: -8"。

当设置了限制为0，将使用编译PCRE库的默认"match limit"。这也是这个配置的默认值。

这个指令是在`v0.8.5`发行版被首次引入的。

[返回目录](#directives)

lua_package_path
----------------

**语法:** *lua_package_path &lt;lua-style-path-str&gt;*

**默认:** *The content of LUA_PATH environ variable or Lua's compiled-in defaults.*

**环境:** *http*

设置[set_by_lua](#set_by_lua)，[content_by_lua](#content_by_lua) 和 其他脚本对Lua模块的查找路径。 路径字符串是标准Lua路径格式，`;;` 可被用来代表原始搜索路径。

从`v0.5.0rc29`发行版开始，特殊符号`$prefix` 或 `${prefix}`可用于搜索路径字符串中。`server prefix`的值，通常是由Nginx服务启动时的`-p PATH`命令行决定的。

[Back to TOC](#directives)

lua_package_cpath
-----------------

**语法:** *lua_package_cpath &lt;lua-style-cpath-str&gt;*

**默认:** *The content of LUA_CPATH environment variable or Lua's compiled-in defaults.*

**环境:** *http*

设置[set_by_lua](#set_by_lua)，[content_by_lua](#content_by_lua) 和 其他脚本对Lua C模块的查找路径。 cpath路径字符串是标准Luacpath路径格式，`;;` 可被用来代表原始cpath路径。

从`v0.5.0rc29`发行版开始，特殊符号`$prefix` 或 `${prefix}`可用于搜索路径字符串中。`server prefix`的值，通常是由Nginx服务启动时的`-p PATH`命令行决定的。

[返回目录](#directives)

init_by_lua
-----------

**语法:** *init_by_lua &lt;lua-script-str&gt;*

**环境:** *http*

**阶段:** *loading-config*

**注意** 自从 `v0.9.17` 版本, 不鼓励使用该指令，应使用新的 [init_by_lua_block](#init_by_lua_block) 指令进行替代。

当Nginx master进程（如果有）加载Nginx配置文件时，在全局的Lua虚拟机上运行`<lua-script-str>`指定的Lua代码。

当Nginx收到`HUP`信号并开始重新加载配置文件，Lua虚拟机将重新创建并且`init_by_lua`在新的Lua虚拟机中再次执行。为防止[lua_code_cache](#lua_code_cache)指令是关闭的（默认打开），对于这个场景`init_by_lua`将在每个请求之上运行，因为在这个场景中，每个请求都会创建新的Lua虚拟机，他们都是独立存在。

通常，你可以在服务启动时注册Lua全局变量或预加载Lua模块。这是个预加载Lua模块的示例代码：

```nginx

 init_by_lua 'cjson = require "cjson"';

 server {
     location = /api {
         content_by_lua '
             ngx.say(cjson.encode({dog = 5, cat = 6}))
         ';
     }
 }
```

你也可以在这个阶段初始化[lua_shared_dict](#lua_shared_dict)共享内存内容。这里是示例代码：

```nginx

 lua_shared_dict dogs 1m;

 init_by_lua '
     local dogs = ngx.shared.dogs;
     dogs:set("Tom", 56)
 ';

 server {
     location = /api {
         content_by_lua '
             local dogs = ngx.shared.dogs;
             ngx.say(dogs:get("Tom"))
         ';
     }
 }
```

需要注意，当配置重载（例如`HUP`信号）时[lua_shared_dict](#lua_shared_dict)的共享数据是不会被清空。这种情况下，如果你不想在`init_by_lua`再次再初始化你的共享数据，你需要设置一个个性标识并且在你的`init_by_lua`代码中每次都做检查。

因为在这个指令的Lua代码执行是在Nginx fork他的工作进程（如果有），加载的数据和代码将被友好[Copy-on-write (COW)](http://en.wikipedia.org/wiki/Copy-on-write)特性提供给其他所有工作进程，从而节省了大量内存。

不要在这个上下文中初始化你自己的Lua全局变量，因为全局变量的使用有性能损失并会带来全局命名污染（可以查看[Lua 变量范围](#lua-variable-scope)获取更多细节）。推荐的方式是正确使用[Lua模块](http://www.lua.org/manual/5.1/manual.html#5.3) 文件（不要使用标准Lua函数[module()](http://www.lua.org/manual/5.1/manual.html#pdf-module)来定义Lua模块，因为它同样对全局命名空间有污染），在`init_by_lua` 或 其他上下文中调用[require()](http://www.lua.org/manual/5.1/manual.html#pdf-require)来加载你自己的模块文件。[require()](http://www.lua.org/manual/5.1/manual.html#pdf-require)会在全局Lua注册的`package.loaded`表中缓存Lua模块，所以在整个Lua虚拟机实例中你的模块将只会加载一次。

在这个上下文中，只有一小部分的[Nginx Lua API](#nginx-api-for-lua)是被支持的：


* 记录日志的APIs：[ngx.log](#ngxlog) 和 [print](#print)
* 共享内存字典APIs：[ngx.shared.DICT](#ngxshareddict)

在这个上下文中，根据用户的后续需要，将会支持更多的Nginx Lua APIs。

基本上，在这个上下文中，你可以保守使用Lua库完成阻塞I/O调用，因为在master进程的阻塞调用在服务的启动过程中是完全没问题的。进一步说在配置加载阶段，Nginx核心就是阻塞 I/O 方式处理的（至少在解析上游主机名称时）。

你应该非常小心，在这种情况下注册的Lua代码潜在的安全漏洞，因为Nginx的主进程经常是'root`帐户下运行。

这个指令是`v0.5.5`版本中第一次引入的。

[返回目录](#directives)

init_by_lua_block
-----------------

**语法:** *init_by_lua_block { lua-script }*

**环境:** *http*

**阶段:** *loading-config*

与 [init_by_lua](#init_by_lua) 指令相似，只不过该指令在一对括号（`{}`）中直接内嵌 Lua 代码，替代之前 NGINX 的字符串（需要特殊字符转义）。

例如：

```nginx

 init_by_lua_block {
     print("I need no extra escaping here, for example: \r\nblah")
 }
```

该指令在 `v0.9.17` 版本首次引入。

[返回目录](#directives)

init_by_lua_file
----------------

**语法:** *init_by_lua_file &lt;path-to-lua-script-file&gt;*

**环境:** *http*

**阶段:** *loading-config*

与[init_by_lua](#init_by_lua)等价，通过`<path-to-lua-script-file>`指定文件的Lua 代码 或 [Lua/LuaJIT 字节码](#lualuajit-bytecode-support)来执行。

当给定了一个相对路径如`foo/bar.lua`，它将会被转换成绝对路径，前面增加的部分路径是Nginx服务启动时通过命令行选项`-p PATH`决定的`server prefix`。

该指令在`v0.5.5`发行版第一次被引入。

[返回目录](#directives)

init_worker_by_lua
------------------

**语法:** *init_worker_by_lua &lt;lua-script-str&gt;*

**环境:** *http*

**阶段:** *starting-worker*

**注意** 自从 `v0.9.17` 版本, 使用该指令是 *不爽* 的，应使用新的 [init_worker_by_lua_block](#init_worker_by_lua_block) 指令进行替代。

开启master进程模式，Nginx工作进程启动时执行指定的Lua代码。关闭master模式，将在[init_by_lua*](#init_by_lua)后直接运行。

这个指令经常被用来创建单进程的反复执行定时器（通过[ngx.timer.at](#ngxtimerat) Lua API创建），可以是后端服务健康检查，也可以是其他定时的日常工作。下面是个例子：

```nginx

 init_worker_by_lua '
     local delay = 3  -- in seconds
     local new_timer = ngx.timer.at
     local log = ngx.log
     local ERR = ngx.ERR
     local check

     check = function(premature)
         if not premature then
             -- do the health check or other routine work
             local ok, err = new_timer(delay, check)
             if not ok then
                 log(ERR, "failed to create timer: ", err)
                 return
             end
         end
     end

     local ok, err = new_timer(delay, check)
     if not ok then
         log(ERR, "failed to create timer: ", err)
         return
     end
 ';
```

这个指令是在`v0.9.5`发行版第一次引入。

[返回目录](#directives)

init_worker_by_lua_block
------------------------

**语法:** *init_worker_by_lua_block { lua-script }*

**环境:** *http*

**阶段:** *starting-worker*

与 [init_worker_by_lua](#init_worker_by_lua) 指令相似，只不过该指令在一对括号（`{}`）中直接内嵌 Lua 代码，替代之前 NGINX 的字符串（需要特殊字符转义）。

例如：

```nginx

 init_worker_by_lua_block {
     print("I need no extra escaping here, for example: \r\nblah")
 }
```

该指令在 `v0.9.17` 版本首次引入。

[Back to TOC](#directives)

init_worker_by_lua_file
-----------------------

**语法:** *init_worker_by_lua_file &lt;lua-file-path&gt;*

**环境:** *http*

**阶段:** *starting-worker*

与[init_worker_by_lua](#init_worker_by_lua)等价，通过`<lua-file-path>`指定的Lua 或 [Lua/LuaJIT 字节码](#lualuajit-bytecode-support)文件来执行。

该指令是在`v0.9.5`发行版第一次引入。

[返回目录](#directives)

set_by_lua
----------

**语法:** *set_by_lua $res &lt;lua-script-str&gt; [$arg1 $arg2 ...]*

**环境:** *server, server if, location, location if*

**阶段:** *rewrite*

**注意** 自从 `v0.9.17` 版本, 使用该指令是 *不爽* 的，应使用新的 [set_by_lua_block](#set_by_lua_block) 指令进行替代。

使用可选的输入参数`$arg1 $arg2 ...`，执行指定的代码`<lua-script-str>`，并返回字符串结果到`$res`。

`<lua-script-str>`的代码可以做[API调用](#nginx-api-for-lua)，并能从`ngx.arg`表中获取输入参数（下标起始值是`1`并顺序增长）。

该指令被设计为执行短小、快速的代码块，因为代码执行时Nginx的事件循环是被阻塞的。因此应避免耗时的代码处理。

这个指令是通过挂载自定义命令到标准[ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)模块列表来实现。因为模块[ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)中是不支持非阻塞I/O，所以在本指令中，是无法yield当前Lua的“轻线程”。

在`set_by_lua`的上下文中，至少下列API 函数目前是被禁止的：

* 输出 API 函数 (例如 [ngx.say](#ngxsay) 和 [ngx.send_headers](#ngxsend_headers))
* 控制 API 函数 (例如 [ngx.exit](#ngxexit))
* 子请求 API 函数 (例如 [ngx.location.capture](#ngxlocationcapture) 和 [ngx.location.capture_multi](#ngxlocationcapture_multi))
* Cosocket API 函数 (例如 [ngx.socket.tcp](#ngxsockettcp) 和 [ngx.req.socket](#ngxreqsocket))
* 休眠 API 函数 [ngx.sleep](#ngxsleep)

额外注意的，本指令一次只能写回一个值到一个Nginx变量。尽管如此，可以使用[ngx.var.VARIABLE](#ngxvarvariable)接口绕过这个限制。

```nginx

 location /foo {
     set $diff ''; # we have to predefine the $diff variable here

     set_by_lua $sum '
         local a = 32
         local b = 56

         ngx.var.diff = a - b;  -- write to $diff directly
         return a + b;          -- return the $sum value normally
     ';

     echo "sum = $sum, diff = $diff";
 }
```

这个指令可以自由的与其他指令模块混合使用，如[ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html), [set-misc-nginx-module](http://github.com/openresty/set-misc-nginx-module), 和 [array-var-nginx-module](http://github.com/openresty/array-var-nginx-module)。所有这些指令的执行顺序，将和他们配置文件中出现的顺序一致。


```nginx

 set $foo 32;
 set_by_lua $bar 'return tonumber(ngx.var.foo) + 1';
 set $baz "bar: $bar";  # $baz == "bar: 33"
```

自 `v0.5.0rc29` 版本开始，本指令的 `<lua-script-str>` 参数中不再支持内联 Nginx 变量，所以可以直接使用 $ 字符作为其字面值。

这个指令需要[ngx_devel_kit](https://github.com/simpl/ngx_devel_kit)模块。

[返回目录](#directives)

set_by_lua_block
----------------

**语法:** *set_by_lua_block $res { lua-script }*

**环境:** *server, server if, location, location if*

**阶段:** *rewrite*

与 [set_by_lua](#set_by_lua) 指令相似，以下情况除外：

1. 该指令在一对括号（`{}`）中直接内嵌 Lua 代码，替代之前 NGINX 的字符串（需要特殊字符转义）
1. 该指令和 [set_by_lua](#set_by_lua) 一样，在 Lua 脚本的后面不支持额外参数

例如：

```nginx

 set_by_lua_block $res { return 32 + math.cos(32) }
 # $res now has the value "32.834223360507" or alike.
```

在 Lua 代码块中无需任何的特殊转义。

该指令在 `v0.9.17` 版本首次引入。

[返回目录C](#directives)

set_by_lua_file
---------------
**syntax:** *set_by_lua_file $res &lt;path-to-lua-script-file&gt; [$arg1 $arg2 ...]*

**context:** *server, server if, location, location if*

**phase:** *rewrite*

除了通过文件`<path-to-lua-script-file>`的内容指定 Lua 代码外，该指令与[set_by_lua](#set_by_lua)是等价的，该指令从`v0.5.0rc32`开始支持[Lua/LuaJIT bytecode](#lualuajit-bytecode-support)的执行。

对于该指令，对`<path-to-lua-script-file>`的字符串参数支持内联 Nginx 变量。但必须要额外注意注入攻击。

当给定了一个相对路径如`foo/bar.lua`，它将会被转换成绝对路径，前面增加的部分路径是 Nginx 服务启动时通过命令行选项`-p PATH`决定的`server prefix`。

当 Lua 代码缓存开启（默认），用户代码在第一次请求时完成加载（只有一次）并缓存，当 Lua 文件被修改时，每次都要对 Nginx 配置进行重新加载。Lua 代码缓存是可以暂时被禁用，通过开关[lua_code_cache](#lua_code_cache)在`nginx.conf`中设置为`off`，这样就可以避免反复重新加载 Nginx。

该指令需要 [ngx_devel_kit](https://github.com/simpl/ngx_devel_kit) 模块。

[返回目录](#directives)

content_by_lua
--------------

**语法：** *content_by_lua &lt;lua-script-str&gt;*

**环境：** *location, location if*

**阶段：** *content*

**注意** 自从 `v0.9.17` 版本, 使用该指令是 *不爽* 的，应使用新的 [content_by_lua_block](#content_by_lua_block) 指令进行替代。

作为"内容处理程序"，为每一个请求执行`<lua-script-str>`中指定的Lua代码。

这些 Lua 代码可以调用[全部 API](#nginx-api-for-lua)，并作为一个新的协程，在一个独立的全局环境中执行（就像一个沙盒）。

不要将本指令和其他内容处理程序指令放到同一个location中。
比如，本指令和[proxy_pass](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)指令就不能在同一个location中使用。

[返回目录](#directives)

content_by_lua_block
--------------------

**语法:** *content_by_lua_block { lua-script }*

**环境:** *location, location if*

**阶段:** *content*

与 [content_by_lua](#content_by_lua) 指令相似，只不过该指令在一对括号（`{}`）中直接内嵌 Lua 代码，替代之前 NGINX 的字符串（需要特殊字符转义）。

例如：

```nginx

 content_by_lua_block {
     ngx.say("I need no extra escaping here, for example: \r\nblah")
 }
```

该指令在 `v0.9.17` 版本首次引入。

[返回目录](#directives)

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

rewrite_by_lua
--------------

**语法:** *rewrite_by_lua &lt;lua-script-str&gt;*

**环境:** *http, server, location, location if*

**阶段:** *rewrite tail*

**注意** 自从 `v0.9.17` 版本, 使用该指令是 *不爽* 的，应使用新的 [rewrite_by_lua_block](#rewrite_by_lua_block) 指令进行替代。

作为一个重写阶段的处理程序，为每个请求执行由`<lua-script-str>`指定的 Lua 代码。

这些 Lua 代码可以调用[全部 API](#nginx-api-for-lua)，并作为一个新的协程，在一个独立的全局环境中执行（就像一个沙盒）。

注意这个处理过程总是在标准[ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)的 *后* 面。所以下面的示例可以按照预期执行：

```nginx

 location /foo {
     set $a 12; # create and initialize $a
     set $b ""; # create and initialize $b
     rewrite_by_lua 'ngx.var.b = tonumber(ngx.var.a) + 1';
     echo "res = $b";
 }
```

因为 `set $a 12` 和 `set $b ""` 的执行都在[rewrite_by_lua](#rewrite_by_lua)的前面。

另一方面，下面的示例是不能按照预期执行：

```nginx

 ?  location /foo {
 ?      set $a 12; # create and initialize $a
 ?      set $b ''; # create and initialize $b
 ?      rewrite_by_lua 'ngx.var.b = tonumber(ngx.var.a) + 1';
 ?      if ($b = '13') {
 ?         rewrite ^ /bar redirect;
 ?         break;
 ?      }
 ?
 ?      echo "res = $b";
 ?  }
```

因为 `if` 是[rewrite_by_lua](#rewrite_by_lua)之 *前* 执行的，尽管在配置中它被放到[rewrite_by_lua](#rewrite_by_lua)后面。

正确的处理方式应该是这样：

```nginx

 location /foo {
     set $a 12; # create and initialize $a
     set $b ''; # create and initialize $b
     rewrite_by_lua '
         ngx.var.b = tonumber(ngx.var.a) + 1
         if tonumber(ngx.var.b) == 13 then
             return ngx.redirect("/bar");
         end
     ';

     echo "res = $b";
 }
```

注意，[ngx_eval](http://www.grid.net.ru/nginx/eval.en.html)模块可以近似的使用[rewrite_by_lua](#rewrite_by_lua)。例如：

```nginx

 location / {
     eval $res {
         proxy_pass http://foo.com/check-spam;
     }

     if ($res = 'spam') {
         rewrite ^ /terms-of-use.html redirect;
     }

     fastcgi_pass ...;
 }
```

在ngx_lua中可以这样实施：

```nginx

 location = /check-spam {
     internal;
     proxy_pass http://foo.com/check-spam;
 }

 location / {
     rewrite_by_lua '
         local res = ngx.location.capture("/check-spam")
         if res.body == "spam" then
             return ngx.redirect("/terms-of-use.html")
         end
     ';

     fastcgi_pass ...;
 }
```

如同其他重写阶段处理， [rewrite_by_lua](#rewrite_by_lua)在子请求中也执行的。

注意，在[rewrite_by_lua](#rewrite_by_lua)处理内部，当调用`ngx.exit(ngx.OK)`时，nginx请求将继续下一阶段的内容处理。要在[rewrite_by_lua](#rewrite_by_lua)处理中终结当前请求，调用[ngx.exit](#ngxexit)，成功的请求设定 status >= 200 (`ngx.HTTP_OK`) 并 status < 300 (`ngx.HTTP_SPECIAL_RESPONSE`)，失败的请求设定`ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR)` (或其他相关的)。

如果使用了[ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)的[rewrite](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite)指令来改变 URI 并发起内部重定向，那么在当前 location 内任何有序的[rewrite_by_lua](#rewrite_by_lua) 或 [rewrite_by_lua_file](#rewrite_by_lua_file)代码都将不被执行。例如：

```nginx

 location /foo {
     rewrite ^ /bar;
     rewrite_by_lua 'ngx.exit(503)';
 }
 location /bar {
     ...
 }
```

<!-- 这里的 Lua 代码 `ngx.exit(503)` 是永远不能执行的。 waiting todo completely -->
Here the Lua code `ngx.exit(503)` will never run. This will be the case if `rewrite ^ /bar last` is used as this will similarly initiate an internal redirection. If the `break` modifier is used instead, there will be no internal redirection and the `rewrite_by_lua` code will be executed.

`rewrite_by_lua`代码将永远在 `rewrite` 请求处理阶段后面，除非[rewrite_by_lua_no_postpone](#rewrite_by_lua_no_postpone)配置开启。

[返回目录](#directives)

rewrite_by_lua_block
--------------------

**语法:** *rewrite_by_lua_block { lua-script }*

**环境:** *http, server, location, location if*

**阶段:** *rewrite tail*

与 [rewrite_by_lua](#rewrite_by_lua) 指令相似，只不过该指令在一对括号（`{}`）中直接内嵌 Lua 代码，替代之前 NGINX 的字符串（需要特殊字符转义）。

例如：

```nginx

 rewrite_by_lua_block {
     do_something("hello, world!\nhiya\n")
 }
```

该指令在 `v0.9.17` 版本首次引入。

[返回目录](#directives)

rewrite_by_lua_file
-------------------

**语法:** *rewrite_by_lua_file &lt;path-to-lua-script-file&gt;*

**环境:** *http, server, location, location if*

**阶段:** *rewrite tail*

除了通过文件`<path-to-lua-script-file>`的内容指定 Lua 代码外，该指令与[rewrite_by_lua](#rewrite_by_lua)是等价的，该指令从`v0.5.0rc32`开始支持[Lua/LuaJIT bytecode](#lualuajit-bytecode-support)的执行。

在`<path-to-lua-script-file>`中可以使用 Nginx 的内置变量用来提高灵活性，然而这带有一定的风险，通常并不推荐使用。

当给定了一个相对路径如`foo/bar.lua`，它将会被转换成绝对路径，前面增加的部分路径是 Nginx 服务启动时通过命令行选项`-p PATH`决定的`server prefix`。

当 Lua 代码缓存开启（默认），用户代码在第一次请求时完成加载（只有一次）并缓存，当 Lua 文件被修改时，每次都要对 Nginx 配置进行重新加载。Lua 代码缓存是可以暂时被禁用，通过开关[lua_code_cache](#lua_code_cache)在`nginx.conf`中设置为`off`，这样就可以避免反复重新加载 Nginx。

`rewrite_by_lua_file`代码将永远在 `rewrite` 请求处理阶段后面，除非[rewrite_by_lua_no_postpone](#rewrite_by_lua_no_postpone)配置开启。

支持通过 Nginx 变量完成动态调度文件路径，就像 [content_by_lua_file](#content_by_lua_file) 一样。

[返回目录](#directives)

access_by_lua
-------------

**语法:** *access_by_lua &lt;lua-script-str&gt;*

**环境:** *http, server, location, location if*

**阶段:** *access tail*

**注意** 自从 `v0.9.17` 版本, 使用该指令是 *不爽* 的，应使用新的 [access_by_lua_block](#access_by_lua_block) 指令进行替代。

扮演 access 阶段处理，对每次请求执行在`<lua-script-str>`中指名的 Lua 代码。

这些 Lua 代码可以调用[全部 API](#nginx-api-for-lua)，并作为一个新的协程，在一个独立的全局环境中执行（就像一个沙盒）。

注意：本指令的处理总是在标准[ngx_http_access_module](http://nginx.org/en/docs/http/ngx_http_access_module.html)的*后*面。所以下面的示例可以按照预期工作：

```nginx

 location / {
     deny    192.168.1.1;
     allow   192.168.1.0/24;
     allow   10.1.1.0/16;
     deny    all;

     access_by_lua '
         local res = ngx.location.capture("/mysql", { ... })
         ...
     ';

     # proxy_pass/fastcgi_pass/...
 }
```

换句话说，如果一个客户端 IP 地址在黑名单中，它将在 [access_by_lua](#access_by_lua)中的 Mysql 复杂认证请求之前被拒绝。

注意，[ngx_auth_request](http://mdounin.ru/hg/ngx_http_auth_request_module/)模块可以近似的被[access_by_lua](#access_by_lua)实现：

```nginx

 location / {
     auth_request /auth;

     # proxy_pass/fastcgi_pass/postgres_pass/...
 }
```

使用 ngx_lua 是这样：

```nginx

 location / {
     access_by_lua '
         local res = ngx.location.capture("/auth")

         if res.status == ngx.HTTP_OK then
             return
         end

         if res.status == ngx.HTTP_FORBIDDEN then
             ngx.exit(res.status)
         end

         ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR)
     ';

     # proxy_pass/fastcgi_pass/postgres_pass/...
 }
```

和其他access阶段处理实现，[access_by_lua](#access_by_lua)将*不*能运行在子请求中。

注意，在[access_by_lua](#access_by_lua)处理内部，当调用`ngx.exit(ngx.OK)`时，nginx请求将继续下一阶段的内容处理。要在[access_by_lua](#access_by_lua)处理中终结当前请求，调用[ngx.exit](#ngxexit)，成功的请求设定 status >= 200 (`ngx.HTTP_OK`) 并 status < 300 (`ngx.HTTP_SPECIAL_RESPONSE`)，失败的请求设定`ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR)` (或其他相关的)。

从 `v0.9.20` 版本开始，你可以使用 [access_by_lua_no_postpone](#access_by_lua_no_postpone) 指令来控制该 handler 在 Nginx 的 "access" 请求处理中的执行时机。

[返回目录](#directives)

access_by_lua_block
-------------------

**语法:** *access_by_lua_block { lua-script }*

**环境:** *http, server, location, location if*

**阶段:** *access tail*

与 [access_by_lua](#access_by_lua) 指令相似，只不过该指令在一对括号（`{}`）中直接内嵌 Lua 代码，替代之前 NGINX 的字符串（需要特殊字符转义）。

For instance,

```nginx

 access_by_lua_block {
     do_something("hello, world!\nhiya\n")
 }
```

该指令在 `v0.9.17` 版本首次引入。

access_by_lua_file
------------------

**语法:** *access_by_lua_file &lt;path-to-lua-script-file&gt;*

**环境:** *http, server, location, location if*

**阶段:** *access tail*

除了通过文件`<path-to-lua-script-file>`的内容指定 Lua 代码外，该指令与[access_by_lua](#access_by_lua)是等价的，该指令从`v0.5.0rc32`开始支持[Lua/LuaJIT bytecode](#lualuajit-bytecode-support)的执行。

在`<path-to-lua-script-file>`中可以使用 Nginx 的内置变量用来提高灵活性，然而这带有一定的风险，通常并不推荐使用。

当给定了一个相对路径如`foo/bar.lua`，它将会被转换成绝对路径，前面增加的部分路径是 Nginx 服务启动时通过命令行选项`-p PATH`决定的`server prefix`。

当 Lua 代码缓存开启（默认），用户代码在第一次请求时完成加载（只有一次）并缓存，当 Lua 文件被修改时，每次都要对 Nginx 配置进行重新加载。Lua 代码缓存是可以暂时被禁用，通过开关[lua_code_cache](#lua_code_cache)在`nginx.conf`中设置为`off`，这样就可以避免反复重新加载 Nginx。

支持通过 Nginx 变量完成动态调度文件路径，就像 [content_by_lua_file](#content_by_lua_file) 一样。

[返回目录](#directives)

header_filter_by_lua
--------------------

**语法：** *header_filter_by_lua &lt;lua-script-str&gt;*

**环境：** *http, server, location, location if*

**阶段：** *output-header-filter*

**注意** 自从 `v0.9.17` 版本, 使用该指令是 *不爽* 的，应使用新的 [header_filter_by_lua_block](#header_filter_by_lua_block) 指令进行替代。

用`<lua-script-str>`中指名的lua代码，来完成应答消息头部的过滤。

注意，下列的接口函数在这个执行阶段是被禁用的：

- 输出类函数（例：ngx.say 和 ngx.send_headers）

- 控制类函数（例：ngx.redirect 和 ngx.exec）

- 子请求相关函数（例：ngx.location.capture和ngx.location.capture_multi）

- cosocket 类函数（例：ngx.socket.tcp 和 ngx.req.socket）

这里有个使用 Lua 过滤完成覆盖应答头的例子（如果没有则添加）：

```nginx

 location / {
     proxy_pass http://mybackend;
     header_filter_by_lua 'ngx.header.Foo = "blah"';
 }

```

该指令在版本 `v0.2.1rc20` 中第一次引入。

[返回目录](#directives)

header_filter_by_lua_block
--------------------------

**语法:** *header_filter_by_lua_block { lua-script }*

**环境:** *http, server, location, location if*

**阶段:** *output-header-filter*

与 [header_filter_by_lua](#header_filter_by_lua) 指令相似，只不过该指令在一对括号（`{}`）中直接内嵌 Lua 代码，替代之前 NGINX 的字符串（需要特殊字符转义）。

例如：

```nginx

 header_filter_by_lua_block {
     ngx.header["content-length"] = nil
 }
```

该指令在 `v0.9.17` 版本首次引入。

[返回目录](#directives)

header_filter_by_lua_file
-------------------------

**语法:** *header_filter_by_lua_file &lt;path-to-lua-script-file&gt;*

**环境:** *http, server, location, location if*

**阶段:** *output-header-filter*

除了通过文件`<path-to-lua-script-file>`的内容指定 Lua 代码外，该指令与[header_filter_by_lua](#header_filter_by_lua)是等价的，该指令从`v0.5.0rc32`开始支持[Lua/LuaJIT bytecode](#lualuajit-bytecode-support)的执行。

当给定了一个相对路径如`foo/bar.lua`，它将会被转换成绝对路径，前面增加的部分路径是 Nginx 服务启动时通过命令行选项`-p PATH`决定的`server prefix`。

该指令是在`v0.2.1rc20`版本第一次引入。

[返回目录](#directives)

body_filter_by_lua
------------------

**语法:** *body_filter_by_lua &lt;lua-script-str&gt;*

**环境:** *http, server, location, location if*

**阶段:** *output-body-filter*

**注意** 自从 `v0.9.17` 版本, 使用该指令是 *不爽* 的，应使用新的 [body_filter_by_lua_block](#body_filter_by_lua_block) 指令进行替代。

使用`<lua-script-str>`指定的 Lua 代码定义一个输出应答体过滤器。

输入数据块是 [ngx.arg](#ngxarg)\[1\]（Lua的字符串形式）,结束标识"eof"是应答体数据最后一位[ngx.arg](#ngxarg)\[2\]（Lua的布尔值形式）。

在这个场景下，结束标识"eof"仅仅是 Nginx chain 缓冲区的`last_buf`（主请求）或`last_in_chain`（子请求）。（在`v0.7.14`版本之前，结束标识"eof"在子请求中是完全不能使用的。）

使用下面 Lua 代码，可以对输出数据流立即终止：

```lua

 return ngx.ERROR
```

这样截断响应体，通常导致结果不完整的，也是无效的响应。

本指令的 Lua 代码可以使用 Lua 字符串或字符串的表重写[ngx.arg](#ngxarg)\[1\]输入数据块内容，从而完成 Nginx 输出体下游过滤数据修改。例如，在输出体转换所有的小写字母，我们可以这样用：

```nginx

 location / {
     proxy_pass http://mybackend;
     body_filter_by_lua 'ngx.arg[1] = string.upper(ngx.arg[1])';
 }
```

当设置`nil`或一个空的 Lua 字符串值给`ngx.arg[1]`，将没有任何数据块下发到 Nginx 下游。

同样，新的结束标识"eof"也可以通过对[ngx.arg](#ngxarg)\[2\]设定一个布尔值。例如：

```nginx

 location /t {
     echo hello world;
     echo hiya globe;

     body_filter_by_lua '
         local chunk = ngx.arg[1]
         if string.match(chunk, "hello") then
             ngx.arg[2] = true  -- new eof
             return
         end

         -- just throw away any remaining chunk data
         ngx.arg[1] = nil
     ';
 }
```

然后 `GET /t` 将返回下面的结果：

    hello world

就是说，当应答体过滤发现一个块包含关键字"hello"，它将立即设置结束标识"eof"为 true ，应答内容被截断尽管后面还有有效数据。

当Lua代码可能改变应答体的长度时，我们必须总是清空响应头中的`Content-Length`（如果有），强制使用流式输出，如：

```nginx

 location /foo {
     # fastcgi_pass/proxy_pass/...

     header_filter_by_lua 'ngx.header.content_length = nil';
     body_filter_by_lua 'ngx.arg[1] = string.len(ngx.arg[1]) .. "\\n"';
 }
```

注意：下面这些API函数在这个环境中是禁用的，这受制于当前 Nginx 输出过滤器的实现：

* 输出API函数类（例如：[ngx.say](#ngxsay) 和 [ngx.send_headers](#ngxsend_headers)）
* 控制API函数类（例如：[ngx.redirect](#ngxredirect) 和 [ngx.exec](#ngxexec)）
* 子请求函数类（例如：[ngx.location.capture](#ngxlocationcapture) 和 [ngx.location.capture_multi](#ngxlocationcapture_multi)）
* cosocket 函数类（例如：[ngx.socket.tcp](#ngxsockettcp) 和 [ngx.req.socket](#ngxreqsocket)）

Nginx 输出过滤器在一个单独请求中可能被调用多次，因为应答体可能使用块的方式进行投递。所以，本指令中的 Lua 代码在这个单独的HTTP请求生命周期内，同样会执行多次。

该指令在`v0.5.0rc32`版本中首次引入。

[返回目录](#directives)

body_filter_by_lua_block
------------------------

**语法:** *body_filter_by_lua_block { lua-script-str }*

**环境:** *http, server, location, location if*

**阶段:** *output-body-filter*

与 [body_filter_by_lua](#body_filter_by_lua) 指令相似，只不过该指令在一对括号（`{}`）中直接内嵌 Lua 代码，替代之前 NGINX 的字符串（需要特殊字符转义）。

例如：

```nginx

 body_filter_by_lua_block {
     local data, eof = ngx.arg[1], ngx.arg[2]
 }
```

该指令在 `v0.9.17` 版本首次引入。

[返回目录](#directives)

body_filter_by_lua_file
-----------------------

**语法:** *body_filter_by_lua_file &lt;path-to-lua-script-file&gt;*

**环境:** *http, server, location, location if*

**阶段:** *output-body-filter*

除了通过文件`<path-to-lua-script-file>`的内容指定 Lua 代码外，该指令与[body_filter_by_lua](#body_filter_by_lua)是等价的，该指令从`v0.5.0rc32`开始支持[Lua/LuaJIT bytecode](#lualuajit-bytecode-support)的执行。

当给定了一个相对路径如`foo/bar.lua`，它将会被转换成绝对路径，前面增加的部分路径是 Nginx 服务启动时通过命令行选项`-p PATH`决定的`server prefix`。

该指令是在`v0.5.0rc32`版本第一次引入。

[返回目录](#directives)

log_by_lua
----------

**语法:** *log_by_lua &lt;lua-script-str&gt;*

**环境:** *http, server, location, location if*

**阶段:** *log*

**注意** 自从 `v0.9.17` 版本, 使用该指令是 *不爽* 的，应使用新的 [log_by_lua_block](#log_by_lua_block) 指令进行替代。

在 `log` 请求处理阶段执行内嵌在`<lua-script-str>`的 Lua 代码。它不替代当前access的日志，而是在其后执行。

注意，当前环境中以下 API 函数当前是被禁用的：

* 输出API函数类（例如：[ngx.say](#ngxsay) 和 [ngx.send_headers](#ngxsend_headers)）
* 控制API函数类（例如：[ngx.exit](#ngxexit) 和 [ngx.exec](#ngxexec)）
* 子请求函数类（例如：[ngx.location.capture](#ngxlocationcapture) 和 [ngx.location.capture_multi](#ngxlocationcapture_multi)）
* cosocket 函数类（例如：[ngx.socket.tcp](#ngxsockettcp) 和 [ngx.req.socket](#ngxreqsocket)）

这是一个收集 [$upstream_response_time](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_response_time) 平均处理的例子：

```nginx

 lua_shared_dict log_dict 5M;

 server {
     location / {
         proxy_pass http://mybackend;

         log_by_lua '
             local log_dict = ngx.shared.log_dict
             local upstream_time = tonumber(ngx.var.upstream_response_time)

             local sum = log_dict:get("upstream_time-sum") or 0
             sum = sum + upstream_time
             log_dict:set("upstream_time-sum", sum)

             local newval, err = log_dict:incr("upstream_time-nb", 1)
             if not newval and err == "not found" then
                 log_dict:add("upstream_time-nb", 0)
                 log_dict:incr("upstream_time-nb", 1)
             end
         ';
     }

     location = /status {
         content_by_lua '
             local log_dict = ngx.shared.log_dict
             local sum = log_dict:get("upstream_time-sum")
             local nb = log_dict:get("upstream_time-nb")

             if nb and sum then
                 ngx.say("average upstream response time: ", sum / nb,
                         " (", nb, " reqs)")
             else
                 ngx.say("no data yet")
             end
         ';
     }
 }
```

该指令在`v0.5.0rc31`版本被首次引入。

[返回目录](#directives)

log_by_lua_block
----------------

**语法:** *log_by_lua_block { lua-script }*

**内容:** *http, server, location, location if*

**阶段:** *log*

与 [log_by_lua](#log_by_lua) 指令相似，只不过该指令在一对括号（`{}`）中直接内嵌 Lua 代码，替代之前 NGINX 的字符串（需要特殊字符转义）。

例如：

```nginx

 log_by_lua_block {
     print("I need no extra escaping here, for example: \r\nblah")
 }
```

该指令在 `v0.9.17` 版本首次引入。

[返回目录](#directives)

log_by_lua_file
---------------

**语法:** *log_by_lua_file &lt;path-to-lua-script-file&gt;*

**环境:** *http, server, location, location if*

**阶段:** *log*

除了通过文件`<path-to-lua-script-file>`的内容指定 Lua 代码外，该指令与[log_by_lua](#log_by_lua)是等价的，该指令从`v0.5.0rc32`开始支持[Lua/LuaJIT bytecode](#lualuajit-bytecode-support)的执行。

当给定了一个相对路径如`foo/bar.lua`，它将会被转换成绝对路径，前面增加的部分路径是 Nginx 服务启动时通过命令行选项`-p PATH`决定的`server prefix`。

该指令是在`v0.5.0rc31`版本第一次引入。

[Back to TOC](#directives)

balancer_by_lua_block
---------------------

**语法:** *balancer_by_lua_block { lua-script }*

**环境:** *upstream*

**阶段:** *content*

该指令执行上游的负载均衡 Lua 代码（任何上游实体），代码配置在 `upstream {}` 小节中。

举例：

```nginx

 upstream foo {
     server 127.0.0.1;
     balancer_by_lua_block {
         -- 使用 Lua 作为一个动态均衡器完成一些有趣的事情
     }
 }

 server {
     location / {
         proxy_pass http://foo;
     }
 }
```

Lua 的负载均衡可以和任何已经存在的 nginx 上游模块一起工作，例如：[ngx_proxy](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) 和
[ngx_fastcgi](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html)。

同样， Lua 负载均衡可以和标准的上游连接池机制一起工作，例如标准的 [keepalive](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive) 指令。只要确保在单个`upstream {}` 配置小节中 [keepalive](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive) 指令要放在 `balancer_by_lua_block` 小节的 *后* 面。

Lua 负载均衡能完全忽略配置在 `upstream {}` 小节中定义的服务列表，并且从一个完全动态的服务列表中挑选一个节点（甚至每次请求都在变），所有这些均是通过 [lua-resty-core](https://github.com/openresty/lua-resty-core) 库的 [ngx.balancer](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/balancer.md) 完成。

该指令配置的 Lua 代码在单个下游请求中可能被调用多次，例如使用 [proxy_next_upstream](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_next_upstream) 配置小节，这是 nginx 自身上游尝试请求机制。

这里 Lua 代码的执行环境不支持 yield 操作，所以可能 yield 的 Lua API （例如 cosockets 和 “轻线程”），在这个环境中是被禁用的。一个可以使用并绕过这个限制的玩法，是可以在更早的阶段处理，比如 [access_by_lua*](#access_by_lua))，并传递结果到环境的 [ngx.ctx](#ngxctx) 表。

该指令在 `v0.10.0` 版本首次引入。

[返回目录](#directives)

balancer_by_lua_file
--------------------

**语法:** *balancer_by_lua_file &lt;path-to-lua-script-file&gt;*

**环境:** *upstream*

**阶段:** *content*

除了通过文件 `<path-to-lua-script-file>` 的内容指定 Lua 代码外，该指令与 [balancer_by_lua_block](#balancer_by_lua_block) 是等价的，该指令从 `v0.5.0rc32` 开始支持 [Lua/LuaJIT bytecode](#lualuajit-bytecode-support) 的执行。

当给定了一个相对路径如 `foo/bar.lua`，它将会被转换成绝对路径，前面增加的部分路径是 Nginx 服务启动时通过命令行选项 `-p PATH` 决定的 `server prefix`。

该指令在 `v0.10.0` 版本首次引入。

[返回目录](#directives)

lua_need_request_body
---------------------

**语法:** *lua_need_request_body &lt;on|off&gt;*

**默认:** *off*

**环境:** *http, server, location, location if*

**阶段:** *depends on usage*

在运行 rewrite/access/access_by_lua* 之前决定是否强制获取请求体数据。 Nginx 内部默认不读取客户端请求体，如果需要读取请求体数据，需要使用该指令设置为 `on` 或者在 Lua 代码中调用 [ngx.req.read_body](#ngxreqread_body) 函数。

为了读取请求体数据到[$request_body](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_request_body)变量，[client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size)必须要与[client_max_body_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size)有同样的大小。因为内容大小超过[client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size)但是小于[client_max_body_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size)时， Nginx 将把缓冲内存数据存到一个磁盘的临时文件上，这将导致[$request_body](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_request_body)变量是一个空值。

如果当前location包含 [rewrite_by_lua](#rewrite_by_lua) 或 [rewrite_by_lua_file](#rewrite_by_lua_file) 指令，请求体将在[rewrite_by_lua](#rewrite_by_lua) 或 [rewrite_by_lua_file](#rewrite_by_lua_file)代码运行之前（还是在`rewrite`阶段）被读取。如果只有[content_by_lua](#content_by_lua)指令，请求体直到内容生成的 Lua 代码执行时才会读取（既，请求体在处理生成返回数据阶段才回被读取）。

无论如何都非常推荐，使用[ngx.req.read_body](#ngxreqread_body)和[ngx.req.discard_body](#ngxreqdiscard_body)函数，可以更好的控制请求体的读取过程。

这些也适用于 [access_by_lua](#access_by_lua) 和 [access_by_lua_file](#access_by_lua_file)。

[返回目录](#directives)

ssl_certificate_by_lua_block
----------------------------

**syntax:** *ssl_certificate_by_lua_block { lua-script }*

**context:** *server*

**phase:** *right-before-SSL-handshake*

This directive runs user Lua code when NGINX is about to start the SSL handshake for the downstream
SSL (https) connections.

It is particularly useful for setting the SSL certificate chain and the corresponding private key on a per-request
basis. It is also useful to load such handshake configurations nonblockingly from the remote (for example,
with the [cosocket](#ngxsockettcp) API). And one can also do per-request OCSP stapling handling in pure
Lua here as well.

Another typical use case is to do SSL handshake traffic control nonblockingly in this context,
with the help of the [lua-resty-limit-traffic#readme](https://github.com/openresty/lua-resty-limit-traffic)
library, for example.

One can also do interesting things with the SSL handshake requests from the client side, like
rejecting old SSL clients using the SSLv3 protocol or even below selectively.

The [ngx.ssl](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/ssl.md)
and [ngx.ocsp](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/ocsp.md) Lua modules
provided by the [lua-resty-core](https://github.com/openresty/lua-resty-core/#readme)
library are particularly useful in this context. You can use the Lua API offered by these two Lua modules
to manipulate the SSL certificate chain and private key for the current SSL connection
being initiated.

This Lua handler does not run at all, however, when NGINX/OpenSSL successfully resumes
the SSL session via SSL session IDs or TLS session tickets for the current SSL connection. In
other words, this Lua handler only runs when NGINX has to initiate a full SSL handshake.

Below is a trivial example using the
[ngx.ssl](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/ssl.md) module
at the same time:

```nginx

 server {
     listen 443 ssl;
     server_name   test.com;

     ssl_certificate_by_lua_block {
         print("About to initiate a new SSL handshake!")
     }

     location / {
         root html;
     }
 }
```

See more complicated examples in the [ngx.ssl](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/ssl.md)
and [ngx.ocsp](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/ocsp.md)
Lua modules' official documentation.

Uncaught Lua exceptions in the user Lua code immediately abort the current SSL session, so does the
[ngx.exit](#ngxexit) call with an error code like `ngx.ERROR`.

This Lua code execution context *does* support yielding, so Lua APIs that may yield
(like cosockets, sleeping, and "light threads")
are enabled in this context.

Note, however, you still need to configure the [ssl_certificate](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate) and
[ssl_certificate_key](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate_key)
directives even though you will not use this static certificate and private key at all. This is
because the NGINX core requires their appearance otherwise you are seeing the following error
while starting NGINX:


    nginx: [emerg] no ssl configured for the server


This directive currently requires the following NGINX core patch to work correctly:

<http://mailman.nginx.org/pipermail/nginx-devel/2016-January/007748.html>

The bundled version of the NGINX core in OpenResty 1.9.7.2 (or above) already has this
patch applied.

Furthermore, one needs at least OpenSSL 1.0.2e for this directive to work.

This directive was first introduced in the `v0.10.0` release.

[Back to TOC](#directives)

ssl_certificate_by_lua_file
---------------------------

**syntax:** *ssl_certificate_by_lua_file &lt;path-to-lua-script-file&gt;*

**context:** *server*

**phase:** *right-before-SSL-handshake*

Equivalent to [ssl_certificate_by_lua_block](#ssl_certificate_by_lua_block), except that the file specified by `<path-to-lua-script-file>` contains the Lua code, or, as from the `v0.5.0rc32` release, the [Lua/LuaJIT bytecode](#lualuajit-bytecode-support) to be executed.

When a relative path like `foo/bar.lua` is given, they will be turned into the absolute path relative to the `server prefix` path determined by the `-p PATH` command-line option while starting the Nginx server.

This directive was first introduced in the `v0.10.0` release.

[Back to TOC](#directives)

lua_shared_dict
---------------

**语法:** *lua_shared_dict &lt;name&gt; &lt;size&gt;*

**默认:** *no*

**环境:** *http*

**阶段:** *depends on usage*

声明一个共享内存区块 `<name>`，用来存储基于共享内存的 Lua 字典 `ngx.shared.<name>`。

在当前 Nginx 服务器实例中，共享内存区块被所有 nginx worker 进程共享。

`<size>` 参数可以通过类似 `k` 和 `m` 的大小单位来设置。

```nginx

 http {
     lua_shared_dict dogs 10m;
     ...
 }
```

硬编码限制最小大小是 8KB，而实际的最小大小取决于实际中用户数据集（有些人是从 12KB 开始）。

更多细节请参考 [ngx.shared.DICT](#ngxshareddict)。

这个指令最早出现在版本 `v0.3.1rc22` 中。

[Back to TOC](#directives)

lua_socket_connect_timeout
--------------------------

**语法:** *lua_socket_connect_timeout &lt;time&gt;*

**默认:** *lua_socket_connect_timeout 60s*

**环境:** *http, server, location*

该指令控制 TCP/unix-domain socket 对象的[connect](#tcpsockconnect)方法默认超时时间，这个值可以被[settimeout](#tcpsocksettimeout)方法覆盖。

`<time>`参数可以是整数，后面可以跟着像`s` (秒), `ms` (毫秒), `m` (分钟)的单位可选项。 默认的时间单位是`s`，也就是"秒"。默认值是`60s`。

这个指令最早出现在版本 `v0.5.0rc1` 中。

[返回目录](#directives)

lua_socket_send_timeout
-----------------------

**语法:** *lua_socket_send_timeout &lt;time&gt;*

**默认:** *lua_socket_send_timeout 60s*

**环境:** *http, server, location*

该指令控制 TCP/unix-domain socket 对象的[send](#tcpsocksend)方法默认超时时间，这个值可以被[settimeout](#tcpsocksettimeout)方法覆盖。

`<time>`参数可以是整数，后面可以跟着像`s` (秒), `ms` (毫秒), `m` (分钟)的单位可选项。 默认的时间单位是`s`，也就是"秒"。默认值是`60s`。

该指令是在`v0.5.0rc1`版本第一次引入。

[返回目录](#directives)

lua_socket_send_lowat
---------------------

**语法:** *lua_socket_send_lowat &lt;size&gt;*

**默认:** *lua_socket_send_lowat 0*

**环境:** *http, server, location*

控制 cosocket 发送缓冲区 `lowat`（低水位）的值。

[返回目录](#directives)

lua_socket_read_timeout
-----------------------

**语法:** *lua_socket_read_timeout &lt;time&gt;*

**默认:** *lua_socket_read_timeout 60s*

**环境:** *http, server, location*

**阶段:** *依赖于使用环境*

该指令控制 TCP/unix-domain socket 对象的[receive](#tcpsockreceive)方法、[receiveuntil](#tcpsockreceiveuntil)方法返回迭代函数的默认超时时间。这个值可以被[settimeout](#tcpsocksettimeout)方法覆盖。

`<time>`参数可以是整数，后面可以跟着像`s` (秒), `ms` (毫秒), `m` (分钟)的单位可选项。 默认的时间单位是`s`，也就是"秒"。默认值是`60s`。

该指令是在`v0.5.0rc1`版本第一次引入。

[返回目录](#directives)

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

lua_socket_pool_size
--------------------

**语法:** *lua_socket_pool_size &lt;size&gt;*

**默认:** *lua_socket_pool_size 30*

**环境:** *http, server, location*

指定每个 cosocket 通过远程服务(例如，使用主机+端口配对或 unix socket 文件路径作为标识)关联的连接池的大小限制（每个地址中的连接数）。

每个连接池默认是30个连接。

当连接池中连接数超过限制大小，在连接池中最近最少使用的（空闲）连接将被关闭，给当前连接腾挪空间。

注意，cosocket 连接池是每个 Nginx 工作进程使用的，而不是每个 Nginx 服务实例，所以这里指定的限制也只能在每个独立的 nginx 工作进程上生效。

该指令在`v0.5.0rc1`版本首次引入。

[返回目录](#directives)

lua_socket_keepalive_timeout
----------------------------

**语法:** *lua_socket_keepalive_timeout &lt;time&gt;*

**默认:** *lua_socket_keepalive_timeout 60s*

**环境:** *http, server, location*

该指令控制在 cosocket 连接池中连接的默认最大空闲时间。当这个时间到达，空闲的连接将被关闭并从连接池中移除。这个值可以使用 cosocket 对象的[setkeepalive](#tcpsocksetkeepalive)方法覆盖。

`<time>`参数可以是整数，后面可以跟着像`s` (秒), `ms` (毫秒), `m` (分钟)的单位可选项。 默认的时间单位是`s`，也就是"秒"。默认值是`60s`。

这个指令最早出现在版本 `v0.5.0rc1` 。

[返回目录](#directives)

lua_socket_log_errors
---------------------

**语法:** *lua_socket_log_errors on|off*

**默认:** *lua_socket_log_errors on*

**环境:** *http, server, location*

当TCP 或 UDP cosockets出现失败时，该指令可被用来切换错误日志输出。如果你已经正确处理了你的 Lua 代码错误日志，这里就推荐设置当前指令的开关为 off ，防止数据刷写到你的 nginx 错误日志文件（通常这个代价是比较昂贵的）。

这个指令最早出现在版本 `v0.5.13` 中。

[Back to TOC](#directives)

lua_ssl_ciphers
---------------

**语法:** *lua_ssl_ciphers &lt;ciphers&gt;*

**默认:** *lua_ssl_ciphers DEFAULT*

**环境:** *http, server, location*

指定在[tcpsock:sslhandshake](#tcpsocksslhandshake) 方法中请求 SSL/TLS 服务 的加密方式。 其中参数 ciphers 是 OpenSSL 库里面指定的格式。

可以使用 “openssl ciphers” 来查看完整的加密方式列表。

该指令是在`v0.9.11`版本首次引入的。

[返回目录](#directives)

lua_ssl_crl
-----------

**语法:** *lua_ssl_crl &lt;file&gt;*

**默认:** *no*

**环境:** *http, server, location*

指定一个 PEM 格式吊销证书文件，在[tcpsock:sslhandshake](#tcpsocksslhandshake)方法里验证 SSL/TLS 服务的证书。

该指令是在`v0.9.11`版本首次引入的。

[返回目录](#directives)

lua_ssl_protocols
-----------------

**语法:** *lua_ssl_protocols \[SSLv2\] \[SSLv3\] \[TLSv1\] [TLSv1.1] [TLSv1.2]*

**默认:** *lua_ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2*

**环境:** *http, server, location*

在 [tcpsock:sslhandshake](#tcpsocksslhandshake) 方法中开启请求 SSL/TLS 服务的传输协议列表。

该指令是在`v0.9.11`版本首次引入的。

[返回目录](#directives)

lua_ssl_trusted_certificate
---------------------------

**语法:** *lua_ssl_trusted_certificate &lt;file&gt;*

**默认:** *no*

**环境:** *http, server, location*

指定一个 PEM 格式信任 CA 证书文件，在[tcpsock:sslhandshake](#tcpsocksslhandshake)方法里验证 SSL/TLS 服务的证书。

该指令是在`v0.9.11`版本首次引入的。

也可以看看 [lua_ssl_verify_depth](#lua_ssl_verify_depth).

[返回目录](#directives)

lua_ssl_verify_depth
--------------------

**语法:** *lua_ssl_verify_depth &lt;number&gt;*

**默认:** *lua_ssl_verify_depth 1*

**环境:** *http, server, location*

设置服务端证书链的验证深度。

该指令是在`v0.9.11`版本首次引入的。

也可以看看 [lua_ssl_trusted_certificate](#lua_ssl_trusted_certificate)。

[返回目录](#directives)

lua_http10_buffering
--------------------

**语法:** *lua_http10_buffering on|off*

**默认:** *lua_http10_buffering on*

**环境:** *http, server, location, location-if*

对 HTTP 1.0（或更老）请求，启用或禁用自动应答缓冲区。这个缓冲机制主要用于应答头包含合适`Content-Length`长度的 HTTP 1.0 长连接。

如果 Lua 代码在发送应答头之前明确设置了应答头的`Content-Length`（调用 [ngx.send_headers](#ngxsend_headers) 或 隐式首次调用 [ngx.say](#ngxsay) 或 [ngx.print](#ngxprint) 其中任何一个），HTTP 1.0应答缓冲区都将被禁用，即使这个指令是打开的。

流式输出（例如，调用 [ngx.flush](#ngxflush)）非常大的应答体，为了占用内存最小，该指令必须设置为`off`。

该指令默认值是`on`。

该指令是在`v0.5.0rc19`版本首次引入的。

[返回目录](#directives)

rewrite_by_lua_no_postpone
--------------------------

**语法:** *rewrite_by_lua_no_postpone on|off*

**默认:** *rewrite_by_lua_no_postpone off*

**环境:** *http*

控制是否禁用 [rewrite_by_lua](#rewrite_by_lua)* 指令在`rewrite`阶段的延迟执行。该指令的默认值是 `off` ，在`rewrite`阶段的 Lua 代码将被延迟到最后执行。

该指令是在`v0.5.0rc29`版本首次引入的。

[返回目录](#directives)

access_by_lua_no_postpone
-------------------------

**语法:** *access_by_lua_no_postpone on|off*

**默认:** *access_by_lua_no_postpone off*

**环境:** *http*

控制是否禁用 [access_by_lua](#access_by_lua)* 指令在 `access` 请求处理阶段末尾的推迟执行。默认的，该指令是 `off` 并且 `access` 阶段的 Lua 代码是被延迟到末尾执行。

该指令在 `v0.9.20` 版本首次引入。

[返回目录](#directives)

lua_transform_underscores_in_response_headers
---------------------------------------------

**语法:** *lua_transform_underscores_in_response_headers on|off*

**默认:** *lua_transform_underscores_in_response_headers on*

**环境:** *http, server, location, location-if*

对于[ngx.header.HEADER](#ngxheaderheader) API 中指定响应头，该指令指定是否将下划线(`_`)转化为连接线(`-`)。

该指令是在`v0.5.0rc32`版本首次引入的。

[返回目录](#directives)

lua_check_client_abort
----------------------

**语法:** *lua_check_client_abort on|off*

**默认:** *lua_check_client_abort off*

**环境:** *http, server, location, location-if*

该指令控制是否探测客户端连接的过早终止。

当启用该指令，ngx_lua模块将会在下游连接上监控连接过早关闭事件。当有这样的事件时，它将调用用户指定 Lua 的回调函数（通过 [ngx.on_abort](#ngxon_abort) 注册），当这里没有用户回调函数注册时，将停止当前会话并清理所有当前请求中运行的 Lua "轻线程" 。

根据目前实现，无论如何，如果请求正在通过 [ngx.req.socket](#ngxreqsocket) 读取请求体，在它之前客户连接发生关闭，ngx_lua 将不会停止任何正在执行的“轻线程”也不会调用用户的回调（尽管已经调用 [ngx.on_abort](#ngxon_abort) ）。作为替代，使用 [ngx.req.socket](#ngxreqsocket) 的读操作第二个参数将直接返回错误信息 “client aborted” 作为返回值（第一个返回值确定是`nil`）。

当 TCP 长连接被禁用，它依靠客户端 socket 关闭的优雅实现（通过发送一个`FIN`包或类似的东西）。

对与（软）实时 Web 应用，强烈推荐使用系统 TCP 协议栈支持的选项对 [TCP keepalive](http://tldp.org/HOWTO/TCP-Keepalive-HOWTO/overview.html) 进行配置，以便及时发现“半开” TCP 连接。

例如，在 Linux，在你的`nginx.conf`文件中你能使用标准 [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) 指令配置，像这样：

```nginx

 listen 80 so_keepalive=2s:2s:8;
```

对于 FreeBSD，你可以只调整 TCP 长连接的系统范围的配置，例如：

    # sysctl net.inet.tcp.keepintvl=2000
    # sysctl net.inet.tcp.keepidle=2000

该指令是在`v0.7.4`版本首次引入的。

也可以看看 [ngx.on_abort](#ngxon_abort) 。

[返回目录](#directives)

lua_max_pending_timers
----------------------

**语法:** *lua_max_pending_timers &lt;count&gt;*

**默认:** *lua_max_pending_timers 1024*

**环境:** *http*

控制允许使用的`pending timers`最大数量。

`pending timers` 指的是还没有过期的 `timers` 。

当超过这个限制， [ngx.timer.at](#ngxtimerat) 调用将立即返回 `nil` 和 错误信息 “too many pending timers”。

该指令是在`v0.8.0`版本首次引入的。

[返回目录](#directives)

lua_max_running_timers
----------------------

**语法:** *lua_max_running_timers &lt;count&gt;*

**默认:** *lua_max_running_timers 256*

**环境:** *http*

控制允许的"running timers"最大数量。

"running timers" 指的是那些正在执行用户回调函数的 timers 。

当超过这个限制，Nginx 将停止执行新近过期的 timers 回调，并记录一个错误日志 “N lua_max_running_timers are not enough”，这里的 "N" 是这个指令的当前值。

该指令是在`v0.8.0`版本首次引入的。

[返回目录](#directives)

Nginx API for Lua
=================

* [Introduction](#introduction)
* [ngx.arg](#ngxarg)
* [ngx.var.VARIABLE](#ngxvarvariable)
* [Core constants](#core-constants)
* [HTTP method constants](#http-method-constants)
* [HTTP status constants](#http-status-constants)
* [Nginx log level constants](#nginx-log-level-constants)
* [print](#print)
* [ngx.ctx](#ngxctx)
* [ngx.location.capture](#ngxlocationcapture)
* [ngx.location.capture_multi](#ngxlocationcapture_multi)
* [ngx.status](#ngxstatus)
* [ngx.header.HEADER](#ngxheaderheader)
* [ngx.resp.get_headers](#ngxrespget_headers)
* [ngx.req.is_internal](#ngxreqis_internal)
* [ngx.req.start_time](#ngxreqstart_time)
* [ngx.req.http_version](#ngxreqhttp_version)
* [ngx.req.raw_header](#ngxreqraw_header)
* [ngx.req.get_method](#ngxreqget_method)
* [ngx.req.set_method](#ngxreqset_method)
* [ngx.req.set_uri](#ngxreqset_uri)
* [ngx.req.set_uri_args](#ngxreqset_uri_args)
* [ngx.req.get_uri_args](#ngxreqget_uri_args)
* [ngx.req.get_post_args](#ngxreqget_post_args)
* [ngx.req.get_headers](#ngxreqget_headers)
* [ngx.req.set_header](#ngxreqset_header)
* [ngx.req.clear_header](#ngxreqclear_header)
* [ngx.req.read_body](#ngxreqread_body)
* [ngx.req.discard_body](#ngxreqdiscard_body)
* [ngx.req.get_body_data](#ngxreqget_body_data)
* [ngx.req.get_body_file](#ngxreqget_body_file)
* [ngx.req.set_body_data](#ngxreqset_body_data)
* [ngx.req.set_body_file](#ngxreqset_body_file)
* [ngx.req.init_body](#ngxreqinit_body)
* [ngx.req.append_body](#ngxreqappend_body)
* [ngx.req.finish_body](#ngxreqfinish_body)
* [ngx.req.socket](#ngxreqsocket)
* [ngx.exec](#ngxexec)
* [ngx.redirect](#ngxredirect)
* [ngx.send_headers](#ngxsend_headers)
* [ngx.headers_sent](#ngxheaders_sent)
* [ngx.print](#ngxprint)
* [ngx.say](#ngxsay)
* [ngx.log](#ngxlog)
* [ngx.flush](#ngxflush)
* [ngx.exit](#ngxexit)
* [ngx.eof](#ngxeof)
* [ngx.sleep](#ngxsleep)
* [ngx.escape_uri](#ngxescape_uri)
* [ngx.unescape_uri](#ngxunescape_uri)
* [ngx.encode_args](#ngxencode_args)
* [ngx.decode_args](#ngxdecode_args)
* [ngx.encode_base64](#ngxencode_base64)
* [ngx.decode_base64](#ngxdecode_base64)
* [ngx.crc32_short](#ngxcrc32_short)
* [ngx.crc32_long](#ngxcrc32_long)
* [ngx.hmac_sha1](#ngxhmac_sha1)
* [ngx.md5](#ngxmd5)
* [ngx.md5_bin](#ngxmd5_bin)
* [ngx.sha1_bin](#ngxsha1_bin)
* [ngx.quote_sql_str](#ngxquote_sql_str)
* [ngx.today](#ngxtoday)
* [ngx.time](#ngxtime)
* [ngx.now](#ngxnow)
* [ngx.update_time](#ngxupdate_time)
* [ngx.localtime](#ngxlocaltime)
* [ngx.utctime](#ngxutctime)
* [ngx.cookie_time](#ngxcookie_time)
* [ngx.http_time](#ngxhttp_time)
* [ngx.parse_http_time](#ngxparse_http_time)
* [ngx.is_subrequest](#ngxis_subrequest)
* [ngx.re.match](#ngxrematch)
* [ngx.re.find](#ngxrefind)
* [ngx.re.gmatch](#ngxregmatch)
* [ngx.re.sub](#ngxresub)
* [ngx.re.gsub](#ngxregsub)
* [ngx.shared.DICT](#ngxshareddict)
* [ngx.shared.DICT.get](#ngxshareddictget)
* [ngx.shared.DICT.get_stale](#ngxshareddictget_stale)
* [ngx.shared.DICT.set](#ngxshareddictset)
* [ngx.shared.DICT.safe_set](#ngxshareddictsafe_set)
* [ngx.shared.DICT.add](#ngxshareddictadd)
* [ngx.shared.DICT.safe_add](#ngxshareddictsafe_add)
* [ngx.shared.DICT.replace](#ngxshareddictreplace)
* [ngx.shared.DICT.delete](#ngxshareddictdelete)
* [ngx.shared.DICT.incr](#ngxshareddictincr)
* [ngx.shared.DICT.flush_all](#ngxshareddictflush_all)
* [ngx.shared.DICT.flush_expired](#ngxshareddictflush_expired)
* [ngx.shared.DICT.get_keys](#ngxshareddictget_keys)
* [ngx.socket.udp](#ngxsocketudp)
* [udpsock:setpeername](#udpsocksetpeername)
* [udpsock:send](#udpsocksend)
* [udpsock:receive](#udpsockreceive)
* [udpsock:close](#udpsockclose)
* [udpsock:settimeout](#udpsocksettimeout)
* [ngx.socket.tcp](#ngxsockettcp)
* [tcpsock:connect](#tcpsockconnect)
* [tcpsock:sslhandshake](#tcpsocksslhandshake)
* [tcpsock:send](#tcpsocksend)
* [tcpsock:receive](#tcpsockreceive)
* [tcpsock:receiveuntil](#tcpsockreceiveuntil)
* [tcpsock:close](#tcpsockclose)
* [tcpsock:settimeout](#tcpsocksettimeout)
* [tcpsock:setoption](#tcpsocksetoption)
* [tcpsock:setkeepalive](#tcpsocksetkeepalive)
* [tcpsock:getreusedtimes](#tcpsockgetreusedtimes)
* [ngx.socket.connect](#ngxsocketconnect)
* [ngx.get_phase](#ngxget_phase)
* [ngx.thread.spawn](#ngxthreadspawn)
* [ngx.thread.wait](#ngxthreadwait)
* [ngx.thread.kill](#ngxthreadkill)
* [ngx.on_abort](#ngxon_abort)
* [ngx.timer.at](#ngxtimerat)
* [ngx.timer.running_count](#ngxtimerrunning_count)
* [ngx.timer.pending_count](#ngxtimerpending_count)
* [ngx.config.debug](#ngxconfigdebug)
* [ngx.config.prefix](#ngxconfigprefix)
* [ngx.config.nginx_version](#ngxconfignginx_version)
* [ngx.config.nginx_configure](#ngxconfignginx_configure)
* [ngx.config.ngx_lua_version](#ngxconfigngx_lua_version)
* [ngx.worker.exiting](#ngxworkerexiting)
* [ngx.worker.pid](#ngxworkerpid)
* [ngx.worker.count](#ngxworkercount)
* [ngx.worker.id](#ngxworkerid)
* [ngx.semaphore](#ngxsemaphore)
* [ngx.balancer](#ngxbalancer)
* [ngx.ssl](#ngxssl)
* [ngx.ocsp](#ngxocsp)
* [ndk.set_var.DIRECTIVE](#ndkset_vardirective)
* [coroutine.create](#coroutinecreate)
* [coroutine.resume](#coroutineresume)
* [coroutine.yield](#coroutineyield)
* [coroutine.wrap](#coroutinewrap)
* [coroutine.running](#coroutinerunning)
* [coroutine.status](#coroutinestatus)


[Back to TOC](#table-of-contents)

Introduction
------------
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

[Back to TOC](#nginx-api-for-lua)

ngx.arg
-------
**语法:** *val = ngx.arg\[index\]*

**环境:** *set_by_lua&#42;, body_filter_by_lua&#42;*

当被用在 [set_by_lua](#set_by_lua) 或 [set_by_lua_file](#set_by_lua_file) 指令环境中时，本表是一个只读表，包含输入参数供配置命令使用：

```lua

 value = ngx.arg[n]
```

例如，

```nginx

 location /foo {
     set $a 32;
     set $b 56;

     set_by_lua $sum
         'return tonumber(ngx.arg[1]) + tonumber(ngx.arg[2])'
         $a $b;

     echo $sum;
 }
```

将输出 `88`，是 `32` 和 `56` 的和。

当被用在 [body_filter_by_lua](#body_filter_by_lua) 或 [body_filter_by_lua_file](#body_filter_by_lua_file) 指令环境中时，本表第一个元素是送给输出过滤器的输入数据块，第二个元素是 "eof" 布尔标记，用以标识整个输出数据流是否结束。

可以通过直接给相应的表元素赋值，设置送给下游 Nginx 输出过滤器的数据块和 "eof" 标记。当给 `ngx.arg[1]` 赋值 `nil` 或 Lua 空字符串时，将不发送任何数据给下游的 Nginx 输出过滤器。

[Back to TOC](#nginx-api-for-lua)

ngx.var.VARIABLE
----------------
**语法:** *ngx.var.VAR_NAME*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;*

读写 Nginx 变量值。

```nginx

 value = ngx.var.some_nginx_variable_name
 ngx.var.some_nginx_variable_name = value
```

请注意，只有已经定义的 nginx 变量可以被写入。例如，

```nginx

 location /foo {
     set $my_var ''; # 需要在设置时创建 $my_var 变量
     content_by_lua '
         ngx.var.my_var = 123;
         ...
     ';
 }
```

也就是说，nginx 变量无法“随用随创建”。

一些特殊的 nginx 变量，比如 `$args` 和 `$limit_rate`，可以被赋值，但许多其他的变量不能，包括 `$query_string`，`$arg_PARAMETER`，和 `$http_NAME` 等。

Nginx 正则表达式捕获组变量 `$1`、`$2`、`$3` 等，也可以通过这个界面读取，方式为通过 `ngx.var[1]`、`ngx.var[2]`、`ngx.var[3]` 等。

设置 `ngx.var.Foo` 为 `nil` 值将删除 Nginx 变量 `$Foo`。

```lua

 ngx.var.args = nil
```

**注意** 当从 Nginx 变量中读取值时，Nginx 将从基于请求的内存池中分配内存，只有在请求中止时才释放。所以如果用户的 Lua 代码中需要反复读取 Nginx 变量，请在用户程序的 Lua 变量中缓存，例如，

```lua

 local val = ngx.var.some_var
 --- 在后面反复使用变量 val
```

以避免在当前请求周期内的 (临时) 内存泄露。另外一个缓存结果的方法是使用 [ngx.ctx](#ngxctx) 表。

未定义的 NGINX 变量会被认定为 `nil` ，而未初始化（但已定义）的 NGINX 变量会被认定为空 Lua 字符串。

这个 API 需要进行相对“昂贵”的元方法调用，所以请避免高频使用。

[Back to TOC](#nginx-api-for-lua)

Core constants
--------------
**环境:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, &#42;log_by_lua&#42;, ngx.timer.&#42;*

```lua
    ngx.OK (0)
    ngx.ERROR (-1)
    ngx.AGAIN (-2)
    ngx.DONE (-4)
    ngx.DECLINED (-5)
```

请注意，这些常量中只有三个可以被 [Nginx API for Lua](#nginx-api-for-lua) 使用
(即 [ngx.exit](#ngxexit) 只接受 `NGX_OK`, `NGX_ERROR`, 和 `NGX_DECLINED` 作为输入)。

```lua

   ngx.null
```

`ngx.null` 常量是一个 `NULL` 的[轻量用户数据](http://www.lua.org/pil/28.5.html)，一般被用来表达 Lua table 等里面的 nil (空) 值，类似于 [lua-cjson](http://www.kyne.com.au/~mark/software/lua-cjson.php) 库中的 `cjson.null` 常量。在`v0.5.0rc5` 版本中首次引入这个常量。

`ngx.DECLINED` 这个常量在`v0.5.0rc19`版本中首次引入。

[Back to TOC](#nginx-api-for-lua)

HTTP method constants
---------------------
**环境:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;, ngx.timer.&#42;*

      ngx.HTTP_GET
      ngx.HTTP_HEAD
      ngx.HTTP_PUT
      ngx.HTTP_POST
      ngx.HTTP_DELETE
      ngx.HTTP_OPTIONS   (v0.5.0rc24 版本加入)
      ngx.HTTP_MKCOL     (v0.8.2 版本加入)
      ngx.HTTP_COPY      (v0.8.2 版本加入)
      ngx.HTTP_MOVE      (v0.8.2 版本加入)
      ngx.HTTP_PROPFIND  (v0.8.2 版本加入)
      ngx.HTTP_PROPPATCH (v0.8.2 版本加入)
      ngx.HTTP_LOCK      (v0.8.2 版本加入)
      ngx.HTTP_UNLOCK    (v0.8.2 版本加入)
      ngx.HTTP_PATCH     (v0.8.2 版本加入)
      ngx.HTTP_TRACE     (v0.8.2 版本加入)


这些常量一般被用在 [ngx.location.capture](#ngxlocationcapture) 和 [ngx.location.capture_multi](#ngxlocationcapture_multi) 方法中。

[Back to TOC](#nginx-api-for-lua)

HTTP status constants
---------------------
**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;, ngx.timer.&#42;*

```nginx

   value = ngx.HTTP_CONTINUE (100) (first added in the v0.9.20 release)
   value = ngx.HTTP_SWITCHING_PROTOCOLS (101) (first added in the v0.9.20 release)
   value = ngx.HTTP_OK (200)
   value = ngx.HTTP_CREATED (201)
   value = ngx.HTTP_ACCEPTED (202) (first added in the v0.9.20 release)
   value = ngx.HTTP_NO_CONTENT (204) (first added in the v0.9.20 release)
   value = ngx.HTTP_PARTIAL_CONTENT (206) (first added in the v0.9.20 release)
   value = ngx.HTTP_SPECIAL_RESPONSE (300)
   value = ngx.HTTP_MOVED_PERMANENTLY (301)
   value = ngx.HTTP_MOVED_TEMPORARILY (302)
   value = ngx.HTTP_SEE_OTHER (303)
   value = ngx.HTTP_NOT_MODIFIED (304)
   value = ngx.HTTP_TEMPORARY_REDIRECT (307) (first added in the v0.9.20 release)
   value = ngx.HTTP_BAD_REQUEST (400)
   value = ngx.HTTP_UNAUTHORIZED (401)
   value = ngx.HTTP_PAYMENT_REQUIRED (402) (first added in the v0.9.20 release)
   value = ngx.HTTP_FORBIDDEN (403)
   value = ngx.HTTP_NOT_FOUND (404)
   value = ngx.HTTP_NOT_ALLOWED (405)
   value = ngx.HTTP_NOT_ACCEPTABLE (406) (first added in the v0.9.20 release)
   value = ngx.HTTP_REQUEST_TIMEOUT (408) (first added in the v0.9.20 release)
   value = ngx.HTTP_CONFLICT (409) (first added in the v0.9.20 release)
   value = ngx.HTTP_GONE (410)
   value = ngx.HTTP_UPGRADE_REQUIRED (426) (first added in the v0.9.20 release)
   value = ngx.HTTP_TOO_MANY_REQUESTS (429) (first added in the v0.9.20 release)
   value = ngx.HTTP_CLOSE (444) (first added in the v0.9.20 release)
   value = ngx.HTTP_ILLEGAL (451) (first added in the v0.9.20 release)
   value = ngx.HTTP_INTERNAL_SERVER_ERROR (500)
   value = ngx.HTTP_METHOD_NOT_IMPLEMENTED (501)
   value = ngx.HTTP_BAD_GATEWAY (502) (first added in the v0.9.20 release)
   value = ngx.HTTP_SERVICE_UNAVAILABLE (503)
   value = ngx.HTTP_GATEWAY_TIMEOUT (504) (first added in the v0.3.1rc38 release)
   value = ngx.HTTP_VERSION_NOT_SUPPORTED (505) (first added in the v0.9.20 release)
   value = ngx.HTTP_INSUFFICIENT_STORAGE (507) (first added in the v0.9.20 release)
```

[返回目录](#nginx-api-for-lua)

Nginx log level constants
-------------------------
**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;, ngx.timer.&#42;*

```lua

   ngx.STDERR
   ngx.EMERG
   ngx.ALERT
   ngx.CRIT
   ngx.ERR
   ngx.WARN
   ngx.NOTICE
   ngx.INFO
   ngx.DEBUG
```

这些常量一般用于 [ngx.log](#ngxlog) 方法.

[Back to TOC](#nginx-api-for-lua)

print
-----
**语法:** *print(...)*

**环境:** *init_by_lua&#42;, init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;, ngx.timer.&#42;, balancer_by_lua&#42;, certificate_by_lua&#42;*

将参数值以 `ngx.NOTICE` 日志级别写入 nginx 的 `error.log` 文件。
Writes argument values into the nginx `error.log` file with the `ngx.NOTICE` log level.

等同于

```lua

 ngx.log(ngx.NOTICE, ...)
```

Lua 的 `nil` 值输出 `"nil"` 字符串，Lua 的布尔值输出 `"true"` 或 `"false"` 字符串。`ngx.null` 常量输出为 `"null"` 字符串。

在 Nginx 内核中硬编码限制了单条错误信息最长为 `2048` 字节。这个长度包含了最后的换行符和开始的时间戳。如果信息长度超过这个限制，Nginx 将把信息文本截断。这个限制可以通过修改 Nginx 源码中 `src/core/ngx_log.h` 文件中的 `NGX_MAX_ERROR_STR` 宏定义调整。

[Back to TOC](#nginx-api-for-lua)

ngx.ctx
-------
**环境:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;, ngx.timer.&#42;, balancer_by_lua&#42;*

本 Lua 表可以用来存储基于请求的 Lua 环境数据，其生存周期与当前请求相同 (类似 Nginx 变量)。

参考下面例子，

```nginx

 location /test {
     rewrite_by_lua '
         ngx.ctx.foo = 76
     ';
     access_by_lua '
         ngx.ctx.foo = ngx.ctx.foo + 3
     ';
     content_by_lua '
         ngx.say(ngx.ctx.foo)
     ';
 }
```

访问 `GET /test` 输出

```bash

 79
```

也就是说，`ngx.ctx.foo` 条目跨越一个请求的 rewrite (重写)，access (访问)，和 content (内容) 各处理阶段保持一致。

每个请求，包括子请求，都有一份自己的 `ngx.ctx` 表。例如：

```nginx

 location /sub {
     content_by_lua '
         ngx.say("sub pre: ", ngx.ctx.blah)
         ngx.ctx.blah = 32
         ngx.say("sub post: ", ngx.ctx.blah)
     ';
 }

 location /main {
     content_by_lua '
         ngx.ctx.blah = 73
         ngx.say("main pre: ", ngx.ctx.blah)
         local res = ngx.location.capture("/sub")
         ngx.print(res.body)
         ngx.say("main post: ", ngx.ctx.blah)
     ';
 }
```

访问 `GET /main` 输出

```bash

 main pre: 73
 sub pre: nil
 sub post: 32
 main post: 73
```

这里，在子请求中修改 `ngx.ctx.blah` 条目并不影响父请求中的同名条目，因为它们各自维护不同版本的 `ngx.ctx.blah`。

内部重定向将摧毁原始请求中的 `ngx.ctx` 数据 (如果有)，新请求将会有一个空白的 `ngx.ctx` 表。例如，

```nginx

 location /new {
     content_by_lua '
         ngx.say(ngx.ctx.foo)
     ';
 }

 location /orig {
     content_by_lua '
         ngx.ctx.foo = "hello"
         ngx.exec("/new")
     ';
 }
```

访问 `GET /orig` 将输出

```bash

 nil
```

而不是原始的 `"hello"` 值。

任意数据值，包括 Lua 闭包与嵌套表，都可以被插入这个“魔法”表，也允许注册自定义元方法。

也可以将 `ngx.ctx` 覆盖为一个新 Lua 表，例如，

```lua

 ngx.ctx = { foo = 32, bar = 54 }
```

当用在 [init_worker_by_lua*](#init_worker_by_lua) 环境中，这个表与当前 Lua 句柄生命周期相同。

The `ngx.ctx` lookup requires relatively expensive metamethod calls and it is much slower than explicitly passing per-request data along by your own function arguments. So do not abuse this API for saving your own function arguments because it usually has quite some performance impact.

`ngx.ctx` 表查询需要相对昂贵的元方法调用，这比通过用户自己的函数参数直接传递基于请求的数据要慢得多。所以不要为了节约用户函数参数而滥用此 API，因为它可能对性能有明显影响。

Because of the metamethod magic, never "local" the `ngx.ctx` table outside your Lua function scope on the Lua module level level due to [worker-level data sharing](#data-sharing-within-an-nginx-worker). For example, the following is bad:

而且由于元方法“魔法”，不要在 lua 模块级别试图使用 "local" 级别的 `ngx.ctx` ，例如 [worker-level data sharing](#data-sharing-within-an-nginx-worker)。下面示例是糟糕的：

```lua

 -- mymodule.lua
 local _M = {}

 -- 下面一行的 ngx.ctx 是属于单个请求的，但 `ctx` 变量是在 Lua 模块级别
 -- 并且属于单个 worker 的。
 local ctx = ngx.ctx

 function _M.main()
     ctx.foo = "bar"
 end

 return _M
```

应使用下面方式替代：

```lua

 -- mymodule.lua
 local _M = {}

 function _M.main(ctx)
     ctx.foo = "bar"
 end

 return _M
```

就是说，调用者对 `ctx` 表调用应通过函数传参方式完成。

[Back to TOC](#nginx-api-for-lua)

ngx.location.capture
--------------------
**语法:** *res = ngx.location.capture(uri, options?)*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

向 `uri` 发起一个同步非阻塞 *Nginx 子请求*。

Nginx 子请求是一种非常强有力的方式，它可以发起非阻塞的内部请求访问目标 location。目标 location 可以是配置文件中其他文件目录，或 *任何* 其他 nginx C 模块，包括 `ngx_proxy`、`ngx_fastcgi`、`ngx_memc`、`ngx_postgres`、`ngx_drizzle`，甚至 ngx_lua 自身等等 。

需要注意的是，子请求只是模拟 HTTP 接口的形式， *没有* 额外的 HTTP/TCP 流量，也 *没有* IPC (进程间通信) 调用。所有工作在内部高效地在 C 语言级别完成。

子请求与 HTTP 301/302 重定向指令 (通过 [ngx.redirect](#ngxredirect)) 完全不同，也与内部重定向 ((通过 [ngx.exec](#ngxexec)) 完全不同。

在发起子请求前，用户程序应总是读取完整的 HTTP 请求体 (通过调用 [ngx.req.read_body](#ngxreqread_body) 或设置 [lua_need_request_body](#lua_need_request_body) 指令为 on).

该 API 方法（[ngx.location.capture_multi](#ngxlocationcapture_multi) 也一样）总是缓冲整个请求体到内存中。因此，当需要处理一个大的子请求响应，用户程序应使用 [cosockets](#ngxsockettcp) 进行流式处理，

下面是一个简单例子：

```lua

 res = ngx.location.capture(uri)
```

返回一个包含四个元素的 Lua 表 (`res.status`, `res.header`, `res.body`, 和 `res.truncated`)。

`res.status` (状态) 保存子请求的响应状态码。

`res.header` (头) 用一个标准 Lua 表储子请求响应的所有头信息。如果是“多值”响应头，这些值将使用 Lua (数组) 表顺序存储。例如，如果子请求响应头包含下面的行：

```bash

 Set-Cookie: a=3
 Set-Cookie: foo=bar
 Set-Cookie: baz=blah
```

则 `res.header["Set-Cookie"]` 将存储 Lua 表 `{"a=3", "foo=bar", "baz=blah"}`。

`res.body` (体) 保存子请求的响应体数据，它可能被截断。用户需要检测 `res.truncated` (截断) 布尔值标记来判断 `res.body` 是否包含截断的数据。这种数据截断的原因只可能是因为子请求发生了不可恢复的错误，例如远端在发送响应体时过早中断了连接，或子请求在接收远端响应体时超时。

URI 请求串可以与 URI 本身连在一起，例如，

```lua

 res = ngx.location.capture('/foo/bar?a=3&b=4')
```

因为 Nginx 内核限制，子请求不允许类似 `@foo` 命名 location。请使用标准 location，并设置 `internal` 指令，仅服务内部请求。

可选的选项表可以作为第二个参数传入，支持以下选项：

* `method`
    指定子请求的请求方法, 只接受类似 `ngx.HTTP_POST` 的常量。
* `body`
    指定子请求的请求体 (仅接受字符串值)。
* `args`
    指定子请求的 URI 请求参数 (可以是字符串或者 Lua 表)。
* `ctx`
        指定一个 Lua 表作为子请求的 [ngx.ctx](#ngxctx) 表，可以是当前请求的 [ngx.ctx](#ngxctx) 表。这种方式可以让父请求和子请求共享完全相同的上下文环境。此选项最早出现在版本 `v0.3.1rc25` 中。
* `vars`
    用一个 Lua 表设置子请求中的 Nginx 变量值。此选项最早出现在版本 `v0.3.1rc31` 中。
* `copy_all_vars`
        设置是否复制所有当前请求的 Nginx 变量值到子请求中，修改子请求的 nginx 变量值不影响当前 (父) 请求。此选项最早出现在版本 `v0.3.1rc31` 中。
* `share_all_vars`
    设置是否共享所有当前 (父) 请求的 Nginx 变量值到子请求中，修改子请求的 nginx 变量值将影响当前 (父) 请求。应用此选项将可能导致非常难以发现的错误，这种副作用是非常有害的。所以只有当完全明确知道自己在做什么时才打开此选项。
* `always_forward_body`
        当设置为 true 时，如果没有设置 `body` 选项，当前 (父) 请求的请求体将被转发给子请求。被 [ngx.req.read_body()](#ngxreqread_body) 或 [lua_need_request_body on](#lua_need_request_body) 指令读取的请求体将直接转发给子请求，而不是在创建子请求时再复制整个请求体 (无论此请求体是缓存在内存中还是临时文件中)。默认情况下，此选项值为 `false`，当 `body` 选项没有设置，且当子请求的请求方法是 `PUT` 和 `POST` 时，当前 (父) 请求的请求体才被转发。

例如，发送一个 POST 子请求，可以这样做：

```lua

 res = ngx.location.capture(
     '/foo/bar',
     { method = ngx.HTTP_POST, body = 'hello, world' }
 )
```

除了 POST 的其他 HTTP 请求方法请参考 [HTTP method constants](#http-method-constants)。
`method` 选项默认值是 `ngx.HTTP_GET`。

`args` 选项可以设置附加的 URI 参数，例如：

```lua

 ngx.location.capture('/foo?a=1',
     { args = { b = 3, c = ':' } }
 )
```

等同于

```lua

 ngx.location.capture('/foo?a=1&b=3&c=%3a')
```

也就是说，这个方法将根据 URI 规则转义参数键和值，并将它们拼接在一起组成一个完整的请求串。`args` 选项要求的 Lua 表的格式与 [ngx.encode_args](#ngxencode_args) 方法中使用的完全相同。

`args` 选项也可以直接包含 (转义过的) 请求串：

```lua

 ngx.location.capture('/foo?a=1',
     { args = 'b=3&c=%3a' } }
 )
```

这个例子与上个例子的功能相同。

`share_all_vars` 选项控制是否让当前请求与其子请求共享 nginx 变量。如果设为 `true`，则当前请求与所有子请求共享所有 nginx 变量作用域。因此，在子请求中修改 nginx 变量将影响当前请求。

使用此选项时需要非常小心，因为共享变量作用域容易导致难以预测的副作用。一般来说，推荐使用 `args`、`vars`、或 `copy_all_vars` 来代替。

此选项默认值是 `false`。

```nginx

 location /other {
     set $dog "$dog world";
     echo "$uri dog: $dog";
 }

 location /lua {
     set $dog 'hello';
     content_by_lua '
         res = ngx.location.capture("/other",
             { share_all_vars = true });

         ngx.print(res.body)
         ngx.say(ngx.var.uri, ": ", ngx.var.dog)
     ';
 }
```

请求 location `/lua` 将输出


    /other dog: hello world
    /lua: hello world


`copy_all_vars` 在子请求被创建时，提供给它一份父请求的 Nginx 变量拷贝。在子请求中对这些变量的修改将不会影响父请求和其他任何共享父请求变量的子请求。

```nginx

 location /other {
     set $dog "$dog world";
     echo "$uri dog: $dog";
 }

 location /lua {
     set $dog 'hello';
     content_by_lua '
         res = ngx.location.capture("/other",
             { copy_all_vars = true });

         ngx.print(res.body)
         ngx.say(ngx.var.uri, ": ", ngx.var.dog)
     ';
 }
```

请求 `GET /lua` 会输出


    /other dog: hello world
    /lua: hello


请注意，当 `share_all_vars` 和 `copy_all_vars` 都被设置为 true 时，`share_all_vars` 优先。

除了上面提到的两个设置方法外，也可以通过配置 `vars` 选项来设置子请求的变量值。在共享或复制父请求的 nginx 变量值后，子请求被设置这些值。和把特定参数通过 URL 参数编码传入子请求，再在 Nginx 配置文件中解码相比，这个方法效率更高。

```nginx

 location /other {
     content_by_lua '
         ngx.say("dog = ", ngx.var.dog)
         ngx.say("cat = ", ngx.var.cat)
     ';
 }

 location /lua {
     set $dog '';
     set $cat '';
     content_by_lua '
         res = ngx.location.capture("/other",
             { vars = { dog = "hello", cat = 32 }});

         ngx.print(res.body)
     ';
 }
```

访问 `/lua` 将输出


    dog = hello
    cat = 32


选项 `ctx` 被用于给子请求设定一个自定义的 Lua 表作为 [ngx.ctx](#ngxctx) 表。

```nginx

 location /sub {
     content_by_lua '
         ngx.ctx.foo = "bar";
     ';
 }
 location /lua {
     content_by_lua '
         local ctx = {}
         res = ngx.location.capture("/sub", { ctx = ctx })

         ngx.say(ctx.foo);
         ngx.say(ngx.ctx.foo);
     ';
 }
```

请求 `GET /lua` 输出


    bar
    nil


也可以通过 `ctx` 选项设置当前 (父) 请求与子请求共享同一个 [ngx.ctx](#ngxctx) 表。

```nginx

 location /sub {
     content_by_lua '
         ngx.ctx.foo = "bar";
     ';
 }
 location /lua {
     content_by_lua '
         res = ngx.location.capture("/sub", { ctx = ngx.ctx })
         ngx.say(ngx.ctx.foo);
     ';
 }
```

请求 `GET /lua` 输出


    bar


请注意，通过 [ngx.location.capture](#ngxlocationcapture) 创建的子请求默认继承当前请求的所有请求头信息，这有可能导致子请求响应中不可预测的副作用。例如，当使用标准的 `ngx_proxy` 模块服务子请求时，如果主请求头中包含 "Accept-Encoding: gzip"，可能导致子请求返回 Lua 代码无法正确处理的 gzip 压缩过的结果。通过设置 [proxy_pass_request_headers](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass_request_headers) 为 `off` ，在子请求 location 中忽略原始请求头。

当没有设置 `body` 选项，且 `always_forward_body` 选项为 false (默认值) 时，`POST` 和 `PUT` 子请求将继承父请求的请求体 (如果有的话)。

Nginx 代码中有一个硬编码的数字，来控制每个主请求最多可以有多少个并发的子请求。在旧版 Nginx 中，这个数字是 `50`，自 Nginx `1.1.x` 以后，并发子请求数被提高到了 `200`。当超过这个限制时，`error.log` 文件中将出现下面的信息：


    [error] 13983#0: *1 subrequests cycle while processing "/uri"


如果需要修改这个限制，可以手工在 Nginx 源代码树的 `nginx/src/http/ngx_http_request.h` 文件中修改 `NGX_HTTP_MAX_SUBREQUESTS` 宏定义。

请参考 [subrequest directives of other modules](#locations-configured-by-subrequest-directives-of-other-modules) 了解目标 location 的配置限制。

[Back to TOC](#nginx-api-for-lua)

ngx.location.capture_multi
--------------------------
**语法:** *res1, res2, ... = ngx.location.capture_multi({ {uri, options?}, {uri, options?}, ... })*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

与 [ngx.location.capture](#ngxlocationcapture) 类似，并允许多个子请求并行访问。

此函数根据输入列表并行发起多个子请求，并按原顺序返回结果。例如，

```lua

 res1, res2, res3 = ngx.location.capture_multi{
     { "/foo", { args = "a=3&b=4" } },
     { "/bar" },
     { "/baz", { method = ngx.HTTP_POST, body = "hello" } },
 }

 if res1.status == ngx.HTTP_OK then
     ...
 end

 if res2.body == "BLAH" then
     ...
 end
```

此函数只有在所有子请求中止后才返回。
总等待时间是耗时最长一个子请求的时间，而不是所有子请求等待时间的总和。

当子请求数量预先未知时，可以通过 Lua 表来存储所有请求和返回。

```lua

 -- 创建请求表
 local reqs = {}
 table.insert(reqs, { "/mysql" })
 table.insert(reqs, { "/postgres" })
 table.insert(reqs, { "/redis" })
 table.insert(reqs, { "/memcached" })

 -- 同时执行所有请求，并等待所有返回
 local resps = { ngx.location.capture_multi(reqs) }

 -- 循环返回表
 for i, resp in ipairs(resps) do
     -- 处理返回表 "resp"
 end
```

[ngx.location.capture](#ngxlocationcapture) 函数是本函数的特殊形式。逻辑上说，[ngx.location.capture](#ngxlocationcapture) 函数可以按下面方法实现：

```lua

 ngx.location.capture =
     function (uri, args)
         return ngx.location.capture_multi({ {uri, args} })
     end
```

请参考 [subrequest directives of other modules](#locations-configured-by-subrequest-directives-of-other-modules) 了解目标 location 的配置限制。

[Back to TOC](#nginx-api-for-lua)

ngx.status
----------
**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;*

读写当前请求的响应状态码。这个方法需要在发送响应头前调用。

```lua

 ngx.status = ngx.HTTP_CREATED
 status = ngx.status
```

在发送响应头之后设置 `ngx.status` 不会生效，且 nginx 的错误日志中会有下面一条记录：


    attempt to set ngx.status after sending out response headers


[Back to TOC](#nginx-api-for-lua)

ngx.header.HEADER
-----------------
**语法:** *ngx.header.HEADER = VALUE*

**语法:** *value = ngx.header.HEADER*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;*

修改、添加、或清除当前请求待发送的 `HEADER` 响应头信息。

头名称中的下划线 (`_`) 将被默认替换为连字符 (`-`)。可以通过 [lua_transform_underscores_in_response_headers](#lua_transform_underscores_in_response_headers) 指令关闭这个替换。

头名称大小写不敏感。

```lua

 -- 与 ngx.header["Content-Type"] = 'text/plain' 相同
 ngx.header.content_type = 'text/plain';

 ngx.header["X-My-Header"] = 'blah blah';
```

多值头信息可以按以下方法设置：

```lua

 ngx.header['Set-Cookie'] = {'a=32; path=/', 'b=4; path=/'}
```

在响应头中将输出：

```bash

 Set-Cookie: a=32; path=/
 Set-Cookie: b=4; path=/
```

只接受 Lua 表输入 (对于类似 `Content-Type` 之类的只接受一个值的标准头信息，只有 Lua 表中的最后一个元素有效)。

```lua

 ngx.header.content_type = {'a', 'b'}
```

等同于

```lua

 ngx.header.content_type = 'b'
```

将一个头信息的值设为 `nil` 将从响应头中移除该输出：

```lua

 ngx.header["X-My-Header"] = nil;
```

赋值一个空 Lua 表效果相同：

```lua

 ngx.header["X-My-Header"] = {};
```

在输出响应头 (不管是显示的通过 [ngx.send_headers](#ngxsend_headers) 或隐式的通过类似 [ngx.print](#ngxprint) 指令) 以后设置 `ngx.header.HEADER` 将抛出 Lua 异常。

读取 `ngx.header.HEADER` 将返回响应头中名为 `HEADER` 的头信息的值。

读取时，头名称中的下划线 (`_`) 也会被替换成连字符 (`-`)，并且大小写不敏感。如果该头信息不存在，将返回 `nil`。

这个 API 在 [header_filter_by_lua](#header_filter_by_lua) 和 [header_filter_by_lua_file](#header_filter_by_lua_file) 环境中非常有用，例如：

```nginx

 location /test {
     set $footer '';

     proxy_pass http://some-backend;

     header_filter_by_lua '
         if ngx.header["X-My-Header"] == "blah" then
             ngx.var.footer = "some value"
         end
     ';

     echo_after_body $footer;
 }
```

对于多值头信息，所有值将被按顺序放入一个 Lua 表中，例如，响应头


    Foo: bar
    Foo: baz


在读取 `ngx.header.Foo` 时返回的结果为

```lua

 {"bar", "baz"}
```

需要注意的是，`ngx.header` 不是一个标准 Lua 表，不能通过 Lua 的 `ipairs` 函数进行迭代查询。

读取 *请求* 头信息，请使用 [ngx.req.get_headers](#ngxreqget_headers) 函数。

[Back to TOC](#nginx-api-for-lua)

ngx.resp.get_headers
--------------------
**语法:** *headers = ngx.resp.get_headers(max_headers?, raw?)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;, balancer_by_lua&#42;*

返回一个 Lua 表，包含当前请求的所有响应头信息。

```lua

 local h = ngx.resp.get_headers()
 for k, v in pairs(h) do
     ...
 end
```

此函数与 [ngx.req.get_headers](#ngxreqget_headers) 有相似之处，唯一区别是获取的是响应头信息而不是请求头信息。

这个 API 最早出现在 `v0.9.5` 版中。

[Back to TOC](#nginx-api-for-lua)

ngx.req.is_internal
-------------------
**语法:** *is_internal = ngx.req.is_internal()*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;*

返回一个布尔值，说明当前请求是否是一个“内部请求”，既：一个请求的初始化是在当前 nginx 服务端完成初始化，不是在客户端。

子请求都是内部请求，并且都是内部重定向后的请求。

该 API 在 `v0.9.20` 版本首次引入。

[返回目录](#nginx-api-for-lua)

ngx.req.start_time
------------------
**语法:** *secs = ngx.req.start_time()*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;*

返回当前请求创建时的时间戳，格式为浮点数，其中小数部分代表毫秒值。

以下用 Lua 代码模拟计算了 `$request_time` 变量值 (由 [ngx_http_log_module](http://nginx.org/en/docs/http/ngx_http_log_module.html) 模块生成)

```lua

 local request_time = ngx.now() - ngx.req.start_time()
```

这个函数在 `v0.7.7` 版本中首次引入。

更多使用方法请参考 [ngx.now](#ngxnow) 和 [ngx.update_time](#ngxupdate_time)。

[Back to TOC](#nginx-api-for-lua)

ngx.req.http_version
--------------------
**语法:** *num = ngx.req.http_version()*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;*

返回一个 Lua 数字代表当前请求的 HTTP 版本号。

当前的可能结果值为 1.0, 1.1 和 0.9。无法识别时值时返回 `nil`。

这个方法在 `v0.7.17` 版本中首次引入。

[Back to TOC](#nginx-api-for-lua)

ngx.req.raw_header
------------------
**语法:** *str = ngx.req.raw_header(no_request_line?)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;*

返回 Nginx 服务器接收到的原始 HTTP 协议头。

默认时，请求行和末尾的 `CR LF` 结束符也被包括在内。例如，

```lua

 ngx.print(ngx.req.raw_header())
```

输出结果类似：


    GET /t HTTP/1.1
    Host: localhost
    Connection: close
    Foo: bar



可以通过指定可选的 `no_request_line` 参数为 `true` 来去除结果中的请求行。例如，

```lua

 ngx.print(ngx.req.raw_header(true))
```

输出结果类似：

    Host: localhost
    Connection: close
    Foo: bar

这个方法在 `v0.7.17` 版本中首次引入。

[Back to TOC](#nginx-api-for-lua)

ngx.req.get_method
------------------
**语法:** *method_name = ngx.req.get_method()*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, balancer_by_lua&#42;*

获取当前请求的 HTTP 请求方法名称。结果为类似 `"GET"` 和 `"POST"` 的字符串，而不是 [HTTP 方法常量](#http-method-constants) 中定义的数值。

如果当前请求为 Nginx 子请求，将返回子请求的 HTTP 请求方法名称。

这个方法在 `v0.5.6` 版本中首次引入。

更多用法请参考 [ngx.req.set_method](#ngxreqset_method)。

[Back to TOC](#nginx-api-for-lua)

ngx.req.set_method
------------------
**语法:** *ngx.req.set_method(method_id)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;*

用 `method_id` 参数的值改写当前请求的 HTTP 请求方法。当前仅支持 [HTTP 请求方法](#http-method-constants) 中定义的数值常量，例如 `ngx.HTTP_POST` 和 `ngx.HTTP_GET`。

如果当前请求是 Nginx 子请求，子请求的 HTTP 请求方法将被改写。

这个方法在 `v0.5.6` 版本中首次引入。

更多用法请参考 [ngx.req.get_method](#ngxreqget_method)。

[Back to TOC](#nginx-api-for-lua)

ngx.req.set_uri
---------------
**语法:** *ngx.req.set_uri(uri, jump?)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;*

用 `uri` 参数重写当前请求 (已解析过的) URI。该 `uri` 参数必须是 Lua 字符串，并且长度不能是 0，否则将抛出 Lua 异常。

可选的布尔值参数 `jump` 会触发类似 [ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html) 中 [rewrite](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite) 指令的 location 重匹配 (或 location 跳转)。换句话说，当 `jump` 参数是 `true` (默认值 `false`) 时，此函数将不会返回，它会让 Nginx 在之后的 `post-rewrite` 执行阶段，根据新的 URI 重新搜索 location，并跳转到新 location。

默认值时，location 跳转不会被触发，只有当前请求的 URI 被改写。当 `jump` 参数值为 `false` 或不存在时，此函数将正常返回，但没有返回值。

例如，下面 nginx 配置片段

```nginx

 rewrite ^ /foo last;
```

可以通过 Lua 代码写成下面这样：

```lua

 ngx.req.set_uri("/foo", true)
```

类似的，Nginx 配置

```nginx

 rewrite ^ /foo break;
```

可以通过 Lua 代码写成：

```lua

 ngx.req.set_uri("/foo", false)
```

等同于写成：

```lua

 ngx.req.set_uri("/foo")
```

`jump` 参数只可以在 [rewrite_by_lua](#rewrite_by_lua) 和 [rewrite_by_lua_file](#rewrite_by_lua_file) 指令中被设置为 `true`。不能在其他环境中使用 jump，否则将抛出 Lua 异常。

下面的示例复杂一些，包含正则表达式替换：

```nginx

 location /test {
     rewrite_by_lua '
         local uri = ngx.re.sub(ngx.var.uri, "^/test/(.*)", "/$1", "o")
         ngx.req.set_uri(uri)
     ';
     proxy_pass http://my_backend;
 }
```

功能上等同于：

```nginx

 location /test {
     rewrite ^/test/(.*) /$1 break;
     proxy_pass http://my_backend;
 }
```

请注意，不能使用这个函数重写 URI 参数，应该使用 [ngx.req.set_uri_args](#ngxreqset_uri_args) 代替。例如，Nginx 配置

```nginx

 rewrite ^ /foo?a=3? last;
```

可以被写成

```nginx

 ngx.req.set_uri_args("a=3")
 ngx.req.set_uri("/foo", true)
```

或

```nginx

 ngx.req.set_uri_args({a = 3})
 ngx.req.set_uri("/foo", true)
```

这个方法在 `v0.3.1rc14` 版本中首次引入。

[Back to TOC](#nginx-api-for-lua)

ngx.req.set_uri_args
--------------------
**语法:** *ngx.req.set_uri_args(args)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;*

用 `args` 参数重写当前请求的 URI 请求参数。`args` 参数可以是一个 Lua 字符串，比如

```lua

 ngx.req.set_uri_args("a=3&b=hello%20world")
```

或一个包含请求参数 key-value 对的 Lua table，例如

```lua

 ngx.req.set_uri_args({ a = 3, b = "hello world" })
```

在第二种情况下，本方法将根据 URI 转义规则转义参数的 key 和 value。

本方法也支持多值参数：

```lua

 ngx.req.set_uri_args({ a = 3, b = {5, 6} })
```

此时请求参数字符串为 `a=3&b=5&b=6`。

这个方法在 `v0.3.1rc13` 版本中首次引入。

更多用法请参考 [ngx.req.set_uri](#ngxreqset_uri)。

[Back to TOC](#nginx-api-for-lua)

ngx.req.get_uri_args
--------------------
**语法:** *args = ngx.req.get_uri_args(max_args?)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;*

返回一个 Lua table，包含当前请求的所有 URL 查询参数。

```nginx

 location = /test {
     content_by_lua '
         local args = ngx.req.get_uri_args()
         for key, val in pairs(args) do
             if type(val) == "table" then
                 ngx.say(key, ": ", table.concat(val, ", "))
             else
                 ngx.say(key, ": ", val)
             end
         end
     ';
 }
```

访问 `GET /test?foo=bar&bar=baz&bar=blah` 将输出：

```bash

 foo: bar
 bar: baz, blah
```

多次出现同一个参数 key 时，将生成一个 Lua table，按顺序保存其所有 value。

key 和 value 将根据 URI 编码规则进行解码。访问上面的配置文件，`GET /test?a%20b=1%61+2` 将输出：

```bash

 a b: 1a 2
```

不包含 `=<value>` 部分的参数被视为布尔值参数。`POST /test`，请求体是 `foo&bar` 则输出：

```bash

 foo: true
 bar: true
```

换句话说，它们将被赋值 Lua 布尔值 `true`。但是，它们与空字符串值参数不同，如 `GET /test?foo=&bar=` 将输出：

```bash

 foo:
 bar:
```

没有 key 的参数将被忽略。例如 `GET /test?=hello&=world` 将没有任何输出。

支持通过 nginx 变量 `$args` (或在 Lua 中访问 `ngx.var.args`) 动态更新查询参数：

```lua

 ngx.var.args = "a=3&b=42"
 local args = ngx.req.get_uri_args()
```

这个例子里的 `args` table 将一直是：

```lua

 {a = 3, b = 42}
```

而无论实际请求查询串是什么内容。

请注意，为防止拒绝服务式攻击 (denial of service attacks)，默认最多解析前 100 个请求参数 (包括同名的)，更多的参数将直接忽略。

可选的 `max_args` 函数参数可以用来修改这个限制：

```lua

 local args = ngx.req.get_uri_args(10)
```

这个参数可以被设置为 0 以移除此限制，此时将解析所有接收到的请求参数：

```lua

 local args = ngx.req.get_uri_args(0)
```

强烈不推荐移除 `max_args` 限制。

[Back to TOC](#nginx-api-for-lua)

ngx.req.get_post_args
---------------------
**语法:** *args, err = ngx.req.get_post_args(max_args?)*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;*

返回一个 Lua table，包含当前请求的所有 POST 查询参数 (MIME type 是 `application/x-www-form-urlencoded`)。使用前需要调 用 [ngx.req.read_body](#ngxreqread_body) 读取完整请求体，或通过设置 [lua_need_request_body](#lua_need_request_body) 指令为 on 以避免报错。

```nginx

 location = /test {
     content_by_lua '
         ngx.req.read_body()
         local args, err = ngx.req.get_post_args()
         if not args then
             ngx.say("failed to get post args: ", err)
             return
         end
         for key, val in pairs(args) do
             if type(val) == "table" then
                 ngx.say(key, ": ", table.concat(val, ", "))
             else
                 ngx.say(key, ": ", val)
             end
         end
     ';
 }
```

请求

```bash

 # Post request with the body 'foo=bar&bar=baz&bar=blah'
 $ curl --data 'foo=bar&bar=baz&bar=blah' localhost/test
```

将输出：

```bash

 foo: bar
 bar: baz, blah
```

多次出现同一个参数 key 时，将生成一个 Lua table，按顺序保存其所有 value。

key 和 value 将根据 URI 编码规则进行解码。

访问上面的配置文件，

```bash

 # POST request with body 'a%20b=1%61+2'
 $ curl -d 'a%20b=1%61+2' localhost/test
```

将输出：

```bash

 a b: 1a 2
```

不包含 `=<value>` 部分的参数被视为布尔值参数。例如 `POST /test` 的请求体是 `foo&bar` 时输出：

```bash

 foo: true
 bar: true
```

换句话说，它们将被赋值 Lua 布尔值 `true`。但是，它们与空字符串值参数不同，如 `POST /test` 的请求体是 `foo=&bar=` 将输出：

```bash

 foo:
 bar:
```

没有 key 的参数将被忽略。例如 `POST /test` 的请求体是 `=hello&=world` 时将没有任何输出。

请注意，为防止拒绝服务式攻击 (denial of service attacks)，默认最多解析前 100 个请求参数 (包括同名的)，更多的参数将直接忽略。

可选的 `max_args` 函数参数可以用来修改这个限制：

```lua

 local args = ngx.req.get_post_args(10)
```

这个参数可以被设置为 0 以移除此限制，此时将解析所有接收到的请求参数：

```lua

 local args = ngx.req.get_post_args(0)
```

强烈不推荐移除 `max_args` 限制。

[Back to TOC](#nginx-api-for-lua)

ngx.req.get_headers
-------------------
**语法:** *headers = ngx.req.get_headers(max_headers?, raw?)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;*

返回一个 Lua table，包含当前请求的所有请求头信息。

```lua

 local h = ngx.req.get_headers()
 for k, v in pairs(h) do
     ...
 end
```

读某一个头信息：

```lua

 ngx.say("Host: ", ngx.req.get_headers()["Host"])
```

请注意，[ngx.var.HEADER](#ngxvarvariable) API 使用 nginx 内核 [$http_HEADER](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_http) 变量，在读取单个请求头信息时更加适用。

如果请求头中多次出现某一字段，例如：

```bash

 Foo: foo
 Foo: bar
 Foo: baz
```

`ngx.req.get_headers()["Foo"]` 的值会是一个 Lua (数组) table，类似：

```lua

 {"foo", "bar", "baz"}
```

请注意，为防止拒绝服务式攻击 (denial of service attacks)，默认最多解析前 100 个请求头信息 (包括同名的)，更多的头信息将直接忽略。

可选的 `max_headers` 函数参数可以用来修改这个限制：

```lua

 local headers = ngx.req.get_headers(10)
```

这个参数可以被设置为 0 以移除此限制，此时将解析所有接收到的请求头信息：

```lua

 local headers = ngx.req.get_headers(0)
```

强烈不推荐移除 `max_headers` 限制。

自 `0.6.9` 版本开始，默认情况下，返回的 Lua table 中的所有头名称被转换成纯小写字母形式，除非设置 `raw` 参数为 `true` (默认是 `false`)。

同时，默认情况下，Lua table 中将被加入 `__index` 元方法 ([metamethod](http://www.lua.org/pil/13.4.1.html))， 用来在查询失败时，将 key 转换为全小写字母、且下划线变为连字符后，重新搜索。例如，如果请求头信息中有 `My-Foo-Header` 字段，下面的调用都将正确取出值：

```lua

 ngx.say(headers.my_foo_header)
 ngx.say(headers["My-Foo-Header"])
 ngx.say(headers["my-foo-header"])
```

当 `raw` 参数设置为 `true` 时，`__index` 元方法不会被加入。

[Back to TOC](#nginx-api-for-lua)

ngx.req.set_header
------------------
**语法:** *ngx.req.set_header(header_name, header_value)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;*

将当前请求的名为 `header_name` 的头信息值设置为 `header_value`，如果此头信息名称已经存在，修改其值。

默认时，之后通过 [ngx.location.capture](#ngxlocationcapture) 和 [ngx.location.capture_multi](#ngxlocationcapture_multi) 发起的所有子请求都将继承新的头信息。

下面的例子中将设置 `Content-Type` 头信息：

```lua

 ngx.req.set_header("Content-Type", "text/css")
```

`header_value` 可以是一个值数组，例如：

```lua

 ngx.req.set_header("Foo", {"a", "abc"})
```

将生成两个新的请求头信息：

```bash

 Foo: a
 Foo: abc
```

如果原来已经有名为 `Foo` 的头信息，将被覆盖。

当 `header_value` 参数是 `nil` 时，此请求头将被移除。所以，

```lua

 ngx.req.set_header("X-Foo", nil)
```

等同于

```lua

 ngx.req.clear_header("X-Foo")
```

[Back to TOC](#nginx-api-for-lua)

ngx.req.clear_header
--------------------
**语法:** *ngx.req.clear_header(header_name)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;*

清除当前请求的名为 `header_name` 的请求头信息。已经存在的子请求不受影响，此命令之后发起的子请求将默认继承修改后的头信息。

[Back to TOC](#nginx-api-for-lua)

ngx.req.read_body
-----------------
**语法:** *ngx.req.read_body()*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

同步读取客户端请求体，不阻塞 Nginx 事件循环。

```lua

 ngx.req.read_body()
 local args = ngx.req.get_post_args()
```

如果已经通过打开 [lua_need_request_body](#lua_need_request_body) 选项或其他模块读取请求体，此函数将不会执行，立即返回。

如果已经通过 [ngx.req.discard_body](#ngxreqdiscard_body) 函数或其他模块明确丢弃请求体，此函数将不会执行，立即返回。

当出错时，例如读取数据时连接出错，此方法将立即抛出 Lua 异常 *或* 以 500 状态码中断当前请求。

通过此函数读取的请求体，之后可以通过 [ngx.req.get_body_data](#ngxreqget_body_data) 获得，或者，通过 [ngx.req.get_body_file](#ngxreqget_body_file) 得到请求体数据缓存在磁盘上的临时文件名。这取决于：

1. 是否当前读求体已经大于 [client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size)，
1. 是否 [client_body_in_file_only](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_in_file_only) 选项被打开。

在当前请求中包含请求体，但不需要时，必须使用 [ngx.req.discard_body](#ngxreqdiscard_body) 明确丢弃请求体，以避免影响 HTTP 1.1 长连接或 HTTP 1.1 流水线 (pipelining)。

这个函数在 `v0.3.1rc17`` 版本中首次引入。

[Back to TOC](#nginx-api-for-lua)

ngx.req.discard_body
--------------------
**语法:** *ngx.req.discard_body()*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

明确丢弃请求体，也就是说，读取连接中的数据后立即丢弃。请注意，忽略请求体并不是丢弃请求体的正确方式，为避免破坏 HTTP 1.1 长连接或 HTTP 1.1 流水线 (pipelining)，必须使用本函数。

这个函数是异步调用，将立即返回。

如果请求体已经被读取，此函数将不会执行，立即返回。

这个函数在 `v0.3.1rc17`` 版本中首次引入。

更多用法请参考 [ngx.req.read_body](#ngxreqread_body)。

[Back to TOC](#nginx-api-for-lua)

ngx.req.get_body_data
---------------------
**语法:** *data = ngx.req.get_body_data()*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

取回内存中的请求体数据。本函数返回 Lua 字符串而不是包含解析过参数的 Lua table。如果想要返回 Lua table，请使用 [ngx.req.get_post_args](#ngxreqget_post_args) 函数。

当以下情况时，此函数返回 `nil`，

1. 请求体尚未被读取，
1. 请求体已经被存入磁盘上的临时文件，
1. 或请求体大小是 0。

如果请求体尚未被读取，请先调用 [ngx.req.read_body](#ngxreqread_body) (或打开 [lua_need_request_body](#lua_need_request_body) 选项强制本模块读取请求体。此方法不推荐）。

如果请求体已经被存入临时文件，请使用 [ngx.req.get_body_file](#ngxreqget_body_file) 函数代替。

如需要强制在内存中保存请求体，请设置 [client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size) 和 [client_max_body_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size) 为同样大小。

请注意，调用此函数比使用 `ngx.var.request_body` 或 `ngx.var.echo_request_body` 更有效率，因为本函数能够节省一次内存分配与数据复制。

这个函数在 `v0.3.1rc17`` 版本中首次引入。

更多用法请参考 [ngx.req.get_body_file](#ngxreqget_body_file)。

[Back to TOC](#nginx-api-for-lua)

ngx.req.get_body_file
---------------------
**语法:** *file_name = ngx.req.get_body_file()*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

获取存储请求体数据的临时文件名。如果请求体尚未被读取或已被读取到内存中，此函数将返回 `nil`。

本函数返回的文件是只读的，通常情况下会被 Nginx 的内存池清理机制清理。不应该手工修改、更名或删除这个文件。

如果请求体尚未被读取，请先调用 [ngx.req.read_body](#ngxreqread_body) (或打开 [lua_need_request_body](#lua_need_request_body) 选项强制本模块读取请求体。此方法不推荐）。

如果请求体已经被读入内存，请使用 [ngx.req.get_body_data](#ngxreqget_body_data) 函数代替。

如需要强制在临时文件中保存请求体，请打开 [client_body_in_file_only](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_in_file_only) 选项。

这个函数在 `v0.3.1rc17`` 版本中首次引入。

更多用法请参考 [ngx.req.get_body_data](#ngxreqget_body_data)。

[Back to TOC](#nginx-api-for-lua)

ngx.req.set_body_data
---------------------
**语法:** *ngx.req.set_body_data(data)*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

使用 `data` 参数指定的内存数据设置当前请求的请求体。

如果当前请求的请求体尚未被读取，它将被安全地丢弃。当请求体已经被读进内存或缓存在磁盘文件中时，相应的内存或磁盘文件将被立即清理回收。

这个函数在 `v0.3.1rc18`` 版本中首次引入。

更多用法请参考 [ngx.req.set_body_file](#ngxreqset_body_file)。

[Back to TOC](#nginx-api-for-lua)

ngx.req.set_body_file
---------------------
**语法:** *ngx.req.set_body_file(file_name, auto_clean?)*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

使用 `file_name` 参数指定的数据文件设置当前请求的请求体。

当可选参数 `auto_clean` 设置为 `true` 时，在本次请求完成，或本次请求内再次调用本函数或 [ngx.req.set_body_data](#ngxreqset_body_data) 时，`file_name` 文件将被删除。`auto_clean` 默认值是 `false`。

请确保 `file_name` 参数指定的文件存在，并设置合适的操作权限对 Nginx worker 可读，以避免抛出 Lua 异常。

如果当前请求的请求体尚未被读取，它将被安全地丢弃。当请求体已经被读进内存或缓存在磁盘文件中时，相应的内存或磁盘文件将被立即清理回收。

这个函数在 `v0.3.1rc18`` 版本中首次引入。

更多用法请参考 [ngx.req.set_body_data](#ngxreqset_body_data)。

[Back to TOC](#nginx-api-for-lua)

ngx.req.init_body
-----------------
**语法:** *ngx.req.init_body(buffer_size?)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

为当前请求创建一个新的空请求体并初始化缓冲区，为后续通过 [ngx.req.append_body](#ngxreqappend_body) 和 [ngx.req.finish_body](#ngxreqfinish_body) API 写请求体数据做好准备。

如果设置了 `buffer_size` 参数，将设置该大小的内存缓冲区，用于后续的 [ngx.req.append_body](#ngxreqappend_body) 写请求体数据。如果省略此参数，将使用 Nginx 标准指令 [client_body_buffer_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size) 中设置的值作为缓冲区大小。

当请求体数据过大，不再能保存在内存缓冲区中时，数据将被写入一个临时文件，类似 Nginx 内核中的标准请求体处理方式。

需要强调的是，在当前请求的所有请求体被写入完成后，必须调用 [ngx.req.finish_body](#ngxreqfinish_body) 以结束写入。另外，当此函数与 [ngx.req.socket](#ngxreqsocket) 一起使用时，需要在执行此函数 *之前* 调用 [ngx.req.socket](#ngxreqsocket)，否则将会报 "request body already exists" (请求体已经存在) 错。

此函数典型用法如下：

```lua

 ngx.req.init_body(128 * 1024)  -- 缓冲区 128KB
 for chunk in next_data_chunk() do
     ngx.req.append_body(chunk) -- 每块可以是 4KB
 end
 ngx.req.finish_body()
```

此函数可以与 [ngx.req.append_body](#ngxreqappend_body)，[ngx.req.finish_body](#ngxreqfinish_body)，和 [ngx.req.socket](#ngxreqsocket) 一起，使用纯 Lua 语言实现高效的输入过滤器 (在 [rewrite_by_lua](#rewrite_by_lua)* 或 [access_by_lua](#access_by_lua)* 环境中)，与其他 Nginx 内容处理程序或上游模块例如 [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) 和 [ngx_http_fastcgi_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html) 配合使用。
<!--todo content handler 翻译需要统一 -->

这个函数在 `v0.5.11` 版本中首次引入。

[Back to TOC](#nginx-api-for-lua)

ngx.req.append_body
-------------------
**语法:** *ngx.req.append_body(data_chunk)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

向已存在的请求体中追加写入 `data_chunk` 参数指定的新数据块，此请求体最初使用 [ngx.req.init_body](#ngxreqinit_body) 创建。

当请求体数据过大，不再能保存在内存缓冲区中时，数据将被写入一个临时文件，类似 Nginx 内核中的标准请求体处理方式。

需要强调的是，在当前请求的所有请求体被写入完成后，必须调用 [ngx.req.finish_body](#ngxreqfinish_body) 以结束写入。

此函数可以与 [ngx.req.init_body](#ngxreqinit_body)，[ngx.req.finish_body](#ngxreqfinish_body)，和 [ngx.req.socket](#ngxreqsocket) 一起，使用纯 Lua 语言实现高效的输入过滤器 (在 [rewrite_by_lua](#rewrite_by_lua)* 或 [access_by_lua](#access_by_lua)* 环境中)，与其他 Nginx 内容处理程序或上游模块例如 [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) 和 [ngx_http_fastcgi_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html) 配合使用。
<!--todo content handler 翻译需要统一 -->

这个函数在 `v0.5.11` 版本中首次引入。

更多用法请参考 [ngx.req.init_body](#ngxreqinit_body)。

[Back to TOC](#nginx-api-for-lua)

ngx.req.finish_body
-------------------
**语法:** *ngx.req.finish_body()*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

结束新请求体构造过程，该请求体由 [ngx.req.init_body](#ngxreqinit_body) 和 [ngx.req.append_body](#ngxreqappend_body) 创建。

此函数可以与 [ngx.req.init_body](#ngxreqinit_body)，[ngx.req.append_body](#ngxreqappend_body)，和 [ngx.req.socket](#ngxreqsocket) 一起，使用纯 Lua 语言实现高效的输入过滤器 (在 [rewrite_by_lua](#rewrite_by_lua)* 或 [access_by_lua](#access_by_lua)* 环境中)，与其他 Nginx 内容处理程序或上游模块例如 [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) 和 [ngx_http_fastcgi_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html) 配合使用。
<!--todo content handler 翻译需要统一 -->

这个函数在 `v0.5.11` 版本中首次引入。

更多用法请参考 [ngx.req.init_body](#ngxreqinit_body)。

[Back to TOC](#nginx-api-for-lua)

ngx.req.socket
--------------
**语法:** *tcpsock, err = ngx.req.socket()*

**语法:** *tcpsock, err = ngx.req.socket(raw)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

返回一个包含下游连接的只读 cosocket 对象。只有 [receive](#tcpsockreceive) 和 [receiveuntil](#tcpsockreceiveuntil) 方法在该对象上是支持的。
Returns a read-only cosocket object that wraps the downstream connection. Only [receive](#tcpsockreceive) and [receiveuntil](#tcpsockreceiveuntil) methods are supported on this object.

错误情况，将返回 `nil` 和错误字符描述信息。
In case of error, `nil` will be returned as well as a string describing the error.

通过该方法返回的 socket 对象，通常是用流式格式读取当前请求体。不要开启 [lua_need_request_body](#lua_need_request_body) 指令，并且不要混合调用 [ngx.req.read_body](#ngxreqread_body) 和 [ngx.req.discard_body](#ngxreqdiscard_body)。
The socket object returned by this method is usually used to read the current request's body in a streaming fashion. Do not turn on the [lua_need_request_body](#lua_need_request_body) directive, and do not mix this call with [ngx.req.read_body](#ngxreqread_body) and [ngx.req.discard_body](#ngxreqdiscard_body).

如果任何的请求体数据已经被预读到 Nginx 内核请求缓冲区，得到的 cosocket 对象需要小心对待，应避免由这种预读导致的潜在数据丢失。
If any request body data has been pre-read into the Nginx core request header buffer, the resulting cosocket object will take care of this to avoid potential data loss resulting from such pre-reading.
Chunked request bodies are not yet supported in this API.

从 `v0.9.0` 版本开始，该函数接受一个可选的布尔值参数 `raw` 。当该参数为 `true` 时，该方法将返回一个包含原生下游连接的全双工 cosocket 对象，你能对它调用 [receive](#tcpsockreceive)， [receiveuntil](#tcpsockreceiveuntil) 和 [send](#tcpsocksend) 。
Since the `v0.9.0` release, this function accepts an optional boolean `raw` argument. When this argument is `true`, this function returns a full-duplex cosocket object wrapping around the raw downstream connection socket, upon which you can call the [receive](#tcpsockreceive), [receiveuntil](#tcpsockreceiveuntil), and [send](#tcpsocksend) methods.

当指定 `raw` 参数为 `true` ，这里需要没有任何来自 [ngx.say](#ngxsay)、[ngx.print](#ngxprint) 或 [ngx.send_headers](#ngxsend_headers) 方法调用的待处理数据。所以如果你有下游输出调用，你应当在调用 `ngx.req.socket(true)` 之前调用 [ngx.flush(true)](#ngxflush) 确保这里没有任何待处理数据。如果请求体还没有读取，那么这个“原生 socket”也能用来读取请求体。
When the `raw` argument is `true`, it is required that no pending data from any previous [ngx.say](#ngxsay), [ngx.print](#ngxprint), or [ngx.send_headers](#ngxsend_headers) calls exists. So if you have these downstream output calls previously, you should call [ngx.flush(true)](#ngxflush) before calling `ngx.req.socket(true)` to ensure that there is no pending output data. If the request body has not been read yet, then this "raw socket" can also be used to read the request body.

你可以使用通过 `ngx.req.socket(true)` 返回的“原生请求 socket”来实现各种样式协议如 [WebSocket](http://en.wikipedia.org/wiki/WebSocket) ，或仅发出自己的 HTTP 请求头或体数据。真实世界，你可以参考 [lua-resty-websocket](https://github.com/openresty/lua-resty-websocket) 库。
You can use the "raw request socket" returned by `ngx.req.socket(true)` to implement fancy protocols like [WebSocket](http://en.wikipedia.org/wiki/WebSocket), or just emit your own raw HTTP response header or body data. You can refer to the [lua-resty-websocket library](https://github.com/openresty/lua-resty-websocket) for a real world example.

该函数是在 `v0.5.0rc1` 版本首次引入的。

[返回目录](#nginx-api-for-lua)

ngx.exec
--------
**语法:** *ngx.exec(uri, args?)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

使用 `uri`、`args` 参数执行一个内部跳转，与 [echo-nginx-module](http://github.com/openresty/echo-nginx-module) 的 [echo_exec](http://github.com/openresty/echo-nginx-module#echo_exec) 指令有些相似。

```lua

 ngx.exec('/some-location');
 ngx.exec('/some-location', 'a=3&b=5&c=6');
 ngx.exec('/some-location?a=3&b=5', 'c=6');
```

可选第二个参数 `args` 可被用来指名额外的 URI 查询参数，例如：

```lua

 ngx.exec("/foo", "a=3&b=hello%20world")
```

另外，对于 `args` 参数可使用一个 Lua 表，内部通过 ngx_lua 完成 URI 转义和字符串的连接。

```lua

 ngx.exec("/foo", { a = 3, b = "hello world" })
```

该结果和上一个示例是一样的。

作为 `args` 参数的 Lua 表格式化结果，与使用 [ngx.encode_args](#ngxencode_args) 方法格式化结果是相同的。

命名 location 也是支持的，但是第二个 `args` 参数将被忽略（如果存在的话），并且对新目标地址的查询字符串会从引用 location （如果有）中继承。

下面的例子中， `GET /foo/file.php?a=hello` 将返回 "hello" ，而不是 "goodbye" ：

```nginx

 location /foo {
     content_by_lua '
         ngx.exec("@bar", "a=goodbye");
     ';
 }

 location @bar {
     content_by_lua '
         local args = ngx.req.get_uri_args()
         for key, val in pairs(args) do
             if key == "a" then
                 ngx.say(val)
             end
         end
     ';
 }
```

注意，`ngx.exec` 方法与 [ngx.redirect](#ngxredirect) 是完全不同的，前者是个纯粹的内部跳转并且没有引入任何额外 HTTP 信号。

注意，此方法的调用终止当前请求的处理，并且它 *必须* 在 [ngx.send_headers](#ngxsend_headers) 或明确有响应体应答（比如 [ngx.print](#ngxprint) 或 [ngx.say](#ngxsay) 之一）之前调用。

该方法调用与 `return` 语句联合使用，是推荐的代码样式，例如，通过`return ngx.redirect(...)`，可使用在 [header_filter_by_lua](#header_filter_by_lua) 之外的环境中加强处理该请求被终止。

[返回目录](#nginx-api-for-lua)

ngx.redirect
------------
**语法:** *ngx.redirect(uri, status?)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

发出一个 HTTP `301` 或 `302` 重定向到 `uri`。

可选项 `status` 参数指定 `301` 或 `302` 哪个被使用。 默认使用 `302` （`ngx.HTTP_MOVED_TEMPORARILY`）。

假设当前服务名是 `localhost` 并且监听端口是 1984，这里有个例子：

```lua

 return ngx.redirect("/foo")
```

和下面例子是等价的：

```lua

 return ngx.redirect("/foo", ngx.HTTP_MOVED_TEMPORARILY)
```

重定向到任意外部 URL 也是支持的，例如：

```lua

 return ngx.redirect("http://www.google.com")
```

我们也可以使用数值类型的值当做第二个 `status` 参数：

```lua

 return ngx.redirect("/foo", 301)
```

该方法与使用 `redirect` 修饰的 [rewrite](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite) 标准指令 [ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html) 有些相似，例如，该 `nginx.conf` 片段：

```nginx

 rewrite ^ /foo? redirect;  # nginx config
```

与下面 Lua 代码是等价的：

```lua

 return ngx.redirect('/foo');  -- Lua code
```

当这样

```nginx

 rewrite ^ /foo? permanent;  # nginx config
```

等同于：

```lua

 return ngx.redirect('/foo', ngx.HTTP_MOVED_PERMANENTLY)  -- Lua code
```

也可以指定具体 URI 参数，例如：、

```lua

 return ngx.redirect('/foo?a=3&b=4')
```

注意，此方法的调用终止当前请求的处理，并且它 *必须* 在 [ngx.send_headers](#ngxsend_headers) 或明确有响应体应答（比如 [ngx.print](#ngxprint) 或 [ngx.say](#ngxsay) 之一）之前调用。

该方法调用与 `return` 语句联合使用，是推荐的代码样式，例如，通过`return ngx.redirect(...)`，可使用在 [header_filter_by_lua](#header_filter_by_lua) 之外的环境中加强处理该请求被终止。

[返回目录](#nginx-api-for-lua)

ngx.send_headers
----------------
**语法:** *ok, err = ngx.send_headers()*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

指名把应答头发送出去。

自从 `v0.8.3` 版本开始，成功情况该函数返回 `1` ，否则返回 `nil` 和错误字符描述信息。

注意，在内容通过 [ngx.say](#ngxsay)、[ngx.print](#ngxprint) 输出或当 [content_by_lua](#content_by_lua) 存在时，ngx_lua 将自动发送头部内容，所以通常情况不需要手动发送应答头。

[返回目录](#nginx-api-for-lua)

ngx.headers_sent
----------------
**语法:** *value = ngx.headers_sent*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua**

如果应答头部已经被发送（通过 ngx_lua）返回 `true` ，否则返回 `false` 。

该 API 是在 `v0.3.1rc6` 版本首次引入的。

[返回目录](#nginx-api-for-lua)

ngx.print
---------
**语法:** *ok, err = ngx.print(...)*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

将输入参数合并发送给 HTTP 客户端 (作为 HTTP 响应体)。如果此时还没有发送响应头信息，本函数将先发送 HTTP 响应头，再输出响应体。

自版本 `v0.8.3` 起，本函数当成功时返回 `1`，失败时返回 `nil` 以及一个描述错误的字符串。

Lua 的 `nil` 值输出 `"nil"` 字符串，Lua 的布尔值输出 `"true"` 或 `"false"` 字符串。

输入允许字符串嵌套数组，数组中所有元素相按顺序输出。

```lua

 local table = {
     "hello, ",
     {"world: ", true, " or ", false,
         {": ", nil}}
 }
 ngx.print(table)
```

将输出

```bash

 hello, world: true or false: nil
```

非数组表(哈希表)参数将导致抛出 Lua 异常。

`ngx.null` 常量输出为 `"null"` 字符串。

本函数为异步调用，将立即返回，不会等待所有数据被写入系统发送缓冲区。要以同步模式运行，请在调用 `ngx.print` 之后调用 `ngx.flush(true)`。这种方式在流式输出时非常有用。更多细节请参考 [ngx.flush](#ngxflush)。

请注意，`ngx.print` 和 [ngx.say](#ngxsay) 都会调用 Nginx body 输出过滤器，这种操作非常“昂贵”。所以，在“热”循环中使用这两个函数要非常小心；可以通过 Lua 进行缓存以节约调用。

[Back to TOC](#nginx-api-for-lua)

ngx.say
-------
**语法:** *ok, err = ngx.say(...)*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

与 [ngx.print](#ngxprint) 相同,同时末尾添加一个回车符。

[Back to TOC](#nginx-api-for-lua)

ngx.log
-------
**语法:** *ngx.log(log_level, ...)*

**环境:** *init_by_lua&#42;, init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

将参数拼接起来，按照设定的日志级别记入 error.log。

Lua `nil` 参数将输出 `"nil"` 字符串；Lua 布尔参数将输出 `"true"` 或 `"false"` 字符串；`ngx.null` 常量将输出 `"null"` 字符串。

`log_level` 参数可以使用类似 `ngx.ERR` 和 `ngx.WARN` 的常量。更多信息请参考 [Nginx log level constants](#nginx-log-level-constants)。

在 Nginx 内核中硬编码限制了单条错误信息最长为 `2048` 字节。这个长度包含了最后的换行符和开始的时间戳。如果信息长度超过这个限制，Nginx 将把信息文本截断。这个限制可以通过修改 Nginx 源码中 `src/core/ngx_log.h` 文件中的 `NGX_MAX_ERROR_STR` 宏定义调整。

[Back to TOC](#nginx-api-for-lua)

ngx.flush
---------
**语法:** *ok, err = ngx.flush(wait?)*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

向客户端刷新响应输出。

自 `v0.3.1rc34` 版本开始，`ngx.flush` 接受一个布尔型可选参数 `wait` (默认值 `false`)。当通过默认参数调用时，本函数发起一个异步调用 (将直接返回，不等待输出数据被写入系统发送缓冲区)。当把 `wait` 参数设置为 `true` 时，本函数将以同步模式执行。

在同步模式下，本函数不会立即返回，一直到所有输出数据被写入系统输出缓冲区，或者到达发送超时  [send_timeout](http://nginx.org/en/docs/http/ngx_http_core_module.html#send_timeout) 时间。请注意，因为使用了 Lua 协程机制，本函数即使在同步模式下也不会阻塞 Nginx 事件循环。

当 `ngx.flush(true)` 在 [ngx.print](#ngxprint) 或 [ngx.say](#ngxsay) 之后被立刻调用时，它将使这两个函数以同步模式执行。这在流式输出时非常有用。

请注意，`ngx.flush` 在 HTTP 1.0 缓冲输出模式下不起作用。详情请参考 [HTTP 1.0 support](#http-10-support)。

自 `v0.8.3` 版本开始，本函数执行成功是返回 `1`，否则返回 `nil` 和错误信息串。

[Back to TOC](#nginx-api-for-lua)

ngx.exit
--------
**语法:** *ngx.exit(status)*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, ngx.timer.&#42;*

当 `status >= 200` (即 `ngx.HTTP_OK` 及以上) 时，本函数中断当前请求执行并返回状态值给 nginx。

当 `status == 0` (即 `ngx.OK`) 时，本函数退出当前的“处理阶段句柄” (或当使用 [content_by_lua](#content_by_lua) 指令时的“内容句柄”) ，继续执行当前请求的下一个阶段 (如果有)。

`status` 参数可以是 `status` argument can be `ngx.OK`, `ngx.ERROR`, `ngx.HTTP_NOT_FOUND`, `ngx.HTTP_MOVED_TEMPORARILY` 或其它 [HTTP status constants](#http-status-constants)。

要返回一个自定义内容的错误页，使用类似下面的代码：

```lua

 ngx.status = ngx.HTTP_GONE
 ngx.say("This is our own content")
 -- 退出整个请求而不是当前处理阶段
 ngx.exit(ngx.HTTP_OK)
```

实际效果:

```bash

 $ curl -i http://localhost/test
 HTTP/1.1 410 Gone
 Server: nginx/1.0.6
 Date: Thu, 15 Sep 2011 00:51:48 GMT
 Content-Type: text/plain
 Transfer-Encoding: chunked
 Connection: keep-alive

 This is our own content
```

可以直接使用数字作为参数，例如：

```lua

 ngx.exit(501)
```

请注意，虽然此方法接受所有 [HTTP status constants](#http-status-constants) 作为输入，但在 [core constants](#core-constants)  中仅支持 `NGX_OK` 和 `NGX_ERROR` 作为输入。

同时请注意，因为调用此方法会中断当前请求处理，所以建议代码风格是与 `return` 语句一起调用此方法，即，使用 `return ngx.exit(...)` 来强调请求处理过程已经中断。

当使用在 [header_filter_by_lua](#header_filter_by_lua) 环境中时，`ngx.exit()` 是一个异步操作，会立即返回。这个行为在未来版本中可能会改变，所以建议用户一直使用上述与 `returen` 同时使用的代码风格。

[Back to TOC](#nginx-api-for-lua)

ngx.eof
-------
**语法:** *ok, err = ngx.eof()*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

明确指定响应输出流的末尾。在 HTTP 1.1 分块编码输出模式下，它会触发 Nginx 内核发送“最后一块”。

当禁用下游连接的 HTTP 1.1 保持连接功能后，用户程序可以通过调用此方法，使下游 HTTP 客户端主动关闭连接。这招可以用来执行后台任务，而无需 HTTP 客户端等待连接关闭，例如：

```nginx

 location = /async {
     keepalive_timeout 0;
     content_by_lua '
         ngx.say("got the task!")
         ngx.eof()  -- 下游 HTTP 客户端将在这里断开连接
         -- 在这里访问 MySQL, PostgreSQL, Redis, Memcached 等 ...
     ';
 }
```

但是，如果用户程序创建子请求通过 Nginx 上游模块访问其他 location 时，需要配置上游模块忽略客户端连接中断 (如果不是默认)。例如，默认时，基本模块 [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) 在客户端关闭连接后，立刻中断主请求和子请求，所以在 [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) 配置的 location 块中打开 [proxy_ignore_client_abort](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ignore_client_abort) 开关非常重要：

```nginx

 proxy_ignore_client_abort on;
```

一个执行后台任务的方法是使用 [ngx.timer.at](#ngxtimerat) API。

自版本 `v0.8.3`开始，此函数成功时返回 `1`，失败时返回 `nil` 和错误信息串。

[Back to TOC](#nginx-api-for-lua)

ngx.sleep
---------
**语法:** *ngx.sleep(seconds)*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;*

无阻塞地休眠特定秒。时间可以精确到 0.001 秒 (毫秒)。

在后台，此方法使用 Nginx 的定时器。

自版本 `0.7.20` 开始，`0` 也可以作为时间参数被指定。

这个方法最早在版本 `0.5.0rc30` 中出现。

[Back to TOC](#nginx-api-for-lua)

ngx.escape_uri
--------------
**语法:** *newstr = ngx.escape_uri(str)*

**环境:** *init_by_lua&#42;, init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

对 `str` 进行 URI 编码。

[Back to TOC](#nginx-api-for-lua)

ngx.unescape_uri
----------------
**语法:** *newstr = ngx.unescape_uri(str)*

**环境:** *init_by_lua&#42;, init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

将转义过的 URI 内容 `str` 解码。

例如,

```lua

 ngx.say(ngx.unescape_uri("b%20r56+7"))
```

输出


    b r56 7


[Back to TOC](#nginx-api-for-lua)

ngx.encode_args
---------------
**语法:** *str = ngx.encode_args(table)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

根据 URI 编码规则，将 Lua 表编码成一个查询参数字符串。

例如，

```lua

 ngx.encode_args({foo = 3, ["b r"] = "hello world"})
```

生成


    foo=3&b%20r=hello%20world


Lua 表的 key 必须是 Lua 字符串。

支持多值参数。可以使用 Lua 表存储参数值，例如：

```lua

 ngx.encode_args({baz = {32, "hello"}})
```

输出


    baz=32&baz=hello


如果 value 表是空的，效果等同于 `nil` 值。

支持布尔值参数，例如，

```lua

 ngx.encode_args({a = true, b = 1})
```

输出


    a&b=1


如果参数值是 `false`，效果等同于 `nil` 值。

这个方法最早出现在版本 `v0.3.1rc27` 中。

[Back to TOC](#nginx-api-for-lua)

ngx.decode_args
---------------
**语法:** *table = ngx.decode_args(str, max_args?)*

**环境:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

将 URI 编码的查询字符串解码为 Lua 表。本函数是 [ngx.encode_args](#ngxencode_args) 的逆函数。

可选的参数 `max_args` 可以用来指定从 `str` 中最多解析的参数个数。默认时，最多解析 100 个请求参数 (包括同名的)。为避免潜在的拒绝服务式攻击 (denial of services, DOS)，超过 `max_args` 数量上限的 URI 参数被丢弃，

这个参数可以被设成 0 以去掉解析参数数量上限：

```lua

 local args = ngx.decode_args(str, 0)
```

强烈不推荐移除 `max_args` 限制。

这个方法最早出现在版本 `v0.5.0rc29` 中。

[Back to TOC](#nginx-api-for-lua)

ngx.encode_base64
-----------------
**syntax:** *newstr = ngx.encode_base64(str, no_padding?)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Encodes `str` to a base64 digest.

Since the `0.9.16` release, an optional boolean-typed `no_padding` argument can be specified to control whether the base64 padding should be appended to the resulting digest (default to `false`, i.e., with padding enabled). This enables streaming base64 digest calculation by (data chunks) though it would be the caller's responsibility to append an appropriate padding at the end of data stream.

[Back to TOC](#nginx-api-for-lua)

ngx.decode_base64
-----------------
**syntax:** *newstr = ngx.decode_base64(str)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Decodes the `str` argument as a base64 digest to the raw form. Returns `nil` if `str` is not well formed.

[Back to TOC](#nginx-api-for-lua)

ngx.crc32_short
---------------
**syntax:** *intval = ngx.crc32_short(str)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Calculates the CRC-32 (Cyclic Redundancy Code) digest for the `str` argument.

This method performs better on relatively short `str` inputs (i.e., less than 30 ~ 60 bytes), as compared to [ngx.crc32_long](#ngxcrc32_long). The result is exactly the same as [ngx.crc32_long](#ngxcrc32_long).

Behind the scene, it is just a thin wrapper around the `ngx_crc32_short` function defined in the Nginx core.

This API was first introduced in the `v0.3.1rc8` release.

[Back to TOC](#nginx-api-for-lua)

ngx.crc32_long
--------------
**syntax:** *intval = ngx.crc32_long(str)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Calculates the CRC-32 (Cyclic Redundancy Code) digest for the `str` argument.

This method performs better on relatively long `str` inputs (i.e., longer than 30 ~ 60 bytes), as compared to [ngx.crc32_short](#ngxcrc32_short).  The result is exactly the same as [ngx.crc32_short](#ngxcrc32_short).

Behind the scene, it is just a thin wrapper around the `ngx_crc32_long` function defined in the Nginx core.

This API was first introduced in the `v0.3.1rc8` release.

[Back to TOC](#nginx-api-for-lua)

ngx.hmac_sha1
-------------
**syntax:** *digest = ngx.hmac_sha1(secret_key, str)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Computes the [HMAC-SHA1](http://en.wikipedia.org/wiki/HMAC) digest of the argument `str` and turns the result using the secret key `<secret_key>`.

The raw binary form of the `HMAC-SHA1` digest will be generated, use [ngx.encode_base64](#ngxencode_base64), for example, to encode the result to a textual representation if desired.

For example,

```lua

 local key = "thisisverysecretstuff"
 local src = "some string we want to sign"
 local digest = ngx.hmac_sha1(key, src)
 ngx.say(ngx.encode_base64(digest))
```

yields the output


    R/pvxzHC4NLtj7S+kXFg/NePTmk=


This API requires the OpenSSL library enabled in the Nginx build (usually by passing the `--with-http_ssl_module` option to the `./configure` script).

This function was first introduced in the `v0.3.1rc29` release.

[Back to TOC](#nginx-api-for-lua)

ngx.md5
-------
**syntax:** *digest = ngx.md5(str)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns the hexadecimal representation of the MD5 digest of the `str` argument.

For example,

```nginx

 location = /md5 {
     content_by_lua 'ngx.say(ngx.md5("hello"))';
 }
```

yields the output


    5d41402abc4b2a76b9719d911017c592


See [ngx.md5_bin](#ngxmd5_bin) if the raw binary MD5 digest is required.

[Back to TOC](#nginx-api-for-lua)

ngx.md5_bin
-----------
**syntax:** *digest = ngx.md5_bin(str)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns the binary form of the MD5 digest of the `str` argument.

See [ngx.md5](#ngxmd5) if the hexadecimal form of the MD5 digest is required.

[Back to TOC](#nginx-api-for-lua)

ngx.sha1_bin
------------
**syntax:** *digest = ngx.sha1_bin(str)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns the binary form of the SHA-1 digest of the `str` argument.

This function requires SHA-1 support in the Nginx build. (This usually just means OpenSSL should be installed while building Nginx).

This function was first introduced in the `v0.5.0rc6`.

[Back to TOC](#nginx-api-for-lua)

ngx.quote_sql_str
-----------------
**syntax:** *quoted_value = ngx.quote_sql_str(raw_value)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns a quoted SQL string literal according to the MySQL quoting rules.

[Back to TOC](#nginx-api-for-lua)

ngx.today
---------
**syntax:** *str = ngx.today()*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns current date (in the format `yyyy-mm-dd`) from the nginx cached time (no syscall involved unlike Lua's date library).

This is the local time.

[Back to TOC](#nginx-api-for-lua)

ngx.time
--------
**syntax:** *secs = ngx.time()*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns the elapsed seconds from the epoch for the current time stamp from the nginx cached time (no syscall involved unlike Lua's date library).

Updates of the Nginx time cache an be forced by calling [ngx.update_time](#ngxupdate_time) first.

[Back to TOC](#nginx-api-for-lua)

ngx.now
-------
**syntax:** *secs = ngx.now()*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns a floating-point number for the elapsed time in seconds (including milliseconds as the decimal part) from the epoch for the current time stamp from the nginx cached time (no syscall involved unlike Lua's date library).

You can forcibly update the Nginx time cache by calling [ngx.update_time](#ngxupdate_time) first.

This API was first introduced in `v0.3.1rc32`.

[Back to TOC](#nginx-api-for-lua)

ngx.update_time
---------------
**syntax:** *ngx.update_time()*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Forcibly updates the Nginx current time cache. This call involves a syscall and thus has some overhead, so do not abuse it.

This API was first introduced in `v0.3.1rc32`.

[Back to TOC](#nginx-api-for-lua)

ngx.localtime
-------------
**syntax:** *str = ngx.localtime()*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns the current time stamp (in the format `yyyy-mm-dd hh:mm:ss`) of the nginx cached time (no syscall involved unlike Lua's [os.date](http://www.lua.org/manual/5.1/manual.html#pdf-os.date) function).

This is the local time.

[Back to TOC](#nginx-api-for-lua)

ngx.utctime
-----------
**syntax:** *str = ngx.utctime()*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns the current time stamp (in the format `yyyy-mm-dd hh:mm:ss`) of the nginx cached time (no syscall involved unlike Lua's [os.date](http://www.lua.org/manual/5.1/manual.html#pdf-os.date) function).

This is the UTC time.

[Back to TOC](#nginx-api-for-lua)

ngx.cookie_time
---------------
**syntax:** *str = ngx.cookie_time(sec)*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns a formatted string can be used as the cookie expiration time. The parameter `sec` is the time stamp in seconds (like those returned from [ngx.time](#ngxtime)).

```nginx

 ngx.say(ngx.cookie_time(1290079655))
     -- yields "Thu, 18-Nov-10 11:27:35 GMT"
```

[Back to TOC](#nginx-api-for-lua)

ngx.http_time
-------------
**syntax:** *str = ngx.http_time(sec)*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns a formated string can be used as the http header time (for example, being used in `Last-Modified` header). The parameter `sec` is the time stamp in seconds (like those returned from [ngx.time](#ngxtime)).

```nginx

 ngx.say(ngx.http_time(1290079655))
     -- yields "Thu, 18 Nov 2010 11:27:35 GMT"
```

[Back to TOC](#nginx-api-for-lua)

ngx.parse_http_time
-------------------
**syntax:** *sec = ngx.parse_http_time(str)*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Parse the http time string (as returned by [ngx.http_time](#ngxhttp_time)) into seconds. Returns the seconds or `nil` if the input string is in bad forms.

```nginx

 local time = ngx.parse_http_time("Thu, 18 Nov 2010 11:27:35 GMT")
 if time == nil then
     ...
 end
```

[Back to TOC](#nginx-api-for-lua)

ngx.is_subrequest
-----------------
**syntax:** *value = ngx.is_subrequest*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;*

Returns `true` if the current request is an nginx subrequest, or `false` otherwise.

[Back to TOC](#nginx-api-for-lua)

ngx.re.match
------------
**syntax:** *captures, err = ngx.re.match(subject, regex, options?, ctx?, res_table?)*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Matches the `subject` string using the Perl compatible regular expression `regex` with the optional `options`.

Only the first occurrence of the match is returned, or `nil` if no match is found. In case of errors, like seeing a bad regular expression or exceeding the PCRE stack limit, `nil` and a string describing the error will be returned.

When a match is found, a Lua table `captures` is returned, where `captures[0]` holds the whole substring being matched, and `captures[1]` holds the first parenthesized sub-pattern's capturing, `captures[2]` the second, and so on.

```lua

 local m, err = ngx.re.match("hello, 1234", "[0-9]+")
 if m then
     -- m[0] == "1234"

 else
     if err then
         ngx.log(ngx.ERR, "error: ", err)
         return
     end

     ngx.say("match not found")
 end
```

```lua

 local m, err = ngx.re.match("hello, 1234", "([0-9])[0-9]+")
 -- m[0] == "1234"
 -- m[1] == "1"
```

Named captures are also supported since the `v0.7.14` release
and are returned in the same Lua table as key-value pairs as the numbered captures.

```lua

 local m, err = ngx.re.match("hello, 1234", "([0-9])(?<remaining>[0-9]+)")
 -- m[0] == "1234"
 -- m[1] == "1"
 -- m[2] == "234"
 -- m["remaining"] == "234"
```

Unmatched subpatterns will have `nil` values in their `captures` table fields.

```lua

 local m, err = ngx.re.match("hello, world", "(world)|(hello)|(?<named>howdy)")
 -- m[0] == "hello"
 -- m[1] == nil
 -- m[2] == "hello"
 -- m[3] == nil
 -- m["named"] == nil
```

Specify `options` to control how the match operation will be performed. The following option characters are supported:


    a             anchored mode (only match from the beginning)

    d             enable the DFA mode (or the longest token match semantics).
                  this requires PCRE 6.0+ or else a Lua exception will be thrown.
                  first introduced in ngx_lua v0.3.1rc30.

    D             enable duplicate named pattern support. This allows named
                  subpattern names to be repeated, returning the captures in
                  an array-like Lua table. for example,
                    local m = ngx.re.match("hello, world",
                                           "(?<named>\w+), (?<named>\w+)",
                                           "D")
                    -- m["named"] == {"hello", "world"}
                  this option was first introduced in the v0.7.14 release.
                  this option requires at least PCRE 8.12.

    i             case insensitive mode (similar to Perl's /i modifier)

    j             enable PCRE JIT compilation, this requires PCRE 8.21+ which
                  must be built with the --enable-jit option. for optimum performance,
                  this option should always be used together with the 'o' option.
                  first introduced in ngx_lua v0.3.1rc30.

    J             enable the PCRE Javascript compatible mode. this option was
                  first introduced in the v0.7.14 release. this option requires
                  at least PCRE 8.12.

    m             multi-line mode (similar to Perl's /m modifier)

    o             compile-once mode (similar to Perl's /o modifier),
                  to enable the worker-process-level compiled-regex cache

    s             single-line mode (similar to Perl's /s modifier)

    u             UTF-8 mode. this requires PCRE to be built with
                  the --enable-utf8 option or else a Lua exception will be thrown.

    U             similar to "u" but disables PCRE's UTF-8 validity check on
                  the subject string. first introduced in ngx_lua v0.8.1.

    x             extended mode (similar to Perl's /x modifier)


These options can be combined:

```nginx

 local m, err = ngx.re.match("hello, world", "HEL LO", "ix")
 -- m[0] == "hello"
```

```nginx

 local m, err = ngx.re.match("hello, 美好生活", "HELLO, (.{2})", "iu")
 -- m[0] == "hello, 美好"
 -- m[1] == "美好"
```

The `o` option is useful for performance tuning, because the regex pattern in question will only be compiled once, cached in the worker-process level, and shared among all requests in the current Nginx worker process. The upper limit of the regex cache can be tuned via the [lua_regex_cache_max_entries](#lua_regex_cache_max_entries) directive.

The optional fourth argument, `ctx`, can be a Lua table holding an optional `pos` field. When the `pos` field in the `ctx` table argument is specified, `ngx.re.match` will start matching from that offset (starting from 1). Regardless of the presence of the `pos` field in the `ctx` table, `ngx.re.match` will always set this `pos` field to the position *after* the substring matched by the whole pattern in case of a successful match. When match fails, the `ctx` table will be left intact.

```lua

 local ctx = {}
 local m, err = ngx.re.match("1234, hello", "[0-9]+", "", ctx)
      -- m[0] = "1234"
      -- ctx.pos == 5
```

```lua

 local ctx = { pos = 2 }
 local m, err = ngx.re.match("1234, hello", "[0-9]+", "", ctx)
      -- m[0] = "34"
      -- ctx.pos == 5
```

The `ctx` table argument combined with the `a` regex modifier can be used to construct a lexer atop `ngx.re.match`.

Note that, the `options` argument is not optional when the `ctx` argument is specified and that the empty Lua string (`""`) must be used as placeholder for `options` if no meaningful regex options are required.

This method requires the PCRE library enabled in Nginx.  ([Known Issue With Special Escaping Sequences](#special-escaping-sequences)).

To confirm that PCRE JIT is enabled, activate the Nginx debug log by adding the `--with-debug` option to Nginx or ngx_openresty's `./configure` script. Then, enable the "debug" error log level in `error_log` directive. The following message will be generated if PCRE JIT is enabled:


    pcre JIT compiling result: 1


Starting from the `0.9.4` release, this function also accepts a 5th argument, `res_table`, for letting the caller supply the Lua table used to hold all the capturing results. Starting from `0.9.6`, it is the caller's responsibility to ensure this table is empty. This is very useful for recycling Lua tables and saving GC and table allocation overhead.

This feature was introduced in the `v0.2.1rc11` release.

[Back to TOC](#nginx-api-for-lua)

ngx.re.find
-----------
**syntax:** *from, to, err = ngx.re.find(subject, regex, options?, ctx?, nth?)*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Similar to [ngx.re.match](#ngxrematch) but only returns the begining index (`from`) and end index (`to`) of the matched substring. The returned indexes are 1-based and can be fed directly into the [string.sub](http://www.lua.org/manual/5.1/manual.html#pdf-string.sub) API function to obtain the matched substring.

In case of errors (like bad regexes or any PCRE runtime errors), this API function returns two `nil` values followed by a string describing the error.

If no match is found, this function just returns a `nil` value.

Below is an example:

```lua

 local s = "hello, 1234"
 local from, to, err = ngx.re.find(s, "([0-9]+)", "jo")
 if from then
     ngx.say("from: ", from)
     ngx.say("to: ", to)
     ngx.say("matched: ", string.sub(s, from, to))
 else
     if err then
         ngx.say("error: ", err)
         return
     end
     ngx.say("not matched!")
 end
```

This example produces the output

    from: 8
    to: 11
    matched: 1234

Because this API function does not create new Lua strings nor new Lua tables, it is much faster than [ngx.re.match](#ngxrematch). It should be used wherever possible.

Since the `0.9.3` release, an optional 5th argument, `nth`, is supported to specify which (submatch) capture's indexes to return. When `nth` is 0 (which is the default), the indexes for the whole matched substring is returned; when `nth` is 1, then the 1st submatch capture's indexes are returned; when `nth` is 2, then the 2nd submatch capture is returned, and so on. When the specified submatch does not have a match, then two `nil` values will be returned. Below is an example for this:

```lua

 local str = "hello, 1234"
 local from, to = ngx.re.find(str, "([0-9])([0-9]+)", "jo", nil, 2)
 if from then
     ngx.say("matched 2nd submatch: ", string.sub(str, from, to))  -- yields "234"
 end
```

This API function was first introduced in the `v0.9.2` release.

[Back to TOC](#nginx-api-for-lua)

ngx.re.gmatch
-------------
**syntax:** *iterator, err = ngx.re.gmatch(subject, regex, options?)*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Similar to [ngx.re.match](#ngxrematch), but returns a Lua iterator instead, so as to let the user programmer iterate all the matches over the `<subject>` string argument with the PCRE `regex`.

In case of errors, like seeing an ill-formed regular expression, `nil` and a string describing the error will be returned.

Here is a small example to demonstrate its basic usage:

```lua

 local iterator, err = ngx.re.gmatch("hello, world!", "([a-z]+)", "i")
 if not iterator then
     ngx.log(ngx.ERR, "error: ", err)
     return
 end

 local m
 m, err = iterator()    -- m[0] == m[1] == "hello"
 if err then
     ngx.log(ngx.ERR, "error: ", err)
     return
 end

 m, err = iterator()    -- m[0] == m[1] == "world"
 if err then
     ngx.log(ngx.ERR, "error: ", err)
     return
 end

 m, err = iterator()    -- m == nil
 if err then
     ngx.log(ngx.ERR, "error: ", err)
     return
 end
```

More often we just put it into a Lua loop:

```lua

 local it, err = ngx.re.gmatch("hello, world!", "([a-z]+)", "i")
 if not it then
     ngx.log(ngx.ERR, "error: ", err)
     return
 end

 while true do
     local m, err = it()
     if err then
         ngx.log(ngx.ERR, "error: ", err)
         return
     end

     if not m then
         -- no match found (any more)
         break
     end

     -- found a match
     ngx.say(m[0])
     ngx.say(m[1])
 end
```

The optional `options` argument takes exactly the same semantics as the [ngx.re.match](#ngxrematch) method.

The current implementation requires that the iterator returned should only be used in a single request. That is, one should *not* assign it to a variable belonging to persistent namespace like a Lua package.

This method requires the PCRE library enabled in Nginx.  ([Known Issue With Special Escaping Sequences](#special-escaping-sequences)).

This feature was first introduced in the `v0.2.1rc12` release.

[Back to TOC](#nginx-api-for-lua)

ngx.re.sub
----------
**syntax:** *newstr, n, err = ngx.re.sub(subject, regex, replace, options?)*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Substitutes the first match of the Perl compatible regular expression `regex` on the `subject` argument string with the string or function argument `replace`. The optional `options` argument has exactly the same meaning as in [ngx.re.match](#ngxrematch).

This method returns the resulting new string as well as the number of successful substitutions. In case of failures, like syntax errors in the regular expressions or the `<replace>` string argument, it will return `nil` and a string describing the error.

When the `replace` is a string, then it is treated as a special template for string replacement. For example,

```lua

 local newstr, n, err = ngx.re.sub("hello, 1234", "([0-9])[0-9]", "[$0][$1]")
 if newstr then
     -- newstr == "hello, [12][1]34"
     -- n == 1
 else
     ngx.log(ngx.ERR, "error: ", err)
     return
 end
```

where `$0` referring to the whole substring matched by the pattern and `$1` referring to the first parenthesized capturing substring.

Curly braces can also be used to disambiguate variable names from the background string literals: 

```lua

 local newstr, n, err = ngx.re.sub("hello, 1234", "[0-9]", "${0}00")
     -- newstr == "hello, 100234"
     -- n == 1
```

Literal dollar sign characters (`$`) in the `replace` string argument can be escaped by another dollar sign, for instance,

```lua

 local newstr, n, err = ngx.re.sub("hello, 1234", "[0-9]", "$$")
     -- newstr == "hello, $234"
     -- n == 1
```

Do not use backlashes to escape dollar signs; it will not work as expected.

When the `replace` argument is of type "function", then it will be invoked with the "match table" as the argument to generate the replace string literal for substitution. The "match table" fed into the `replace` function is exactly the same as the return value of [ngx.re.match](#ngxrematch). Here is an example:

```lua

 local func = function (m)
     return "[" .. m[0] .. "][" .. m[1] .. "]"
 end
 local newstr, n, err = ngx.re.sub("hello, 1234", "( [0-9] ) [0-9]", func, "x")
     -- newstr == "hello, [12][1]34"
     -- n == 1
```

The dollar sign characters in the return value of the `replace` function argument are not special at all.

This method requires the PCRE library enabled in Nginx.  ([Known Issue With Special Escaping Sequences](#special-escaping-sequences)).

This feature was first introduced in the `v0.2.1rc13` release.

[Back to TOC](#nginx-api-for-lua)

ngx.re.gsub
-----------
**syntax:** *newstr, n, err = ngx.re.gsub(subject, regex, replace, options?)*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Just like [ngx.re.sub](#ngxresub), but does global substitution.

Here is some examples:

```lua

 local newstr, n, err = ngx.re.gsub("hello, world", "([a-z])[a-z]+", "[$0,$1]", "i")
 if newstr then
     -- newstr == "[hello,h], [world,w]"
     -- n == 2
 else
     ngx.log(ngx.ERR, "error: ", err)
     return
 end
```

```lua

 local func = function (m)
     return "[" .. m[0] .. "," .. m[1] .. "]"
 end
 local newstr, n, err = ngx.re.gsub("hello, world", "([a-z])[a-z]+", func, "i")
     -- newstr == "[hello,h], [world,w]"
     -- n == 2
```

This method requires the PCRE library enabled in Nginx.  ([Known Issue With Special Escaping Sequences](#special-escaping-sequences)).

This feature was first introduced in the `v0.2.1rc15` release.

[Back to TOC](#nginx-api-for-lua)

ngx.shared.DICT
---------------
**syntax:** *dict = ngx.shared.DICT*

**syntax:** *dict = ngx.shared\[name_var\]*

**context:** *init_by_lua&#42;, init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Fetching the shm-based Lua dictionary object for the shared memory zone named `DICT` defined by the [lua_shared_dict](#lua_shared_dict) directive.

Shared memory zones are always shared by all the nginx worker processes in the current nginx server instance.

The resulting object `dict` has the following methods:

* [get](#ngxshareddictget)
* [get_stale](#ngxshareddictget_stale)
* [set](#ngxshareddictset)
* [safe_set](#ngxshareddictsafe_set)
* [add](#ngxshareddictadd)
* [safe_add](#ngxshareddictsafe_add)
* [replace](#ngxshareddictreplace)
* [delete](#ngxshareddictdelete)
* [incr](#ngxshareddictincr)
* [flush_all](#ngxshareddictflush_all)
* [flush_expired](#ngxshareddictflush_expired)
* [get_keys](#ngxshareddictget_keys)

Here is an example:

```nginx

 http {
     lua_shared_dict dogs 10m;
     server {
         location /set {
             content_by_lua '
                 local dogs = ngx.shared.dogs
                 dogs:set("Jim", 8)
                 ngx.say("STORED")
             ';
         }
         location /get {
             content_by_lua '
                 local dogs = ngx.shared.dogs
                 ngx.say(dogs:get("Jim"))
             ';
         }
     }
 }
```

Let us test it:

```bash

 $ curl localhost/set
 STORED

 $ curl localhost/get
 8

 $ curl localhost/get
 8
```

The number `8` will be consistently output when accessing `/get` regardless of how many Nginx workers there are because the `dogs` dictionary resides in the shared memory and visible to *all* of the worker processes.

The shared dictionary will retain its contents through a server config reload (either by sending the `HUP` signal to the Nginx process or by using the `-s reload` command-line option).

The contents in the dictionary storage will be lost, however, when the Nginx server quits.

This feature was first introduced in the `v0.3.1rc22` release.

[Back to TOC](#nginx-api-for-lua)

ngx.shared.DICT.get
-------------------
**syntax:** *value, flags = ngx.shared.DICT:get(key)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Retrieving the value in the dictionary [ngx.shared.DICT](#ngxshareddict) for the key `key`. If the key does not exist or has been expired, then `nil` will be returned.

In case of errors, `nil` and a string describing the error will be returned.

The value returned will have the original data type when they were inserted into the dictionary, for example, Lua booleans, numbers, or strings.

The first argument to this method must be the dictionary object itself, for example,

```lua

 local cats = ngx.shared.cats
 local value, flags = cats.get(cats, "Marry")
```

or use Lua's syntactic sugar for method calls:

```lua

 local cats = ngx.shared.cats
 local value, flags = cats:get("Marry")
```

These two forms are fundamentally equivalent.

If the user flags is `0` (the default), then no flags value will be returned.

This feature was first introduced in the `v0.3.1rc22` release.

See also [ngx.shared.DICT](#ngxshareddict).

[Back to TOC](#nginx-api-for-lua)

ngx.shared.DICT.get_stale
-------------------------
**syntax:** *value, flags, stale = ngx.shared.DICT:get_stale(key)*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Similar to the [get](#ngxshareddictget) method but returns the value even if the key has already expired.

Returns a 3rd value, `stale`, indicating whether the key has expired or not.

Note that the value of an expired key is not guaranteed to be available so one should never rely on the availability of expired items.

This method was first introduced in the `0.8.6` release.

See also [ngx.shared.DICT](#ngxshareddict).

[Back to TOC](#nginx-api-for-lua)

ngx.shared.DICT.set
-------------------
**syntax:** *success, err, forcible = ngx.shared.DICT:set(key, value, exptime?, flags?)*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Unconditionally sets a key-value pair into the shm-based dictionary [ngx.shared.DICT](#ngxshareddict). Returns three values:

* `success`: boolean value to indicate whether the key-value pair is stored or not.
* `err`: textual error message, can be `"no memory"`.
* `forcible`: a boolean value to indicate whether other valid items have been removed forcibly when out of storage in the shared memory zone.

The `value` argument inserted can be Lua booleans, numbers, strings, or `nil`. Their value type will also be stored into the dictionary and the same data type can be retrieved later via the [get](#ngxshareddictget) method.

The optional `exptime` argument specifies expiration time (in seconds) for the inserted key-value pair. The time resolution is `0.001` seconds. If the `exptime` takes the value `0` (which is the default), then the item will never expire.

The optional `flags` argument specifies a user flags value associated with the entry to be stored. It can also be retrieved later with the value. The user flags is stored as an unsigned 32-bit integer internally. Defaults to `0`. The user flags argument was first introduced in the `v0.5.0rc2` release.

When it fails to allocate memory for the current key-value item, then `set` will try removing existing items in the storage according to the Least-Recently Used (LRU) algorithm. Note that, LRU takes priority over expiration time here. If up to tens of existing items have been removed and the storage left is still insufficient (either due to the total capacity limit specified by [lua_shared_dict](#lua_shared_dict) or memory segmentation), then the `err` return value will be `no memory` and `success` will be `false`.

If this method succeeds in storing the current item by forcibly removing other not-yet-expired items in the dictionary via LRU, the `forcible` return value will be `true`. If it stores the item without forcibly removing other valid items, then the return value `forcible` will be `false`.

The first argument to this method must be the dictionary object itself, for example,

```lua

 local cats = ngx.shared.cats
 local succ, err, forcible = cats.set(cats, "Marry", "it is a nice cat!")
```

or use Lua's syntactic sugar for method calls:

```lua

 local cats = ngx.shared.cats
 local succ, err, forcible = cats:set("Marry", "it is a nice cat!")
```

These two forms are fundamentally equivalent.

This feature was first introduced in the `v0.3.1rc22` release.

Please note that while internally the key-value pair is set atomically, the atomicity does not go across the method call boundary.

See also [ngx.shared.DICT](#ngxshareddict).

[Back to TOC](#nginx-api-for-lua)

ngx.shared.DICT.safe_set
------------------------
**syntax:** *ok, err = ngx.shared.DICT:safe_set(key, value, exptime?, flags?)*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Similar to the [set](#ngxshareddictset) method, but never overrides the (least recently used) unexpired items in the store when running out of storage in the shared memory zone. In this case, it will immediately return `nil` and the string "no memory".

This feature was first introduced in the `v0.7.18` release.

See also [ngx.shared.DICT](#ngxshareddict).

[Back to TOC](#nginx-api-for-lua)

ngx.shared.DICT.add
-------------------
**syntax:** *success, err, forcible = ngx.shared.DICT:add(key, value, exptime?, flags?)*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Just like the [set](#ngxshareddictset) method, but only stores the key-value pair into the dictionary [ngx.shared.DICT](#ngxshareddict) if the key does *not* exist.

If the `key` argument already exists in the dictionary (and not expired for sure), the `success` return value will be `false` and the `err` return value will be `"exists"`.

This feature was first introduced in the `v0.3.1rc22` release.

See also [ngx.shared.DICT](#ngxshareddict).

[Back to TOC](#nginx-api-for-lua)

ngx.shared.DICT.safe_add
------------------------
**syntax:** *ok, err = ngx.shared.DICT:safe_add(key, value, exptime?, flags?)*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Similar to the [add](#ngxshareddictadd) method, but never overrides the (least recently used) unexpired items in the store when running out of storage in the shared memory zone. In this case, it will immediately return `nil` and the string "no memory".

This feature was first introduced in the `v0.7.18` release.

See also [ngx.shared.DICT](#ngxshareddict).

[Back to TOC](#nginx-api-for-lua)

ngx.shared.DICT.replace
-----------------------
**syntax:** *success, err, forcible = ngx.shared.DICT:replace(key, value, exptime?, flags?)*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Just like the [set](#ngxshareddictset) method, but only stores the key-value pair into the dictionary [ngx.shared.DICT](#ngxshareddict) if the key *does* exist.

If the `key` argument does *not* exist in the dictionary (or expired already), the `success` return value will be `false` and the `err` return value will be `"not found"`.

This feature was first introduced in the `v0.3.1rc22` release.

See also [ngx.shared.DICT](#ngxshareddict).

[Back to TOC](#nginx-api-for-lua)

ngx.shared.DICT.delete
----------------------
**syntax:** *ngx.shared.DICT:delete(key)*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Unconditionally removes the key-value pair from the shm-based dictionary [ngx.shared.DICT](#ngxshareddict).

It is equivalent to `ngx.shared.DICT:set(key, nil)`.

This feature was first introduced in the `v0.3.1rc22` release.

See also [ngx.shared.DICT](#ngxshareddict).

[Back to TOC](#nginx-api-for-lua)

ngx.shared.DICT.incr
--------------------
**syntax:** *newval, err = ngx.shared.DICT:incr(key, value)*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Increments the (numerical) value for `key` in the shm-based dictionary [ngx.shared.DICT](#ngxshareddict) by the step value `value`. Returns the new resulting number if the operation is successfully completed or `nil` and an error message otherwise.

The key must already exist in the dictionary, otherwise it will return `nil` and `"not found"`.

If the original value is not a valid Lua number in the dictionary, it will return `nil` and `"not a number"`.

The `value` argument can be any valid Lua numbers, like negative numbers or floating-point numbers.

This feature was first introduced in the `v0.3.1rc22` release.

See also [ngx.shared.DICT](#ngxshareddict).

[Back to TOC](#nginx-api-for-lua)

ngx.shared.DICT.flush_all
-------------------------
**syntax:** *ngx.shared.DICT:flush_all()*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Flushes out all the items in the dictionary. This method does not actuall free up all the memory blocks in the dictionary but just marks all the existing items as expired.

This feature was first introduced in the `v0.5.0rc17` release.

See also [ngx.shared.DICT.flush_expired](#ngxshareddictflush_expired) and [ngx.shared.DICT](#ngxshareddict).

[Back to TOC](#nginx-api-for-lua)

ngx.shared.DICT.flush_expired
-----------------------------
**syntax:** *flushed = ngx.shared.DICT:flush_expired(max_count?)*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Flushes out the expired items in the dictionary, up to the maximal number specified by the optional `max_count` argument. When the `max_count` argument is given `0` or not given at all, then it means unlimited. Returns the number of items that have actually been flushed.

Unlike the [flush_all](#ngxshareddictflush_all) method, this method actually free up the memory used by the expired items.

This feature was first introduced in the `v0.6.3` release.

See also [ngx.shared.DICT.flush_all](#ngxshareddictflush_all) and [ngx.shared.DICT](#ngxshareddict).

[Back to TOC](#nginx-api-for-lua)

ngx.shared.DICT.get_keys
------------------------
**syntax:** *keys = ngx.shared.DICT:get_keys(max_count?)*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Fetch a list of the keys from the dictionary, up to `<max_count>`.

By default, only the first 1024 keys (if any) are returned. When the `<max_count>` argument is given the value `0`, then all the keys will be returned even there is more than 1024 keys in the dictionary.

**WARNING** Be careful when calling this method on dictionaries with a really huge number of keys. This method may lock the dictionary for quite a while and block all the nginx worker processes that are trying to access the dictionary.

This feature was first introduced in the `v0.7.3` release.

[Back to TOC](#nginx-api-for-lua)

ngx.socket.udp
--------------
**syntax:** *udpsock = ngx.socket.udp()*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Creates and returns a UDP or datagram-oriented unix domain socket object (also known as one type of the "cosocket" objects). The following methods are supported on this object:

* [setpeername](#udpsocksetpeername)
* [send](#udpsocksend)
* [receive](#udpsockreceive)
* [close](#udpsockclose)
* [settimeout](#udpsocksettimeout)

It is intended to be compatible with the UDP API of the [LuaSocket](http://w3.impa.br/~diego/software/luasocket/udp.html) library but is 100% nonblocking out of the box.

This feature was first introduced in the `v0.5.7` release.

See also [ngx.socket.tcp](#ngxsockettcp).

[Back to TOC](#nginx-api-for-lua)

udpsock:setpeername
-------------------
**syntax:** *ok, err = udpsock:setpeername(host, port)*

**syntax:** *ok, err = udpsock:setpeername("unix:/path/to/unix-domain.socket")*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Attempts to connect a UDP socket object to a remote server or to a datagram unix domain socket file. Because the datagram protocol is actually connection-less, this method does not really establish a "connection", but only just set the name of the remote peer for subsequent read/write operations.

Both IP addresses and domain names can be specified as the `host` argument. In case of domain names, this method will use Nginx core's dynamic resolver to parse the domain name without blocking and it is required to configure the [resolver](http://nginx.org/en/docs/http/ngx_http_core_module.html#resolver) directive in the `nginx.conf` file like this:

```nginx

 resolver 8.8.8.8;  # use Google's public DNS nameserver
```

If the nameserver returns multiple IP addresses for the host name, this method will pick up one randomly.

In case of error, the method returns `nil` followed by a string describing the error. In case of success, the method returns `1`.

Here is an example for connecting to a UDP (memcached) server:

```nginx

 location /test {
     resolver 8.8.8.8;

     content_by_lua '
         local sock = ngx.socket.udp()
         local ok, err = sock:setpeername("my.memcached.server.domain", 11211)
         if not ok then
             ngx.say("failed to connect to memcached: ", err)
             return
         end
         ngx.say("successfully connected to memcached!")
         sock:close()
     ';
 }
```

Since the `v0.7.18` release, connecting to a datagram unix domain socket file is also possible on Linux:

```lua

 local sock = ngx.socket.udp()
 local ok, err = sock:setpeername("unix:/tmp/some-datagram-service.sock")
 if not ok then
     ngx.say("failed to connect to the datagram unix domain socket: ", err)
     return
 end
```

assuming the datagram service is listening on the unix domain socket file `/tmp/some-datagram-service.sock` and the client socket will use the "autobind" feature on Linux.

Calling this method on an already connected socket object will cause the original connection to be closed first.

This method was first introduced in the `v0.5.7` release.

[Back to TOC](#nginx-api-for-lua)

udpsock:send
------------
**syntax:** *ok, err = udpsock:send(data)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Sends data on the current UDP or datagram unix domain socket object.

In case of success, it returns `1`. Otherwise, it returns `nil` and a string describing the error.

The input argument `data` can either be a Lua string or a (nested) Lua table holding string fragments. In case of table arguments, this method will copy all the string elements piece by piece to the underlying Nginx socket send buffers, which is usually optimal than doing string concatenation operations on the Lua land.

This feature was first introduced in the `v0.5.7` release.

[Back to TOC](#nginx-api-for-lua)

udpsock:receive
---------------
**syntax:** *data, err = udpsock:receive(size?)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Receives data from the UDP or datagram unix domain socket object with an optional receive buffer size argument, `size`.

This method is a synchronous operation and is 100% nonblocking.

In case of success, it returns the data received; in case of error, it returns `nil` with a string describing the error.

If the `size` argument is specified, then this method will use this size as the receive buffer size. But when this size is greater than `8192`, then `8192` will be used instead.

If no argument is specified, then the maximal buffer size, `8192` is assumed.

Timeout for the reading operation is controlled by the [lua_socket_read_timeout](#lua_socket_read_timeout) config directive and the [settimeout](#udpsocksettimeout) method. And the latter takes priority. For example:

```lua

 sock:settimeout(1000)  -- one second timeout
 local data, err = sock:receive()
 if not data then
     ngx.say("failed to read a packet: ", data)
     return
 end
 ngx.say("successfully read a packet: ", data)
```

It is important here to call the [settimeout](#udpsocksettimeout) method *before* calling this method.

This feature was first introduced in the `v0.5.7` release.

[Back to TOC](#nginx-api-for-lua)

udpsock:close
-------------
**syntax:** *ok, err = udpsock:close()*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Closes the current UDP or datagram unix domain socket. It returns the `1` in case of success and returns `nil` with a string describing the error otherwise.

Socket objects that have not invoked this method (and associated connections) will be closed when the socket object is released by the Lua GC (Garbage Collector) or the current client HTTP request finishes processing.

This feature was first introduced in the `v0.5.7` release.

[Back to TOC](#nginx-api-for-lua)

udpsock:settimeout
------------------
**syntax:** *udpsock:settimeout(time)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Set the timeout value in milliseconds for subsequent socket operations (like [receive](#udpsockreceive)).

Settings done by this method takes priority over those config directives, like [lua_socket_read_timeout](#lua_socket_read_timeout).

This feature was first introduced in the `v0.5.7` release.

[Back to TOC](#nginx-api-for-lua)

ngx.socket.tcp
--------------
**syntax:** *tcpsock = ngx.socket.tcp()*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Creates and returns a TCP or stream-oriented unix domain socket object (also known as one type of the "cosocket" objects). The following methods are supported on this object:

* [connect](#tcpsockconnect)
* [sslhandshake](#tcpsocksslhandshake)
* [send](#tcpsocksend)
* [receive](#tcpsockreceive)
* [close](#tcpsockclose)
* [settimeout](#tcpsocksettimeout)
* [setoption](#tcpsocksetoption)
* [receiveuntil](#tcpsockreceiveuntil)
* [setkeepalive](#tcpsocksetkeepalive)
* [getreusedtimes](#tcpsockgetreusedtimes)

It is intended to be compatible with the TCP API of the [LuaSocket](http://w3.impa.br/~diego/software/luasocket/tcp.html) library but is 100% nonblocking out of the box. Also, we introduce some new APIs to provide more functionalities.

The cosocket object created by this API function has exactly the same lifetime as the Lua handler creating it. So never pass the cosocket object to any other Lua handler (including ngx.timer callback functions) and never share the cosocket object between different NGINX requests.

For every cosocket object's underlying connection, if you do not
explicitly close it (via [close](#tcpsockclose)) or put it back to the connection
pool (via [setkeepalive](#tcpsocksetkeepalive)), then it is automatically closed when one of
the following two events happens:

* the current request handler completes, or
* the Lua cosocket object value gets collected by the Lua GC.

Fatal errors in cosocket operations always automatically close the current
connection (note that, read timeout error is the only error that is
not fatal), and if you call [close](#tcpsockclose) on a closed connection, you will get
the "closed" error.

Starting from the `0.9.9` release, the cosocket object here is full-duplex, that is, a reader "light thread" and a writer "light thread" can operate on a single cosocket object simultaneously (both "light threads" must belong to the same Lua handler though, see reasons above). But you cannot have two "light threads" both reading (or writing or connecting) the same cosocket, otherwise you might get an error like "socket busy reading" when calling the methods of the cosocket object.

This feature was first introduced in the `v0.5.0rc1` release.

See also [ngx.socket.udp](#ngxsocketudp).

[Back to TOC](#nginx-api-for-lua)

tcpsock:connect
---------------
**syntax:** *ok, err = tcpsock:connect(host, port, options_table?)*

**syntax:** *ok, err = tcpsock:connect("unix:/path/to/unix-domain.socket", options_table?)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Attempts to connect a TCP socket object to a remote server or to a stream unix domain socket file without blocking.

Before actually resolving the host name and connecting to the remote backend, this method will always look up the connection pool for matched idle connections created by previous calls of this method (or the [ngx.socket.connect](#ngxsocketconnect) function).

Both IP addresses and domain names can be specified as the `host` argument. In case of domain names, this method will use Nginx core's dynamic resolver to parse the domain name without blocking and it is required to configure the [resolver](http://nginx.org/en/docs/http/ngx_http_core_module.html#resolver) directive in the `nginx.conf` file like this:

```nginx

 resolver 8.8.8.8;  # use Google's public DNS nameserver
```

If the nameserver returns multiple IP addresses for the host name, this method will pick up one randomly.

In case of error, the method returns `nil` followed by a string describing the error. In case of success, the method returns `1`.

Here is an example for connecting to a TCP server:

```nginx

 location /test {
     resolver 8.8.8.8;

     content_by_lua '
         local sock = ngx.socket.tcp()
         local ok, err = sock:connect("www.google.com", 80)
         if not ok then
             ngx.say("failed to connect to google: ", err)
             return
         end
         ngx.say("successfully connected to google!")
         sock:close()
     ';
 }
```

Connecting to a Unix Domain Socket file is also possible:

```lua

 local sock = ngx.socket.tcp()
 local ok, err = sock:connect("unix:/tmp/memcached.sock")
 if not ok then
     ngx.say("failed to connect to the memcached unix domain socket: ", err)
     return
 end
```

assuming memcached (or something else) is listening on the unix domain socket file `/tmp/memcached.sock`.

Timeout for the connecting operation is controlled by the [lua_socket_connect_timeout](#lua_socket_connect_timeout) config directive and the [settimeout](#tcpsocksettimeout) method. And the latter takes priority. For example:

```lua

 local sock = ngx.socket.tcp()
 sock:settimeout(1000)  -- one second timeout
 local ok, err = sock:connect(host, port)
```

It is important here to call the [settimeout](#tcpsocksettimeout) method *before* calling this method.

Calling this method on an already connected socket object will cause the original connection to be closed first.

An optional Lua table can be specified as the last argument to this method to specify various connect options:

* `pool`
    specify a custom name for the connection pool being used. If omitted, then the connection pool name will be generated from the string template `"<host>:<port>"` or `"<unix-socket-path>"`.

The support for the options table argument was first introduced in the `v0.5.7` release.

This method was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

tcpsock:sslhandshake
--------------------
**syntax:** *session, err = tcpsock:sslhandshake(reused_session?, server_name?, ssl_verify?, send_status_req?)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Does SSL/TLS handshake on the currently established connection.

The optional `reused_session` argument can take a former SSL
session userdata returned by a previous `sslhandshake`
call for exactly the same target. For short-lived connections, reusing SSL
sessions can usually speed up the handshake by one order by magnitude but it
is not so useful if the connection pool is enabled. This argument defaults to
`nil`. If this argument takes the boolean `false` value, no SSL session
userdata would return by this call and only a Lua boolean will be returned as
the first return value; otherwise the current SSL session will
always be returned as the first argument in case of successes.

The optional `server_name` argument is used to specify the server
name for the new TLS extension Server Name Indication (SNI). Use of SNI can
make different servers share the same IP address on the server side. Also,
when SSL verification is enabled, this `server_name` argument is
also used to validate the server name specified in the server certificate sent from
the remote.

The optional `ssl_verify` argument takes a Lua boolean value to
control whether to perform SSL verification. When set to `true`, the server
certificate will be verified according to the CA certificates specified by
the [lua_ssl_trusted_certificate](#lua_ssl_trusted_certificate) directive.
You may also need to adjust the [lua_ssl_verify_depth](#lua_ssl_verify_depth)
directive to control how deep we should follow along the certificate chain.
Also, when the `ssl_verify` argument is true and the
`server_name` argument is also specified, the latter will be used
to validate the server name in the server certificate.

The optional `send_status_req` argument takes a boolean that controls whether to send
the OCSP status request in the SSL handshake request (which is for requesting OCSP stapling).

For connections that have already done SSL/TLS handshake, this method returns
immediately.

This method was first introduced in the `v0.9.11` release.

[Back to TOC](#nginx-api-for-lua)

tcpsock:send
------------
**syntax:** *bytes, err = tcpsock:send(data)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Sends data without blocking on the current TCP or Unix Domain Socket connection.

This method is a synchronous operation that will not return until *all* the data has been flushed into the system socket send buffer or an error occurs.

In case of success, it returns the total number of bytes that have been sent. Otherwise, it returns `nil` and a string describing the error.

The input argument `data` can either be a Lua string or a (nested) Lua table holding string fragments. In case of table arguments, this method will copy all the string elements piece by piece to the underlying Nginx socket send buffers, which is usually optimal than doing string concatenation operations on the Lua land.

Timeout for the sending operation is controlled by the [lua_socket_send_timeout](#lua_socket_send_timeout) config directive and the [settimeout](#tcpsocksettimeout) method. And the latter takes priority. For example:

```lua

 sock:settimeout(1000)  -- one second timeout
 local bytes, err = sock:send(request)
```

It is important here to call the [settimeout](#tcpsocksettimeout) method *before* calling this method.

In case of any connection errors, this method always automatically closes the current connection.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

tcpsock:receive
---------------
**syntax:** *data, err, partial = tcpsock:receive(size)*

**syntax:** *data, err, partial = tcpsock:receive(pattern?)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Receives data from the connected socket according to the reading pattern or size.

This method is a synchronous operation just like the [send](#tcpsocksend) method and is 100% nonblocking.

In case of success, it returns the data received; in case of error, it returns `nil` with a string describing the error and the partial data received so far.

If a number-like argument is specified (including strings that look like numbers), then it is interpreted as a size. This method will not return until it reads exactly this size of data or an error occurs.

If a non-number-like string argument is specified, then it is interpreted as a "pattern". The following patterns are supported:

* `'*a'`: reads from the socket until the connection is closed. No end-of-line translation is performed;
* `'*l'`: reads a line of text from the socket. The line is terminated by a `Line Feed` (LF) character (ASCII 10), optionally preceded by a `Carriage Return` (CR) character (ASCII 13). The CR and LF characters are not included in the returned line. In fact, all CR characters are ignored by the pattern.

If no argument is specified, then it is assumed to be the pattern `'*l'`, that is, the line reading pattern.

Timeout for the reading operation is controlled by the [lua_socket_read_timeout](#lua_socket_read_timeout) config directive and the [settimeout](#tcpsocksettimeout) method. And the latter takes priority. For example:

```lua

 sock:settimeout(1000)  -- one second timeout
 local line, err, partial = sock:receive()
 if not line then
     ngx.say("failed to read a line: ", err)
     return
 end
 ngx.say("successfully read a line: ", line)
```

It is important here to call the [settimeout](#tcpsocksettimeout) method *before* calling this method.

Since the `v0.8.8` release, this method no longer automatically closes the current connection when the read timeout error happens. For other connection errors, this method always automatically closes the connection.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

tcpsock:receiveuntil
--------------------
**syntax:** *iterator = tcpsock:receiveuntil(pattern, options?)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

This method returns an iterator Lua function that can be called to read the data stream until it sees the specified pattern or an error occurs.

Here is an example for using this method to read a data stream with the boundary sequence `--abcedhb`:

```lua

 local reader = sock:receiveuntil("\r\n--abcedhb")
 local data, err, partial = reader()
 if not data then
     ngx.say("failed to read the data stream: ", err)
 end
 ngx.say("read the data stream: ", data)
```

When called without any argument, the iterator function returns the received data right *before* the specified pattern string in the incoming data stream. So for the example above, if the incoming data stream is `'hello, world! -agentzh\r\n--abcedhb blah blah'`, then the string `'hello, world! -agentzh'` will be returned.

In case of error, the iterator function will return `nil` along with a string describing the error and the partial data bytes that have been read so far.

The iterator function can be called multiple times and can be mixed safely with other cosocket method calls or other iterator function calls.

The iterator function behaves differently (i.e., like a real iterator) when it is called with a `size` argument. That is, it will read that `size` of data on each invocation and will return `nil` at the last invocation (either sees the boundary pattern or meets an error). For the last successful invocation of the iterator function, the `err` return value will be `nil` too. The iterator function will be reset after the last successful invocation that returns `nil` data and `nil` error. Consider the following example:

```lua

 local reader = sock:receiveuntil("\r\n--abcedhb")

 while true do
     local data, err, partial = reader(4)
     if not data then
         if err then
             ngx.say("failed to read the data stream: ", err)
             break
         end

         ngx.say("read done")
         break
     end
     ngx.say("read chunk: [", data, "]")
 end
```

Then for the incoming data stream `'hello, world! -agentzh\r\n--abcedhb blah blah'`, we shall get the following output from the sample code above:


    read chunk: [hell]
    read chunk: [o, w]
    read chunk: [orld]
    read chunk: [! -a]
    read chunk: [gent]
    read chunk: [zh]
    read done


Note that, the actual data returned *might* be a little longer than the size limit specified by the `size` argument when the boundary pattern has ambiguity for streaming parsing. Near the boundary of the data stream, the data string actually returned could also be shorter than the size limit.

Timeout for the iterator function's reading operation is controlled by the [lua_socket_read_timeout](#lua_socket_read_timeout) config directive and the [settimeout](#tcpsocksettimeout) method. And the latter takes priority. For example:

```lua

 local readline = sock:receiveuntil("\r\n")

 sock:settimeout(1000)  -- one second timeout
 line, err, partial = readline()
 if not line then
     ngx.say("failed to read a line: ", err)
     return
 end
 ngx.say("successfully read a line: ", line)
```

It is important here to call the [settimeout](#tcpsocksettimeout) method *before* calling the iterator function (note that the `receiveuntil` call is irrelevant here).

As from the `v0.5.1` release, this method also takes an optional `options` table argument to control the behavior. The following options are supported:

* `inclusive`

The `inclusive` takes a boolean value to control whether to include the pattern string in the returned data string. Default to `false`. For example,

```lua

 local reader = tcpsock:receiveuntil("_END_", { inclusive = true })
 local data = reader()
 ngx.say(data)
```

Then for the input data stream `"hello world _END_ blah blah blah"`, then the example above will output `hello world _END_`, including the pattern string `_END_` itself.

Since the `v0.8.8` release, this method no longer automatically closes the current connection when the read timeout error happens. For other connection errors, this method always automatically closes the connection.

This method was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

tcpsock:close
-------------
**syntax:** *ok, err = tcpsock:close()*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Closes the current TCP or stream unix domain socket. It returns the `1` in case of success and returns `nil` with a string describing the error otherwise.

Note that there is no need to call this method on socket objects that have invoked the [setkeepalive](#tcpsocksetkeepalive) method because the socket object is already closed (and the current connection is saved into the built-in connection pool).

Socket objects that have not invoked this method (and associated connections) will be closed when the socket object is released by the Lua GC (Garbage Collector) or the current client HTTP request finishes processing.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

tcpsock:settimeout
------------------
**syntax:** *tcpsock:settimeout(time)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Set the timeout value in milliseconds for subsequent socket operations ([connect](#tcpsockconnect), [receive](#tcpsockreceive), and iterators returned from [receiveuntil](#tcpsockreceiveuntil)).

Settings done by this method takes priority over those config directives, i.e., [lua_socket_connect_timeout](#lua_socket_connect_timeout), [lua_socket_send_timeout](#lua_socket_send_timeout), and [lua_socket_read_timeout](#lua_socket_read_timeout).

Note that this method does *not* affect the [lua_socket_keepalive_timeout](#lua_socket_keepalive_timeout) setting; the `timeout` argument to the [setkeepalive](#tcpsocksetkeepalive) method should be used for this purpose instead.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

tcpsock:setoption
-----------------
**syntax:** *tcpsock:setoption(option, value?)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

This function is added for [LuaSocket](http://w3.impa.br/~diego/software/luasocket/tcp.html) API compatibility and does nothing for now. Its functionality will be implemented in future.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

tcpsock:setkeepalive
--------------------
**syntax:** *ok, err = tcpsock:setkeepalive(timeout?, size?)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Puts the current socket's connection immediately into the cosocket built-in connection pool and keep it alive until other [connect](#tcpsockconnect) method calls request it or the associated maximal idle timeout is expired.

The first optional argument, `timeout`, can be used to specify the maximal idle timeout (in milliseconds) for the current connection. If omitted, the default setting in the [lua_socket_keepalive_timeout](#lua_socket_keepalive_timeout) config directive will be used. If the `0` value is given, then the timeout interval is unlimited.

The second optional argument, `size`, can be used to specify the maximal number of connections allowed in the connection pool for the current server (i.e., the current host-port pair or the unix domain socket file path). Note that the size of the connection pool cannot be changed once the pool is created. When this argument is omitted, the default setting in the [lua_socket_pool_size](#lua_socket_pool_size) config directive will be used.

When the connection pool exceeds the available size limit, the least recently used (idle) connection already in the pool will be closed to make room for the current connection.

Note that the cosocket connection pool is per Nginx worker process rather than per Nginx server instance, so the size limit specified here also applies to every single Nginx worker process.

Idle connections in the pool will be monitored for any exceptional events like connection abortion or unexpected incoming data on the line, in which cases the connection in question will be closed and removed from the pool.

In case of success, this method returns `1`; otherwise, it returns `nil` and a string describing the error.

When the system receive buffer for the current connection has unread data, then this method will return the "connection in dubious state" error message (as the second return value) because the previous session has unread data left behind for the next session and the connection is not safe to be reused.

This method also makes the current cosocket object enter the "closed" state, so there is no need to manually call the [close](#tcpsockclose) method on it afterwards.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

tcpsock:getreusedtimes
----------------------
**syntax:** *count, err = tcpsock:getreusedtimes()*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

This method returns the (successfully) reused times for the current connection. In case of error, it returns `nil` and a string describing the error.

If the current connection does not come from the built-in connection pool, then this method always returns `0`, that is, the connection has never been reused (yet). If the connection comes from the connection pool, then the return value is always non-zero. So this method can also be used to determine if the current connection comes from the pool.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

ngx.socket.connect
------------------
**syntax:** *tcpsock, err = ngx.socket.connect(host, port)*

**syntax:** *tcpsock, err = ngx.socket.connect("unix:/path/to/unix-domain.socket")*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;*

This function is a shortcut for combining [ngx.socket.tcp()](#ngxsockettcp) and the [connect()](#tcpsockconnect) method call in a single operation. It is actually implemented like this:

```lua

 local sock = ngx.socket.tcp()
 local ok, err = sock:connect(...)
 if not ok then
     return nil, err
 end
 return sock
```

There is no way to use the [settimeout](#tcpsocksettimeout) method to specify connecting timeout for this method and the [lua_socket_connect_timeout](#lua_socket_connect_timeout) directive must be set at configure time instead.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

ngx.get_phase
-------------
**syntax:** *str = ngx.get_phase()*

**context:** *init_by_lua&#42;, init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;, balancer_by_lua&#42;, ssl_certificate_by_lua&#42;*

Retrieves the current running phase name. Possible return values are

* `init`
    for the context of [init_by_lua](#init_by_lua) or [init_by_lua_file](#init_by_lua_file).
* `init_worker`
    for the context of [init_worker_by_lua](#init_worker_by_lua) or [init_worker_by_lua_file](#init_worker_by_lua_file).
* `set`
    for the context of [set_by_lua](#set_by_lua) or [set_by_lua_file](#set_by_lua_file).
* `rewrite`
    for the context of [rewrite_by_lua](#rewrite_by_lua) or [rewrite_by_lua_file](#rewrite_by_lua_file).
* `access`
    for the context of [access_by_lua](#access_by_lua) or [access_by_lua_file](#access_by_lua_file).
* `content`
    for the context of [content_by_lua](#content_by_lua) or [content_by_lua_file](#content_by_lua_file).
* `header_filter`
    for the context of [header_filter_by_lua](#header_filter_by_lua) or [header_filter_by_lua_file](#header_filter_by_lua_file).
* `body_filter`
    for the context of [body_filter_by_lua](#body_filter_by_lua) or [body_filter_by_lua_file](#body_filter_by_lua_file).
* `log`
    for the context of [log_by_lua](#log_by_lua) or [log_by_lua_file](#log_by_lua_file).
* `timer`
    for the context of user callback functions for [ngx.timer.*](#ngxtimerat).

This API was first introduced in the `v0.5.10` release.

[Back to TOC](#nginx-api-for-lua)

ngx.thread.spawn
----------------
**syntax:** *co = ngx.thread.spawn(func, arg1, arg2, ...)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Spawns a new user "light thread" with the Lua function `func` as well as those optional arguments `arg1`, `arg2`, and etc. Returns a Lua thread (or Lua coroutine) object represents this "light thread".

"Light threads" are just a special kind of Lua coroutines that are scheduled by the ngx_lua module.

Before `ngx.thread.spawn` returns, the `func` will be called with those optional arguments until it returns, aborts with an error, or gets yielded due to I/O operations via the [Nginx API for Lua](#nginx-api-for-lua) (like [tcpsock:receive](#tcpsockreceive)).

After `ngx.thread.spawn` returns, the newly-created "light thread" will keep running asynchronously usually at various I/O events.

All the Lua code chunks running by [rewrite_by_lua](#rewrite_by_lua), [access_by_lua](#access_by_lua), and [content_by_lua](#content_by_lua) are in a boilerplate "light thread" created automatically by ngx_lua. Such boilerplate "light thread" are also called "entry threads".

By default, the corresponding Nginx handler (e.g., [rewrite_by_lua](#rewrite_by_lua) handler) will not terminate until

1. both the "entry thread" and all the user "light threads" terminates,
1. a "light thread" (either the "entry thread" or a user "light thread" aborts by calling [ngx.exit](#ngxexit), [ngx.exec](#ngxexec), [ngx.redirect](#ngxredirect), or [ngx.req.set_uri(uri, true)](#ngxreqset_uri), or
1. the "entry thread" terminates with a Lua error.

When the user "light thread" terminates with a Lua error, however, it will not abort other running "light threads" like the "entry thread" does.

Due to the limitation in the Nginx subrequest model, it is not allowed to abort a running Nginx subrequest in general. So it is also prohibited to abort a running "light thread" that is pending on one ore more Nginx subrequests. You must call [ngx.thread.wait](#ngxthreadwait) to wait for those "light thread" to terminate before quitting the "world". A notable exception here is that you can abort pending subrequests by calling [ngx.exit](#ngxexit) with and only with the status code `ngx.ERROR` (-1), `408`, `444`, or `499`.

The "light threads" are not scheduled in a pre-emptive way. In other words, no time-slicing is performed automatically. A "light thread" will keep running exclusively on the CPU until

1. a (nonblocking) I/O operation cannot be completed in a single run,
1. it calls [coroutine.yield](#coroutineyield) to actively give up execution, or
1. it is aborted by a Lua error or an invocation of [ngx.exit](#ngxexit), [ngx.exec](#ngxexec), [ngx.redirect](#ngxredirect), or [ngx.req.set_uri(uri, true)](#ngxreqset_uri).

For the first two cases, the "light thread" will usually be resumed later by the ngx_lua scheduler unless a "stop-the-world" event happens.

User "light threads" can create "light threads" themselves. And normal user coroutines created by [coroutine.create](#coroutinecreate) can also create "light threads". The coroutine (be it a normal Lua coroutine or a "light thread") that directly spawns the "light thread" is called the "parent coroutine" for the "light thread" newly spawned.

The "parent coroutine" can call [ngx.thread.wait](#ngxthreadwait) to wait on the termination of its child "light thread".

You can call coroutine.status() and coroutine.yield() on the "light thread" coroutines.

The status of the "light thread" coroutine can be "zombie" if

1. the current "light thread" already terminates (either successfully or with an error),
1. its parent coroutine is still alive, and
1. its parent coroutine is not waiting on it with [ngx.thread.wait](#ngxthreadwait).

The following example demonstrates the use of coroutine.yield() in the "light thread" coroutines
to do manual time-slicing:

```lua

 local yield = coroutine.yield

 function f()
     local self = coroutine.running()
     ngx.say("f 1")
     yield(self)
     ngx.say("f 2")
     yield(self)
     ngx.say("f 3")
 end

 local self = coroutine.running()
 ngx.say("0")
 yield(self)

 ngx.say("1")
 ngx.thread.spawn(f)

 ngx.say("2")
 yield(self)

 ngx.say("3")
 yield(self)

 ngx.say("4")
```

Then it will generate the output


    0
    1
    f 1
    2
    f 2
    3
    f 3
    4


"Light threads" are mostly useful for doing concurrent upstream requests in a single Nginx request handler, kinda like a generalized version of [ngx.location.capture_multi](#ngxlocationcapture_multi) that can work with all the [Nginx API for Lua](#nginx-api-for-lua). The following example demonstrates parallel requests to MySQL, Memcached, and upstream HTTP services in a single Lua handler, and outputting the results in the order that they actually return (very much like the Facebook BigPipe model):

```lua

 -- query mysql, memcached, and a remote http service at the same time,
 -- output the results in the order that they
 -- actually return the results.

 local mysql = require "resty.mysql"
 local memcached = require "resty.memcached"

 local function query_mysql()
     local db = mysql:new()
     db:connect{
                 host = "127.0.0.1",
                 port = 3306,
                 database = "test",
                 user = "monty",
                 password = "mypass"
               }
     local res, err, errno, sqlstate =
             db:query("select * from cats order by id asc")
     db:set_keepalive(0, 100)
     ngx.say("mysql done: ", cjson.encode(res))
 end

 local function query_memcached()
     local memc = memcached:new()
     memc:connect("127.0.0.1", 11211)
     local res, err = memc:get("some_key")
     ngx.say("memcached done: ", res)
 end

 local function query_http()
     local res = ngx.location.capture("/my-http-proxy")
     ngx.say("http done: ", res.body)
 end

 ngx.thread.spawn(query_mysql)      -- create thread 1
 ngx.thread.spawn(query_memcached)  -- create thread 2
 ngx.thread.spawn(query_http)       -- create thread 3
```

This API was first enabled in the `v0.7.0` release.

[Back to TOC](#nginx-api-for-lua)

ngx.thread.wait
---------------
**syntax:** *ok, res1, res2, ... = ngx.thread.wait(thread1, thread2, ...)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;, ssl_certificate_by_lua&#42;*

Waits on one or more child "light threads" and returns the results of the first "light thread" that terminates (either successfully or with an error).

The arguments `thread1`, `thread2`, and etc are the Lua thread objects returned by earlier calls of [ngx.thread.spawn](#ngxthreadspawn).

The return values have exactly the same meaning as [coroutine.resume](#coroutineresume), that is, the first value returned is a boolean value indicating whether the "light thread" terminates successfully or not, and subsequent values returned are the return values of the user Lua function that was used to spawn the "light thread" (in case of success) or the error object (in case of failure).

Only the direct "parent coroutine" can wait on its child "light thread", otherwise a Lua exception will be raised.

The following example demonstrates the use of `ngx.thread.wait` and [ngx.location.capture](#ngxlocationcapture) to emulate [ngx.location.capture_multi](#ngxlocationcapture_multi):

```lua

 local capture = ngx.location.capture
 local spawn = ngx.thread.spawn
 local wait = ngx.thread.wait
 local say = ngx.say

 local function fetch(uri)
     return capture(uri)
 end

 local threads = {
     spawn(fetch, "/foo"),
     spawn(fetch, "/bar"),
     spawn(fetch, "/baz")
 }

 for i = 1, #threads do
     local ok, res = wait(threads[i])
     if not ok then
         say(i, ": failed to run: ", res)
     else
         say(i, ": status: ", res.status)
         say(i, ": body: ", res.body)
     end
 end
```

Here it essentially implements the "wait all" model.

And below is an example demonstrating the "wait any" model:

```lua

 function f()
     ngx.sleep(0.2)
     ngx.say("f: hello")
     return "f done"
 end

 function g()
     ngx.sleep(0.1)
     ngx.say("g: hello")
     return "g done"
 end

 local tf, err = ngx.thread.spawn(f)
 if not tf then
     ngx.say("failed to spawn thread f: ", err)
     return
 end

 ngx.say("f thread created: ", coroutine.status(tf))

 local tg, err = ngx.thread.spawn(g)
 if not tg then
     ngx.say("failed to spawn thread g: ", err)
     return
 end

 ngx.say("g thread created: ", coroutine.status(tg))

 ok, res = ngx.thread.wait(tf, tg)
 if not ok then
     ngx.say("failed to wait: ", res)
     return
 end

 ngx.say("res: ", res)

 -- stop the "world", aborting other running threads
 ngx.exit(ngx.OK)
```

And it will generate the following output:


    f thread created: running
    g thread created: running
    g: hello
    res: g done


This API was first enabled in the `v0.7.0` release.

[Back to TOC](#nginx-api-for-lua)

ngx.thread.kill
---------------
**syntax:** *ok, err = ngx.thread.kill(thread)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;*

Kills a running "light thread" created by [ngx.thread.spawn](#ngxthreadspawn). Returns a true value when successful or `nil` and a string describing the error otherwise.

According to the current implementation, only the parent coroutine (or "light thread") can kill a thread. Also, a running "light thread" with pending NGINX subrequests (initiated by [ngx.location.capture](#ngxlocationcapture) for example) cannot be killed due to a limitation in the NGINX core.

This API was first enabled in the `v0.9.9` release.

[Back to TOC](#nginx-api-for-lua)

ngx.on_abort
------------
**syntax:** *ok, err = ngx.on_abort(callback)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;*

Registers a user Lua function as the callback which gets called automatically when the client closes the (downstream) connection prematurely.

Returns `1` if the callback is registered successfully or returns `nil` and a string describing the error otherwise.

All the [Nginx API for Lua](#nginx-api-for-lua) can be used in the callback function because the function is run in a special "light thread", just as those "light threads" created by [ngx.thread.spawn](#ngxthreadspawn).

The callback function can decide what to do with the client abortion event all by itself. For example, it can simply ignore the event by doing nothing and the current Lua request handler will continue executing without interruptions. And the callback function can also decide to terminate everything by calling [ngx.exit](#ngxexit), for example,

```lua

 local function my_cleanup()
     -- custom cleanup work goes here, like cancelling a pending DB transaction

     -- now abort all the "light threads" running in the current request handler
     ngx.exit(499)
 end

 local ok, err = ngx.on_abort(my_cleanup)
 if not ok then
     ngx.log(ngx.ERR, "failed to register the on_abort callback: ", err)
     ngx.exit(500)
 end
```

When [lua_check_client_abort](#lua_check_client_abort) is set to `off` (which is the default), then this function call will always return the error message "lua_check_client_abort is off".

According to the current implementation, this function can only be called once in a single request handler; subsequent calls will return the error message "duplicate call".

This API was first introduced in the `v0.7.4` release.

See also [lua_check_client_abort](#lua_check_client_abort).

[Back to TOC](#nginx-api-for-lua)

ngx.timer.at
------------
**syntax:** *ok, err = ngx.timer.at(delay, callback, user_arg1, user_arg2, ...)*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Creates an Nginx timer with a user callback function as well as optional user arguments.

The first argument, `delay`, specifies the delay for the timer,
in seconds. One can specify fractional seconds like `0.001` to mean 1
millisecond here. `0` delay can also be specified, in which case the
timer will immediately expire when the current handler yields
execution.

The second argument, `callback`, can
be any Lua function, which will be invoked later in a background
"light thread" after the delay specified. The user callback will be
called automatically by the Nginx core with the arguments `premature`,
`user_arg1`, `user_arg2`, and etc, where the `premature`
argument takes a boolean value indicating whether it is a premature timer
expiration or not, and `user_arg1`, `user_arg2`, and etc, are
those (extra) user arguments specified when calling `ngx.timer.at`
as the remaining arguments.

Premature timer expiration happens when the Nginx worker process is
trying to shut down, as in an Nginx configuration reload triggered by
the `HUP` signal or in an Nginx server shutdown. When the Nginx worker
is trying to shut down, one can no longer call `ngx.timer.at` to
create new timers with nonzero delays and in that case `ngx.timer.at` will return `nil` and
a string describing the error, that is, "process exiting".

Starting from the `v0.9.3` release, it is allowed to create zero-delay timers even when the Nginx worker process starts shutting down.

When a timer expires, the user Lua code in the timer callback is
running in a "light thread" detached completely from the original
request creating the timer. So objects with the same lifetime as the
request creating them, like [cosockets](#ngxsockettcp), cannot be shared between the
original request and the timer user callback function.

Here is a simple example:

```nginx

 location / {
     ...
     log_by_lua '
         local function push_data(premature, uri, args, status)
             -- push the data uri, args, and status to the remote
             -- via ngx.socket.tcp or ngx.socket.udp
             -- (one may want to buffer the data in Lua a bit to
             -- save I/O operations)
         end
         local ok, err = ngx.timer.at(0, push_data,
                                      ngx.var.uri, ngx.var.args, ngx.header.status)
         if not ok then
             ngx.log(ngx.ERR, "failed to create timer: ", err)
             return
         end
     ';
 }
```

One can also create infinite re-occuring timers, for instance, a timer getting triggered every `5` seconds, by calling `ngx.timer.at` recursively in the timer callback function. Here is such an example,

```lua

 local delay = 5
 local handler
 handler = function (premature)
     -- do some routine job in Lua just like a cron job
     if premature then
         return
     end
     local ok, err = ngx.timer.at(delay, handler)
     if not ok then
         ngx.log(ngx.ERR, "failed to create the timer: ", err)
         return
     end
 end

 local ok, err = ngx.timer.at(delay, handler)
 if not ok then
     ngx.log(ngx.ERR, "failed to create the timer: ", err)
     return
 end
```

Because timer callbacks run in the background and their running time
will not add to any client request's response time, they can easily
accumulate in the server and exhaust system resources due to either
Lua programming mistakes or just too much client traffic. To prevent
extreme consequences like crashing the Nginx server, there are
built-in limitations on both the number of "pending timers" and the
number of "running timers" in an Nginx worker process. The "pending
timers" here mean timers that have not yet been expired and "running
timers" are those whose user callbacks are currently running.

The maximal number of pending timers allowed in an Nginx
worker is constrolled by the [lua_max_pending_timers](#lua_max_pending_timers)
directive. The maximal number of running timers is controlled by the
[lua_max_running_timers](#lua_max_running_timers) directive.

According to the current implementation, each "running timer" will
take one (fake) connection record from the global connection record
list configured by the standard [worker_connections](http://nginx.org/en/docs/ngx_core_module.html#worker_connections) directive in
`nginx.conf`. So ensure that the
[worker_connections](http://nginx.org/en/docs/ngx_core_module.html#worker_connections) directive is set to
a large enough value that takes into account both the real connections
and fake connections required by timer callbacks (as limited by the
[lua_max_running_timers](#lua_max_running_timers) directive).

A lot of the Lua APIs for Nginx are enabled in the context of the timer
callbacks, like stream/datagram cosockets ([ngx.socket.tcp](#ngxsockettcp) and [ngx.socket.udp](#ngxsocketudp)), shared
memory dictionaries ([ngx.shared.DICT](#ngxshareddict)), user coroutines ([coroutine.*](#coroutinecreate)),
user "light threads" ([ngx.thread.*](#ngxthreadspawn)), [ngx.exit](#ngxexit), [ngx.now](#ngxnow)/[ngx.time](#ngxtime),
[ngx.md5](#ngxmd5)/[ngx.sha1_bin](#ngxsha1_bin), are all allowed. But the subrequest API (like
[ngx.location.capture](#ngxlocationcapture)), the [ngx.req.*](#ngxreqstart_time) API, the downstream output API
(like [ngx.say](#ngxsay), [ngx.print](#ngxprint), and [ngx.flush](#ngxflush)) are explicitly disabled in
this context.

You can pass most of the standard Lua values (nils, booleans, numbers, strings, tables, closures, file handles, and etc) into the timer callback, either explicitly as user arguments or implicitly as upvalues for the callback closure. There are several exceptions, however: you *cannot* pass any thread objects returned by [coroutine.create](#coroutinecreate) and [ngx.thread.spawn](#ngxthreadspawn) or any cosocket objects returned by [ngx.socket.tcp](#ngxsockettcp), [ngx.socket.udp](#ngxsocketudp), and [ngx.req.socket](#ngxreqsocket) because these objects' lifetime is bound to the request context creating them while the timer callback is detached from the creating request's context (by design) and runs in its own (fake) request context. If you try to share the thread or cosocket objects across the boundary of the creating request, then you will get the "no co ctx found" error (for threads) or "bad request" (for cosockets). It is fine, however, to create all these objects inside your timer callback.

This API was first introduced in the `v0.8.0` release.

[Back to TOC](#nginx-api-for-lua)

ngx.timer.running_count
-----------------------
**syntax:** *count = ngx.timer.running_count()*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns the number of timers currently running.

This directive was first introduced in the `v0.9.20` release.

[Back to TOC](#nginx-api-for-lua)

ngx.timer.pending_count
-----------------------
**syntax:** *count = ngx.timer.pending_count()*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Returns the number of pending timers.

This directive was first introduced in the `v0.9.20` release.

[Back to TOC](#nginx-api-for-lua)

ngx.config.debug
----------------
**syntax:** *debug = ngx.config.debug*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;, init_by_lua&#42;, init_worker_by_lua&#42;*

This boolean field indicates whether the current Nginx is a debug build, i.e., being built by the `./configure` option `--with-debug`.

This field was first introduced in the `0.8.7`.

[Back to TOC](#nginx-api-for-lua)

ngx.config.prefix
-----------------

**syntax:** *prefix = ngx.config.prefix()*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;, init_by_lua&#42;, init_worker_by_lua&#42;*

Returns the Nginx server "prefix" path, as determined by the `-p` command-line option when running the nginx executable, or the path specified by the `--prefix` command-line option when building Nginx with the `./configure` script.

This function was first introduced in the `0.9.2`.

[Back to TOC](#nginx-api-for-lua)

ngx.config.nginx_version
------------------------

**syntax:** *ver = ngx.config.nginx_version*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;, init_by_lua&#42;, init_worker_by_lua&#42;*

This field take an integral value indicating the version number of the current Nginx core being used. For example, the version number `1.4.3` results in the Lua number 1004003.

This API was first introduced in the `0.9.3` release.

[Back to TOC](#nginx-api-for-lua)

ngx.config.nginx_configure
--------------------------

**syntax:** *str = ngx.config.nginx_configure()*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;, init_by_lua&#42;*

This function returns a string for the NGINX `./configure` command's arguments string.

This API was first introduced in the `0.9.5` release.

[Back to TOC](#nginx-api-for-lua)

ngx.config.ngx_lua_version
--------------------------

**syntax:** *ver = ngx.config.ngx_lua_version*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;, init_by_lua&#42;*

This field take an integral value indicating the version number of the current `ngx_lua` module being used. For example, the version number `0.9.3` results in the Lua number 9003.

This API was first introduced in the `0.9.3` release.

[Back to TOC](#nginx-api-for-lua)

ngx.worker.exiting
------------------

**syntax:** *exiting = ngx.worker.exiting()*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;, init_by_lua&#42;, init_worker_by_lua&#42;*

This function returns a boolean value indicating whether the current Nginx worker process already starts exiting. Nginx worker process exiting happens on Nginx server quit or configuration reload (aka HUP reload).

This API was first introduced in the `0.9.3` release.

[Back to TOC](#nginx-api-for-lua)

ngx.worker.pid
--------------

**syntax:** *pid = ngx.worker.pid()*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;, init_by_lua&#42;, init_worker_by_lua&#42;*

This function returns a Lua number for the process ID (PID) of the current Nginx worker process. This API is more efficient than `ngx.var.pid` and can be used in contexts where the [ngx.var.VARIABLE](#ngxvarvariable) API cannot be used (like [init_worker_by_lua](#init_worker_by_lua)).

This API was first introduced in the `0.9.5` release.

[Back to TOC](#nginx-api-for-lua)

ngx.worker.count
----------------

**syntax:** *count = ngx.worker.count()*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;, init_by_lua&#42;*

Returns the total number of the Nginx worker processes (i.e., the value configured
by the [worker_processes](http://nginx.org/en/docs/ngx_core_module.html#worker_processes)
directive in `nginx.conf`).

This API was first introduced in the `0.9.20` release.

[Back to TOC](#nginx-api-for-lua)

ngx.worker.id
-------------

**syntax:** *count = ngx.worker.id()*

**context:** *set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;, init_worker_by_lua&#42;*

Returns the ordinal number of the current Nginx worker processes (starting from number 0).

So if the total number of workers is `N`, then this method may return a number between 0
and `N - 1` (inclusive).

This function returns meaningful values only for NGINX 1.9.1+. With earlier versions of NGINX, it
always returns `nil`.

See also [ngx.worker.count](#ngxworkercount).

This API was first introduced in the `0.9.20` release.

[Back to TOC](#nginx-api-for-lua)

ngx.semaphore
-------------
**syntax:** *local semaphore = require "ngx.semaphore"*

This is a Lua module that implements a classic-style semaphore API for efficient synchronizations among
different "light threads". Sharing the same semaphore among different "light threads" created in different (request)
contexts are also supported as long as the "light threads" reside in the same NGINX worker process
and the [lua_code_cache](#lua_code_cache) directive is turned on (which is the default).

This Lua module does not ship with this ngx_lua module itself rather it is shipped with
the
[lua-resty-core](https://github.com/openresty/lua-resty-core) library.

Please refer to the [documentation](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/semaphore.md)
for this `ngx.semaphore` Lua module in [lua-resty-core](https://github.com/openresty/lua-resty-core)
for more details.

This feature requires at least ngx_lua `v0.10.0`.

[Back to TOC](#nginx-api-for-lua)

ngx.balancer
------------
**syntax:** *local balancer = require "ngx.balancer"*

This is a Lua module that provides a Lua API to allow defining completely dynamic load balancers
in pure Lua.

This Lua module does not ship with this ngx_lua module itself rather it is shipped with
the
[lua-resty-core](https://github.com/openresty/lua-resty-core) library.

Please refer to the [documentation](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/balancer.md)
for this `ngx.balancer` Lua module in [lua-resty-core](https://github.com/openresty/lua-resty-core)
for more details.

This feature requires at least ngx_lua `v0.10.0`.

[Back to TOC](#nginx-api-for-lua)

ngx.ssl
-------
**syntax:** *local ssl = require "ngx.ssl"*

This Lua module provides API functions to control the SSL handshake process in contexts like
[ssl_certificate_by_lua*](#ssl_certificate_by_lua_block).

This Lua module does not ship with this ngx_lua module itself rather it is shipped with
the
[lua-resty-core](https://github.com/openresty/lua-resty-core) library.

Please refer to the [documentation](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/ssl.md)
for this `ngx.ssl` Lua module for more details.

This feature requires at least ngx_lua `v0.10.0`.

[Back to TOC](#nginx-api-for-lua)

ngx.ocsp
--------
**syntax:** *local ocsp = require "ngx.ocsp"*

This Lua module provides API to perform OCSP queries, OCSP response validations, and
OCSP stapling planting.

Usually, this module is used together with the [ngx.ssl](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/ssl.md)
module in the
context of [ssl_certificate_by_lua*](#ssl_certificate_by_lua_block).

This Lua module does not ship with this ngx_lua module itself rather it is shipped with
the
[lua-resty-core](https://github.com/openresty/lua-resty-core) library.

Please refer to the [documentation](https://github.com/openresty/lua-resty-core/blob/ocsp-cert-by-lua-2/lib/ngx/ocsp.md)
for this `ngx.ocsp` Lua module for more details.

This feature requires at least ngx_lua `v0.10.0`.

[Back to TOC](#nginx-api-for-lua)

ndk.set_var.DIRECTIVE
---------------------
**syntax:** *res = ndk.set_var.DIRECTIVE_NAME*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

This mechanism allows calling other nginx C modules' directives that are implemented by [Nginx Devel Kit](https://github.com/simpl/ngx_devel_kit) (NDK)'s set_var submodule's `ndk_set_var_value`.

For example, the following [set-misc-nginx-module](http://github.com/openresty/set-misc-nginx-module) directives can be invoked this way:

* [set_quote_sql_str](http://github.com/openresty/set-misc-nginx-module#set_quote_sql_str)
* [set_quote_pgsql_str](http://github.com/openresty/set-misc-nginx-module#set_quote_pgsql_str)
* [set_quote_json_str](http://github.com/openresty/set-misc-nginx-module#set_quote_json_str)
* [set_unescape_uri](http://github.com/openresty/set-misc-nginx-module#set_unescape_uri)
* [set_escape_uri](http://github.com/openresty/set-misc-nginx-module#set_escape_uri)
* [set_encode_base32](http://github.com/openresty/set-misc-nginx-module#set_encode_base32)
* [set_decode_base32](http://github.com/openresty/set-misc-nginx-module#set_decode_base32)
* [set_encode_base64](http://github.com/openresty/set-misc-nginx-module#set_encode_base64)
* [set_decode_base64](http://github.com/openresty/set-misc-nginx-module#set_decode_base64)
* [set_encode_hex](http://github.com/openresty/set-misc-nginx-module#set_encode_base64)
* [set_decode_hex](http://github.com/openresty/set-misc-nginx-module#set_decode_base64)
* [set_sha1](http://github.com/openresty/set-misc-nginx-module#set_encode_base64)
* [set_md5](http://github.com/openresty/set-misc-nginx-module#set_decode_base64)

For instance,

```lua

 local res = ndk.set_var.set_escape_uri('a/b');
 -- now res == 'a%2fb'
```

Similarly, the following directives provided by [encrypted-session-nginx-module](http://github.com/openresty/encrypted-session-nginx-module) can be invoked from within Lua too:

* [set_encrypt_session](http://github.com/openresty/encrypted-session-nginx-module#set_encrypt_session)
* [set_decrypt_session](http://github.com/openresty/encrypted-session-nginx-module#set_decrypt_session)

This feature requires the [ngx_devel_kit](https://github.com/simpl/ngx_devel_kit) module.

[Back to TOC](#nginx-api-for-lua)

coroutine.create
----------------
**syntax:** *co = coroutine.create(f)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, init_by_lua&#42;, ngx.timer.&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, ssl_certificate_by_lua&#42;*

Creates a user Lua coroutines with a Lua function, and returns a coroutine object.

Similar to the standard Lua [coroutine.create](http://www.lua.org/manual/5.1/manual.html#pdf-coroutine.create) API, but works in the context of the Lua coroutines created by ngx_lua.

This API was first usable in the context of [init_by_lua*](#init_by_lua) since the `0.9.2`.

This API was first introduced in the `v0.6.0` release.

[Back to TOC](#nginx-api-for-lua)

coroutine.resume
----------------
**syntax:** *ok, ... = coroutine.resume(co, ...)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, init_by_lua&#42;, ngx.timer.&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, ssl_certificate_by_lua&#42;*

Resumes the executation of a user Lua coroutine object previously yielded or just created.

Similar to the standard Lua [coroutine.resume](http://www.lua.org/manual/5.1/manual.html#pdf-coroutine.resume) API, but works in the context of the Lua coroutines created by ngx_lua.

This API was first usable in the context of [init_by_lua*](#init_by_lua) since the `0.9.2`.

This API was first introduced in the `v0.6.0` release.

[Back to TOC](#nginx-api-for-lua)

coroutine.yield
---------------
**syntax:** *... = coroutine.yield(...)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, init_by_lua&#42;, ngx.timer.&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, ssl_certificate_by_lua&#42;*

Yields the execution of the current user Lua coroutine.

Similar to the standard Lua [coroutine.yield](http://www.lua.org/manual/5.1/manual.html#pdf-coroutine.yield) API, but works in the context of the Lua coroutines created by ngx_lua.

This API was first usable in the context of [init_by_lua*](#init_by_lua) since the `0.9.2`.

This API was first introduced in the `v0.6.0` release.

[Back to TOC](#nginx-api-for-lua)

coroutine.wrap
--------------
**syntax:** *co = coroutine.wrap(f)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, init_by_lua&#42;, ngx.timer.&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, ssl_certificate_by_lua&#42;*

Similar to the standard Lua [coroutine.wrap](http://www.lua.org/manual/5.1/manual.html#pdf-coroutine.wrap) API, but works in the context of the Lua coroutines created by ngx_lua.

This API was first usable in the context of [init_by_lua*](#init_by_lua) since the `0.9.2`.

This API was first introduced in the `v0.6.0` release.

[Back to TOC](#nginx-api-for-lua)

coroutine.running
-----------------
**syntax:** *co = coroutine.running()*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, init_by_lua&#42;, ngx.timer.&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, ssl_certificate_by_lua&#42;*

Identical to the standard Lua [coroutine.running](http://www.lua.org/manual/5.1/manual.html#pdf-coroutine.running) API.

This API was first usable in the context of [init_by_lua*](#init_by_lua) since the `0.9.2`.

This API was first enabled in the `v0.6.0` release.

[Back to TOC](#nginx-api-for-lua)

coroutine.status
----------------
**syntax:** *status = coroutine.status(co)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, init_by_lua&#42;, ngx.timer.&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, ssl_certificate_by_lua&#42;*

Identical to the standard Lua [coroutine.status](http://www.lua.org/manual/5.1/manual.html#pdf-coroutine.status) API.

This API was first usable in the context of [init_by_lua*](#init_by_lua) since the `0.9.2`.

This API was first enabled in the `v0.6.0` release.

[Back to TOC](#nginx-api-for-lua)

Obsolete Sections
=================

This section is just holding obsolete documentation sections that have been either renamed or removed so that existing links over the web are still valid.

[Back to TOC](#table-of-contents)

Special PCRE Sequences
----------------------

This section has been renamed to [Special Escaping Sequences](#special-escaping-sequences).





