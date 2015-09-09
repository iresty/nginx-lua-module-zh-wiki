lua-variable-scope
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

> English source:

Lua Variable Scope
------------------
Care must be taken when importing modules and this form should be used:

```lua

 local xxx = require('xxx')
```

instead of the old deprecated form:

```lua

 require('xxx')
```

Here is the reason: by design, the global environment has exactly the same lifetime as the Nginx request handler associated with it. Each request handler has its own set of Lua global variables and that is the idea of request isolation. The Lua module is actually loaded by the first Nginx request handler and is cached by the `require()` built-in in the `package.loaded` table for later reference, and the `module()` builtin used by some Lua modules has the side effect of setting a global variable to the loaded module table. But this global variable will be cleared at the end of the request handler,  and every subsequent request handler all has its own (clean) global environment. So one will get Lua exception for accessing the `nil` value.

Generally, use of Lua global variables is a really really bad idea in the context of ngx_lua because

1. misuse of Lua globals has very bad side effects for concurrent requests when these variables are actually supposed to be local only,
1. Lua global variables require Lua table look-up in the global environment (which is just a Lua table), which is kinda expensive, and
1. some Lua global variable references are just typos, which are hard to debug.

It's *highly* recommended to always declare them via "local" in the scope that is reasonable.

To find out all the uses of Lua global variables in your Lua code, you can run the [lua-releng tool](https://github.com/openresty/nginx-devel-utils/blob/master/lua-releng) across all your .lua source files:

    $ lua-releng
    Checking use of Lua global variables in file lib/foo/bar.lua ...
            1       [1489]  SETGLOBAL       7 -1    ; contains
            55      [1506]  GETGLOBAL       7 -3    ; setvar
            3       [1545]  GETGLOBAL       3 -4    ; varexpand

The output says that the line 1489 of file `lib/foo/bar.lua` writes to a global variable named `contains`, the line 1506 reads from the global variable `setvar`, and line 1545 reads the global `varexpand`.

This tool will guarantee that local variables in the Lua module functions are all declared with the `local` keyword, otherwise a runtime exception will be thrown. It prevents undesirable race conditions while accessing such variables. See [Data Sharing within an Nginx Worker](#data-sharing-within-an-nginx-worker) for the reasons behind this.

[Back to TOC](#table-of-contents)


