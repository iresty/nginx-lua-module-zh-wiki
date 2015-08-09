**语法:** *dict = ngx.shared.DICT*

**语法:** *dict = ngx.shared\[name_var\]*

**环境:** *init_by_lua\*, init_worker_by_lua\*, set_by_lua\*, rewrite_by_lua\*, access_by_lua\*, content_by_lua\*, header_filter_by_lua\*, body_filter_by_lua\*, log_by_lua\*, ngx.timer.\**

获取基于共享内存名为 `DICT` 的 Lua 字典对象，它是一个共享内存区块，通过 [lua_shared_dict](#lua_shared_dict) 指令定义。

所有共享内存区块均被当前 nginx 服务器实例的所有 nginx worker 进程共享。

获取的 `dict` 对象有如下方法：

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

例如：

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

测试结果如下：

```bash

 $ curl localhost/set
 STORED

 $ curl localhost/get
 8

 $ curl localhost/get
 8
```

当访问 `/get` 时，无论有多少 Nginx worker 进程，将一直输出数字 `8`，因为 `dogs` 字典存储在共享内存中，对 *所有* worker 进程可见。

当服务器配置重新加载时共享内存字典内容不会丢失 (不管是通过送 `HUP` 信号给 Nginx 进程或使用命令行 `-s reload` 属性)。

但是，当 Nginx 服务器退出时，字典存储内容将丢失。

这个功能最早出现在 `v0.3.1rc22` 版本中。


> English Source

**syntax:** *dict = ngx.shared.DICT*

**syntax:** *dict = ngx.shared\[name_var\]*

**context:** *init_by_lua\*, init_worker_by_lua\*, set_by_lua\*, rewrite_by_lua\*, access_by_lua\*, content_by_lua\*, header_filter_by_lua\*, body_filter_by_lua\*, log_by_lua\*, ngx.timer.\**

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