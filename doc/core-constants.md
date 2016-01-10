核心常量
--------
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


> English Source

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, &#42;log_by_lua&#42;, ngx.timer.&#42;*

```lua

   ngx.OK (0)
   ngx.ERROR (-1)
   ngx.AGAIN (-2)
   ngx.DONE (-4)
   ngx.DECLINED (-5)
```

Note that only three of these constants are utilized by the [Nginx API for Lua](#nginx-api-for-lua) (i.e., [ngx.exit](#ngxexit) accepts `NGX_OK`, `NGX_ERROR`, and `NGX_DECLINED` as input).

```lua

   ngx.null
```

The `ngx.null` constant is a `NULL` light userdata usually used to represent nil values in Lua tables etc and is similar to the [lua-cjson](http://www.kyne.com.au/~mark/software/lua-cjson.php) library's `cjson.null` constant. This constant was first introduced in the `v0.5.0rc5` release.

The `ngx.DECLINED` constant was first introduced in the `v0.5.0rc19` release.

[返回目录](#nginx-api-for-lua)
