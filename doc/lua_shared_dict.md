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


> English Source

**syntax:** *lua_shared_dict &lt;name&gt; &lt;size&gt;*

**default:** *no*

**context:** *http*

**phase:** *depends on usage*

Declares a shared memory zone, `<name>`, to serve as storage for the shm based Lua dictionary `ngx.shared.<name>`.

Shared memory zones are always shared by all the nginx worker processes in the current nginx server instance.

The `<size>` argument accepts size units such as `k` and `m`:

```nginx

 http {
     lua_shared_dict dogs 10m;
     ...
 }
```

The hard-coded minimum size is 8KB while the practical minimum size depends
on actual user data set (some people start with 12KB).

See [ngx.shared.DICT](#ngxshareddict) for details.

This directive was first introduced in the `v0.3.1rc22` release.

[返回目录](#directives)