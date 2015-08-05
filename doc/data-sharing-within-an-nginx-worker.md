Nginx Worker内的数据共享

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
