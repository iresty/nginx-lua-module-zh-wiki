安装
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

 wget 'http://nginx.org/download/nginx-1.7.10.tar.gz'
 tar -xzvf nginx-1.7.10.tar.gz
 cd nginx-1.7.10/

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

> English source:

Installation
============

It is highly recommended to use the [ngx_openresty bundle](http://openresty.org) that bundles Nginx, ngx_lua,  LuaJIT 2.0/2.1 (or the optional standard Lua 5.1 interpreter), as well as a package of powerful companion Nginx modules. The basic installation step is a simple command: `./configure --with-luajit && make && make install`.

Alternatively, ngx_lua can be manually compiled into Nginx:

1. Install LuaJIT 2.0 or 2.1 (recommended) or Lua 5.1 (Lua 5.2 is *not* supported yet). LuaJIT can be downloaded from the [the LuaJIT project website](http://luajit.org/download.html) and Lua 5.1, from the [Lua project website](http://www.lua.org/).  Some distribution package managers also distribute LuajIT and/or Lua.
1. Download the latest version of the ngx_devel_kit (NDK) module [HERE](https://github.com/simpl/ngx_devel_kit/tags).
1. Download the latest version of ngx_lua [HERE](https://github.com/openresty/lua-nginx-module/tags).
1. Download the latest version of Nginx [HERE](http://nginx.org/) (See [Nginx Compatibility](#nginx-compatibility))

Build the source with this module:

```bash

 wget 'http://nginx.org/download/nginx-1.7.10.tar.gz'
 tar -xzvf nginx-1.7.10.tar.gz
 cd nginx-1.7.10/

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

[Back to TOC](#table-of-contents)
