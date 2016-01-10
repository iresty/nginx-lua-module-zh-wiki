ngx.arg
-------
**语法:** *val = ngx.arg\[index\]*

**环境:** *set_by_lua&#42;, body_filter_by_lua&#42;*

当被用在 [set_by_lua](#set_by_lua) 或 [set_by_lua_file](#set_by_lua_file) 指令环境中时，本表是一个只读表，包含输入参数供配置命令使用：

```lua

 value = ngx.arg[n]
```

例如，

```nginx

 location /foo {
     set $a 32;
     set $b 56;

     set_by_lua $sum
         'return tonumber(ngx.arg[1]) + tonumber(ngx.arg[2])'
         $a $b;

     echo $sum;
 }
```

将输出 `88`，是 `32` 和 `56` 的和。

当被用在 [body_filter_by_lua](#body_filter_by_lua) 或 [body_filter_by_lua_file](#body_filter_by_lua_file) 指令环境中时，本表第一个元素是送给输出过滤器的输入数据块，第二个元素是 "eof" 布尔标记，用以标识整个输出数据流是否结束。

可以通过直接给相应的表元素赋值，设置送给下游 Nginx 输出过滤器的数据块和 "eof" 标记。当给 `ngx.arg[1]` 赋值 `nil` 或 Lua 空字符串时，将不发送任何数据给下游的 Nginx 输出过滤器。


> English Source

**syntax:** *val = ngx.arg\[index\]*

**context:** *set_by_lua&#42;, body_filter_by_lua&#42;*

When this is used in the context of the [set_by_lua](#set_by_lua) or [set_by_lua_file](#set_by_lua_file) directives, this table is read-only and holds the input arguments to the config directives:

```lua

 value = ngx.arg[n]
```

Here is an example

```nginx

 location /foo {
     set $a 32;
     set $b 56;

     set_by_lua $sum
         'return tonumber(ngx.arg[1]) + tonumber(ngx.arg[2])'
         $a $b;

     echo $sum;
 }
```

that writes out `88`, the sum of `32` and `56`.

When this table is used in the context of [body_filter_by_lua](#body_filter_by_lua) or [body_filter_by_lua_file](#body_filter_by_lua_file), the first element holds the input data chunk to the output filter code and the second element holds the boolean flag for the "eof" flag indicating the end of the whole output data stream.

The data chunk and "eof" flag passed to the downstream Nginx output filters can also be overridden by assigning values directly to the corresponding table elements. When setting `nil` or an empty Lua string value to `ngx.arg[1]`, no data chunk will be passed to the downstream Nginx output filters at all.


[返回目录](#nginx-api-for-lua)