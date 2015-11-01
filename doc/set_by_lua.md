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

> English source:

set_by_lua
----------

**syntax:** *set_by_lua $res &lt;lua-script-str&gt; [$arg1 $arg2 ...]*

**context:** *server, server if, location, location if*

**phase:** *rewrite*

**WARNING** Since the `v0.9.17` release, use of this directive is *discouraged*; use the new [set_by_lua_block](#set_by_lua_block) directive instead.

Executes code specified in `<lua-script-str>` with optional input arguments `$arg1 $arg2 ...`, and returns string output to `$res`.

The code in `<lua-script-str>` can make [API calls](#nginx-api-for-lua) and can retrieve input arguments from the `ngx.arg` table (index starts from `1` and increases sequentially).

This directive is designed to execute short, fast running code blocks as the Nginx event loop is blocked during code execution. Time consuming code sequences should therefore be avoided.

This directive is implemented by injecting custom commands into the standard [ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)'s command list. Because [ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html) does not support nonblocking I/O in its commands, Lua APIs requiring yielding the current Lua "light thread" cannot work in this directive.

At least the following API functions are currently disabled within the context of `set_by_lua`:

* Output API functions (e.g., [ngx.say](#ngxsay) and [ngx.send_headers](#ngxsend_headers))
* Control API functions (e.g., [ngx.exit](#ngxexit))
* Subrequest API functions (e.g., [ngx.location.capture](#ngxlocationcapture) and [ngx.location.capture_multi](#ngxlocationcapture_multi))
* Cosocket API functions (e.g., [ngx.socket.tcp](#ngxsockettcp) and [ngx.req.socket](#ngxreqsocket)).
* Sleeping API function [ngx.sleep](#ngxsleep).

In addition, note that this directive can only write out a value to a single Nginx variable at
a time. However, a workaround is possible using the [ngx.var.VARIABLE](#ngxvarvariable) interface.

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

This directive can be freely mixed with all directives of the [ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html), [set-misc-nginx-module](http://github.com/openresty/set-misc-nginx-module), and [array-var-nginx-module](http://github.com/openresty/array-var-nginx-module) modules. All of these directives will run in the same order as they appear in the config file.

```nginx

 set $foo 32;
 set_by_lua $bar 'return tonumber(ngx.var.foo) + 1';
 set $baz "bar: $bar";  # $baz == "bar: 33"
```

As from the `v0.5.0rc29` release, Nginx variable interpolation is disabled in the `<lua-script-str>` argument of this directive and therefore, the dollar sign character (`$`) can be used directly.

This directive requires the [ngx_devel_kit](https://github.com/simpl/ngx_devel_kit) module.

[Back to TOC](#directives)
