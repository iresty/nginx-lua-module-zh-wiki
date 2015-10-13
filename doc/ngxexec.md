ngx.exec
--------
**syntax:** *ngx.exec(uri, args?)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

使用 `uri`、`args` 参数执行一个内部跳转，与 [echo-nginx-module](http://github.com/openresty/echo-nginx-module) 的 [echo_exec](http://github.com/openresty/echo-nginx-module#echo_exec) 指令有些相似。

```lua

 ngx.exec('/some-location');
 ngx.exec('/some-location', 'a=3&b=5&c=6');
 ngx.exec('/some-location?a=3&b=5', 'c=6');
```

可选第二个参数 `args` 可被用来指名额外的 URI 查询参数，例如：

```lua

 ngx.exec("/foo", "a=3&b=hello%20world")
```

另外，对于 `args` 参数可使用一个 Lua 表，内部通过 ngx_lua 完成 URI 转义和字符串的连接。

```lua

 ngx.exec("/foo", { a = 3, b = "hello world" })
```

该结果和上一个示例是一样的。

作为 `args` 参数的 Lua 表格式化结果，与使用 [ngx.encode_args](#ngxencode_args) 方法格式化结果是相同的。

命名 location 也是支持的，但是第二个 `args` 参数将被忽略（如果存在的话），并且对新目标地址的查询字符串会从引用 location （如果有）中继承。

下面的例子中， `GET /foo/file.php?a=hello` 将返回 "hello" ，而不是 "goodbye" ：

```nginx

 location /foo {
     content_by_lua '
         ngx.exec("@bar", "a=goodbye");
     ';
 }

 location @bar {
     content_by_lua '
         local args = ngx.req.get_uri_args()
         for key, val in pairs(args) do
             if key == "a" then
                 ngx.say(val)
             end
         end
     ';
 }
```

注意，`ngx.exec` 方法与 [ngx.redirect](#ngxredirect) 是完全不同的，前者是个纯粹的内部跳转并且没有引入任何额外 HTTP 信号。

注意，此方法的调用终止当前请求的处理，并且它 *必须* 在 [ngx.send_headers](#ngxsend_headers) 或明确有响应体应答（比如 [ngx.print](#ngxprint) 或 [ngx.say](#ngxsay) 之一）之前调用。

该方法调用与 `return` 语句联合使用，是推荐的代码样式，例如，通过`return ngx.redirect(...)`，可使用在 [header_filter_by_lua](#header_filter_by_lua) 之外的环境中加强处理该请求被终止。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.exec
--------
**syntax:** *ngx.exec(uri, args?)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua**

Does an internal redirect to `uri` with `args` and is similar to the [echo_exec](http://github.com/openresty/echo-nginx-module#echo_exec) directive of the [echo-nginx-module](http://github.com/openresty/echo-nginx-module).

```lua

 ngx.exec('/some-location');
 ngx.exec('/some-location', 'a=3&b=5&c=6');
 ngx.exec('/some-location?a=3&b=5', 'c=6');
```

The optional second `args` can be used to specify extra URI query arguments, for example:

```lua

 ngx.exec("/foo", "a=3&b=hello%20world")
```

Alternatively, a Lua table can be passed for the `args` argument for ngx_lua to carry out URI escaping and string concatenation.

```lua

 ngx.exec("/foo", { a = 3, b = "hello world" })
```

The result is exactly the same as the previous example.

The format for the Lua table passed as the `args` argument is identical to the format used in the [ngx.encode_args](#ngxencode_args) method.

Named locations are also supported but the second `args` argument will be ignored if present and the querystring for the new target is inherited from the referring location (if any).

`GET /foo/file.php?a=hello` will return "hello" and not "goodbye" in the example below

```nginx

 location /foo {
     content_by_lua '
         ngx.exec("@bar", "a=goodbye");
     ';
 }

 location @bar {
     content_by_lua '
         local args = ngx.req.get_uri_args()
         for key, val in pairs(args) do
             if key == "a" then
                 ngx.say(val)
             end
         end
     ';
 }
```

Note that the `ngx.exec` method is different from [ngx.redirect](#ngxredirect) in that
it is purely an internal redirect and that no new external HTTP traffic is involved.

Also note that this method call terminates the processing of the current request and that it *must* be called before [ngx.send_headers](#ngxsend_headers) or explicit response body
outputs by either [ngx.print](#ngxprint) or [ngx.say](#ngxsay).

It is recommended that a coding style that combines this method call with the `return` statement, i.e., `return ngx.exec(...)` be adopted when this method call is used in contexts other than [header_filter_by_lua](#header_filter_by_lua) to reinforce the fact that the request processing is being terminated.

[Back to TOC](#nginx-api-for-lua)
