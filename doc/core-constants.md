核心常量
--------
**环境:** *init_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua, *\**log_by_lua*\**, ngx.timer.*\*

```lua

   ngx.OK (0)
   ngx.ERROR (-1)
   ngx.AGAIN (-2)
   ngx.DONE (-4)
   ngx.DECLINED (-5)
```

请注意，这些常量中只有三个可以被 [Nginx API for Lua](#nginx-api-for-lua) 使用 (即 [ngx.exit](#ngxexit) 接受 `NGX_OK`, `NGX_ERROR`, 和 `NGX_DECLINED` 作为输入)。

```lua

   ngx.null
```

`ngx.null` 常量是一个 `NULL` 轻用户数据 (light userdata)，一般被用来表达 Lua 表或者其他地方的 nil (空) 值，类似于 [lua-cjson](http://www.kyne.com.au/~mark/software/lua-cjson.php) 库中的 `cjson.null` 常量。此常量最早出现在 `v0.5.0rc5` 版本中。

`ngx.DECLINED` 常量最早出现在 `v0.5.0rc19` 版本中。


> English Source

**context:** *init_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua, *\**log_by_lua*\**, ngx.timer.*\*

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