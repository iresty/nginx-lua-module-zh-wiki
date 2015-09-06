init_by_lua
-----------

**语法:** *init_by_lua &lt;lua-script-str&gt;*

**作用域:** *http*

**阶段:** *loading-config*

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

> English source:

init_by_lua
-----------

**syntax:** *init_by_lua &lt;lua-script-str&gt;*

**context:** *http*

**phase:** *loading-config*

Runs the Lua code specified by the argument `<lua-script-str>` on the global Lua VM level when the Nginx master process (if any) is loading the Nginx config file.

When Nginx receives the `HUP` signal and starts reloading the config file, the Lua VM will also be re-created and `init_by_lua` will run again on the new Lua VM. In case that the [lua_code_cache](#lua_code_cache) directive is turned off (default on), the `init_by_lua` handler will run upon every request because in this special mode a standalone Lua VM is always created for each request.

Usually you can register (true) Lua global variables or pre-load Lua modules at server start-up by means of this hook. Here is an example for pre-loading Lua modules:

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

You can also initialize the [lua_shared_dict](#lua_shared_dict) shm storage at this phase. Here is an example for this:

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

But note that, the [lua_shared_dict](#lua_shared_dict)'s shm storage will not be cleared through a config reload (via the `HUP` signal, for example). So if you do *not* want to re-initialize the shm storage in your `init_by_lua` code in this case, then you just need to set a custom flag in the shm storage and always check the flag in your `init_by_lua` code.

Because the Lua code in this context runs before Nginx forks its worker processes (if any), data or code loaded here will enjoy the [Copy-on-write (COW)](http://en.wikipedia.org/wiki/Copy-on-write) feature provided by many operating systems among all the worker processes, thus saving a lot of memory.

Do *not* initialize your own Lua global variables in this context because use of Lua global variables have performance penalties and can lead to global namespace pollution (see the [Lua Variable Scope](#lua-variable-scope) section for more details). The recommended way is to use proper [Lua module](http://www.lua.org/manual/5.1/manual.html#5.3) files (but do not use the standard Lua function [module()](http://www.lua.org/manual/5.1/manual.html#pdf-module) to define Lua modules because it pollutes the global namespace as well) and call [require()](http://www.lua.org/manual/5.1/manual.html#pdf-require) to load your own module files in `init_by_lua` or other contexts ([require()](http://www.lua.org/manual/5.1/manual.html#pdf-require) does cache the loaded Lua modules in the global `package.loaded` table in the Lua registry so your modules will only loaded once for the whole Lua VM instance).

Only a small set of the [Nginx API for Lua](#nginx-api-for-lua) is supported in this context:

* Logging APIs: [ngx.log](#ngxlog) and [print](#print),
* Shared Dictionary API: [ngx.shared.DICT](#ngxshareddict).

More Nginx APIs for Lua may be supported in this context upon future user requests.

Basically you can safely use Lua libraries that do blocking I/O in this very context because blocking the master process during server start-up is completely okay. Even the Nginx core does blocking I/O (at least on resolving upstream's host names) at the configure-loading phase.

You should be very careful about potential security vulnerabilities in your Lua code registered in this context because the Nginx master process is often run under the `root` account.

This directive was first introduced in the `v0.5.5` release.

[Back to TOC](#directives)

