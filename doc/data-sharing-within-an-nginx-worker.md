Nginx Worker内的数据共享
========================

在同一个nginx worker进程处理的所有请求中共享数据，需要将共享数据封装进一个Lua模块中，并使用Lua语言内置的`require` 方法加载该模块，之后就可以在 Lua 中操作共享数据了。这种方法之所以工作，是因为(在同一个nginx worker中)加载模块的操作仅被执行一次，所有的协程都会共享同一份拷贝(包括代码和数据)。但是请注意，Lua的全局变量(注意，不是模块级变量)将因为“每个请求一个协程”的隔离要求而不被保持。

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

例子中的`mydata`模块将在第一个请求到达`/lua`的时候被加载运行，之后同一个nginx worker进程处理的所有请求都将使用此模块已经加载的实例，并共享实例中的数据，直到nginx主进程接到`HUP`信号强制重新进行加载。这种数据共享技术是基于本模块(ngx_lua)的高性能Lua应用的基础。

注意，这种数据共享方式是基于worker而不是基于服务器的。也就是说，当nginx主进程下面有多个worker进程时，数据共享不能跨越这些worker之间的进程边界。

一般来说，仅推荐使用这种方式共享只读数据。当计算过程中没有非阻塞性I/O操作时(包括[ngx.sleep](#ngxsleep))，你也可以在nginx worker进程内所有并发请求中共享可改变的数据。只要你不把控制权交还给nginx事件循环以及ngx_lua的轻量级线程调度器(包括隐含的)，它们之间就不会有任何竞争。因此，当你决定在worker中共享可变数据时，一定要非常小心。错误的优化经常会导致在高负载时产生竞争，这种bug非常难以发现。

如果需要在服务器级别共享数据，请使用以下方法：

 1. 使用本模块提供的[ngx.shared.DICT](#ngxshareddict) API
 2. 使用单服务器单nginx worker进程(当使用多CPU或者多核CPU的服务器时不推荐)
 3. 使用类似 `memcached`, `redis`, `MySQL` 或 `PostgreSQL` 等数据共享机制。与本模块相关的[ngx_openresty软件包](http://openresty.org)包含了一系列相关的Nginx模块以及Lua库，提供与这些数据存储机制的交互界面。


> English source:

To globally share data among all the requests handled by the same nginx worker process, encapsulate the shared data into a Lua module, use the Lua `require` builtin to import the module, and then manipulate the shared data in Lua. This works because required Lua modules are loaded only once and all coroutines will share the same copy of the module (both its code and data). Note however that Lua global variables (note, not module-level variables) WILL NOT persist between requests because of the one-coroutine-per-request isolation design.

Here is a complete small example:

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

and then accessing it from `nginx.conf`:

```nginx

 location /lua {
     content_by_lua '
         local mydata = require "mydata"
         ngx.say(mydata.get_age("dog"))
     ';
 }
```

The `mydata` module in this example will only be loaded and run on the first request to the location `/lua`,
and all subsequent requests to the same nginx worker process will use the reloaded instance of the
module as well as the same copy of the data in it, until a `HUP` signal is sent to the Nginx master process to force a reload.
This data sharing technique is essential for high performance Lua applications based on this module.

Note that this data sharing is on a *per-worker* basis and not on a *per-server* basis. That is, when there are multiple nginx worker processes under an Nginx master, data sharing cannot cross the process boundary between these workers.

It is usually recommended to share read-only data this way. You can also share changeable data among all the concurrent requests of each nginx worker process as
long as there is *no* nonblocking I/O operations (including [ngx.sleep](#ngxsleep))
in the middle of your calculations. As long as you do not give the
control back to the nginx event loop and ngx_lua's light thread
scheduler (even implicitly), there can never be any race conditions in
between. For this reason, always be very careful when you want to share changeable data on the
worker level. Buggy optimizations can easily lead to hard-to-debug
race conditions under load.

If server-wide data sharing is required, then use one or more of the following approaches:

1. Use the [ngx.shared.DICT](#ngxshareddict) API provided by this module.
1. Use only a single nginx worker and a single server (this is however not recommended when there is a multi core CPU or multiple CPUs in a single machine).
1. Use data storage mechanisms such as `memcached`, `redis`, `MySQL` or `PostgreSQL`. [The ngx_openresty bundle](http://openresty.org) associated with this module comes with a set of companion Nginx modules and Lua libraries that provide interfaces with these data storage mechanisms.

[返回目录](#nginx-api-for-lua)