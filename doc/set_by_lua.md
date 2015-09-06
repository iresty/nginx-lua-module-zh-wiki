set_by_lua
----------

**syntax:** *set_by_lua $res &lt;lua-script-str&gt; [$arg1 $arg2 ...]*

**context:** *server, server if, location, location if*

**phase:** *rewrite*

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
 set_by_lua $bar 'tonumber(ngx.var.foo) + 1';
 set $baz "bar: $bar";  # $baz == "bar: 33"
```

As from the `v0.5.0rc29` release, Nginx variable interpolation is disabled in the `<lua-script-str>` argument of this directive and therefore, the dollar sign character (`$`) can be used directly.

This directive requires the [ngx_devel_kit](https://github.com/simpl/ngx_devel_kit) module.

[Back to TOC](#directives)

> English source:

set_by_lua
----------

**syntax:** *set_by_lua $res &lt;lua-script-str&gt; [$arg1 $arg2 ...]*

**context:** *server, server if, location, location if*

**phase:** *rewrite*

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
 set_by_lua $bar 'tonumber(ngx.var.foo) + 1';
 set $baz "bar: $bar";  # $baz == "bar: 33"
```

As from the `v0.5.0rc29` release, Nginx variable interpolation is disabled in the `<lua-script-str>` argument of this directive and therefore, the dollar sign character (`$`) can be used directly.

This directive requires the [ngx_devel_kit](https://github.com/simpl/ngx_devel_kit) module.

[Back to TOC](#directives)
