ngx.req.set_uri
---------------
**语法:** *ngx.req.set_uri(uri, jump?)*

**环境:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\*

用 `uri` 参数重写当前请求 (已解析过的) URI。该 `uri` 参数必须是 Lua 字符串，并且长度不能是 0，否则将抛出 Lua 异常。
Rewrite the current request's (parsed) URI by the `uri` argument. The `uri` argument must be a Lua string and cannot be of zero length, or a Lua exception will be thrown.

可选的布尔值参数 `jump` 会触发类似 [ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html) 中 [rewrite](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite) 指令的 location 重匹配 (或 location 跳转)。换句话说，当 `jump` 参数是 `true` (默认值 `false`) 时，此函数将不会返回，它会让 Nginx 在之后的 `post-rewrite` 执行阶段，根据新的 URI 值重新搜索 location，并跳转到新 location。
The optional boolean `jump` argument can trigger location rematch (or location jump) as [ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)'s [rewrite](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite) directive, that is, when `jump` is `true` (default to `false`), this function will never return and it will tell Nginx to try re-searching locations with the new URI value at the later `post-rewrite` phase and jumping to the new location.

Location jump will not be triggered otherwise, and only the current request's URI will be modified, which is also the default behavior. This function will return but with no returned values when the `jump` argument is `false` or absent altogether.

例如，下面 nginx 配置片段

```nginx

 rewrite ^ /foo last;
```

可以通过 Lua 代码写成下面这样：

```lua

 ngx.req.set_uri("/foo", true)
```

类似的，Nginx 配置

```nginx

 rewrite ^ /foo break;
```

可以通过 Lua 代码写成：

```lua

 ngx.req.set_uri("/foo", false)
```

等同于写成：

```lua

 ngx.req.set_uri("/foo")
```

`jump` 参数只可以在 [rewrite_by_lua](#rewrite_by_lua) 和 [rewrite_by_lua_file](#rewrite_by_lua_file) 指令中被设置为 `true`。在其他环境中使用 jump 将抛出一个 Lua 异常。
The `jump` argument can only be set to `true` in [rewrite_by_lua](#rewrite_by_lua) and [rewrite_by_lua_file](#rewrite_by_lua_file). Use of jump in other contexts is prohibited and will throw out a Lua exception.

下面的示例复杂一些，包含正则表达式替换：
A more sophisticated example involving regex substitutions is as follows

```nginx

 location /test {
     rewrite_by_lua '
         local uri = ngx.re.sub(ngx.var.uri, "^/test/(.*)", "$1", "o")
         ngx.req.set_uri(uri)
     ';
     proxy_pass http://my_backend;
 }
```

功能上等同于：
which is functionally equivalent to

```nginx

 location /test {
     rewrite ^/test/(.*) /$1 break;
     proxy_pass http://my_backend;
 }
```

请注意，不能使用这个函数重写 URI 参数，应该使用 [ngx.req.set_uri_args](#ngxreqset_uri_args) 代替。例如，Nginx 配置
Note that it is not possible to use this interface to rewrite URI arguments and that [ngx.req.set_uri_args](#ngxreqset_uri_args) should be used for this instead. For instance, Nginx config

```nginx

 rewrite ^ /foo?a=3? last;
```

可以被写成
can be coded as

```nginx

 ngx.req.set_uri_args("a=3")
 ngx.req.set_uri("/foo", true)
```

或

```nginx

 ngx.req.set_uri_args({a = 3})
 ngx.req.set_uri("/foo", true)
```

这个方法在 `v0.3.1rc14` 版本中首次引入。


> English Source

**syntax:** *ngx.req.set_uri(uri, jump?)*

**context:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\*

Rewrite the current request's (parsed) URI by the `uri` argument. The `uri` argument must be a Lua string and cannot be of zero length, or a Lua exception will be thrown.

The optional boolean `jump` argument can trigger location rematch (or location jump) as [ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)'s [rewrite](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite) directive, that is, when `jump` is `true` (default to `false`), this function will never return and it will tell Nginx to try re-searching locations with the new URI value at the later `post-rewrite` phase and jumping to the new location.

Location jump will not be triggered otherwise, and only the current request's URI will be modified, which is also the default behavior. This function will return but with no returned values when the `jump` argument is `false` or absent altogether.

For example, the following nginx config snippet

```nginx

 rewrite ^ /foo last;
```

can be coded in Lua like this:

```lua

 ngx.req.set_uri("/foo", true)
```

Similarly, Nginx config

```nginx

 rewrite ^ /foo break;
```

can be coded in Lua as

```lua

 ngx.req.set_uri("/foo", false)
```

or equivalently,

```lua

 ngx.req.set_uri("/foo")
```

The `jump` argument can only be set to `true` in [rewrite_by_lua](#rewrite_by_lua) and [rewrite_by_lua_file](#rewrite_by_lua_file). Use of jump in other contexts is prohibited and will throw out a Lua exception.

A more sophisticated example involving regex substitutions is as follows

```nginx

 location /test {
     rewrite_by_lua '
         local uri = ngx.re.sub(ngx.var.uri, "^/test/(.*)", "$1", "o")
         ngx.req.set_uri(uri)
     ';
     proxy_pass http://my_backend;
 }
```

which is functionally equivalent to

```nginx

 location /test {
     rewrite ^/test/(.*) /$1 break;
     proxy_pass http://my_backend;
 }
```

Note that it is not possible to use this interface to rewrite URI arguments and that [ngx.req.set_uri_args](#ngxreqset_uri_args) should be used for this instead. For instance, Nginx config

```nginx

 rewrite ^ /foo?a=3? last;
```

can be coded as

```nginx

 ngx.req.set_uri_args("a=3")
 ngx.req.set_uri("/foo", true)
```

or

```nginx

 ngx.req.set_uri_args({a = 3})
 ngx.req.set_uri("/foo", true)
```

This interface was first introduced in the `v0.3.1rc14` release.

[返回目录](#nginx-api-for-lua)
