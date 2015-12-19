ngx.var.VARIABLE
----------------
**语法:** *ngx.var.VAR_NAME*

**环境:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\*

读写 Nginx 变量值。

```nginx

 value = ngx.var.some_nginx_variable_name
 ngx.var.some_nginx_variable_name = value
```

请注意，只有已经定义的 nginx 变量可以被写入。例如，

```nginx

 location /foo {
     set $my_var ''; # 需要在设置时创建 $my_var 变量
     content_by_lua '
         ngx.var.my_var = 123;
         ...
     ';
 }
```

也就是说，nginx 变量无法“随用随创建”。

一些特殊的 nginx 变量，比如 `$args` 和 `$limit_rate`，可以被赋值，但许多其他的变量不能，包括 `$query_string`，`$arg_PARAMETER`，和 `$http_NAME` 等。

Nginx 正则表达式捕获组变量 `$1`、`$2`、`$3` 等，也可以通过这个界面读取，方式为通过 `ngx.var[1]`、`ngx.var[2]`、`ngx.var[3]` 等。

设置 `ngx.var.Foo` 为 `nil` 值将删除 Nginx 变量 `$Foo`。

```lua

 ngx.var.args = nil
```

**注意** 当从 Nginx 变量中读取值时，Nginx 将从基于请求的内存池中分配内存，只有在请求中止时才释放。所以如果用户的 Lua 代码中需要反复读取 Nginx 变量，请在用户程序的 Lua 变量中缓存，例如，

```lua

 local val = ngx.var.some_var
 --- 在后面反复使用变量 val
```

以避免在当前请求周期内的 (临时) 内存泄露。另外一个缓存结果的方法是使用 [ngx.ctx](#ngxctx) 表。

未定义的 NGINX 变量会被认定为 `nil` ，而未初始化（但已定义）的 NGINX 变量会被认定为空 Lua 字符串。

这个 API 需要进行相对“昂贵”的元方法调用，所以请避免高频使用。


> English Source

**syntax:** *ngx.var.VAR_NAME*

**context:** *set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\*

Read and write Nginx variable values.

```nginx

 value = ngx.var.some_nginx_variable_name
 ngx.var.some_nginx_variable_name = value
```

Note that only already defined nginx variables can be written to.
For example:

```nginx

 location /foo {
     set $my_var ''; # this line is required to create $my_var at config time
     content_by_lua '
         ngx.var.my_var = 123;
         ...
     ';
 }
```

That is, nginx variables cannot be created on-the-fly.

Some special nginx variables like `$args` and `$limit_rate` can be assigned a value,
many others are not, like `$query_string`, `$arg_PARAMETER`, and `$http_NAME`.

Nginx regex group capturing variables `$1`, `$2`, `$3`, and etc, can be read by this
interface as well, by writing `ngx.var[1]`, `ngx.var[2]`, `ngx.var[3]`, and etc.

Setting `ngx.var.Foo` to a `nil` value will unset the `$Foo` Nginx variable. 

```lua

 ngx.var.args = nil
```

**WARNING** When reading from an Nginx variable, Nginx will allocate memory in the per-request memory pool which is freed only at request termination. So when you need to read from an Nginx variable repeatedly in your Lua code, cache the Nginx variable value to your own Lua variable, for example,

```lua

 local val = ngx.var.some_var
 --- use the val repeatedly later
```

to prevent (temporary) memory leaking within the current request's lifetime. Another way of caching the result is to use the [ngx.ctx](#ngxctx) table.

Undefined NGINX variables are evaluated to `nil` while uninitialized (but defined) NGINX variables are evaluated to an empty Lua string.

This API requires a relatively expensive metamethod call and it is recommended to avoid using it on hot code paths.

[返回目录](#nginx-api-for-lua)