ngx.req.get_post_args
---------------------
**语法:** *args, err = ngx.req.get_post_args(max_args?)*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;*

返回一个 Lua table，包含当前请求的所有 POST 查询参数 (MIME type 是 `application/x-www-form-urlencoded`)。使用前需要调 用 [ngx.req.read_body](#ngxreqread_body) 读取完整请求体，或通过设置 [lua_need_request_body](#lua_need_request_body) 指令为 on 以避免报错。

```nginx

 location = /test {
     content_by_lua '
         ngx.req.read_body()
         local args, err = ngx.req.get_post_args()
         if not args then
             ngx.say("failed to get post args: ", err)
             return
         end
         for key, val in pairs(args) do
             if type(val) == "table" then
                 ngx.say(key, ": ", table.concat(val, ", "))
             else
                 ngx.say(key, ": ", val)
             end
         end
     ';
 }
```

请求

```bash

 # Post request with the body 'foo=bar&bar=baz&bar=blah'
 $ curl --data 'foo=bar&bar=baz&bar=blah' localhost/test
```

将输出：

```bash

 foo: bar
 bar: baz, blah
```

多次出现同一个参数 key 时，将生成一个 Lua table，按顺序保存其所有 value。

key 和 value 将根据 URI 编码规则进行解码。

访问上面的配置文件，

```bash

 # POST request with body 'a%20b=1%61+2'
 $ curl -d 'a%20b=1%61+2' localhost/test
```

将输出：

```bash

 a b: 1a 2
```

不包含 `=<value>` 部分的参数被视为布尔值参数。例如 `POST /test` 的请求体是 `foo&bar` 时输出：

```bash

 foo: true
 bar: true
```

换句话说，它们将被赋值 Lua 布尔值 `true`。但是，它们与空字符串值参数不同，如 `POST /test` 的请求体是 `foo=&bar=` 将输出：

```bash

 foo:
 bar:
```

没有 key 的参数将被忽略。例如 `POST /test` 的请求体是 `=hello&=world` 时将没有任何输出。

请注意，为防止拒绝服务式攻击 (denial of service attacks)，默认最多解析前 100 个请求参数 (包括同名的)，更多的参数将直接忽略。

可选的 `max_args` 函数参数可以用来修改这个限制：

```lua

 local args = ngx.req.get_post_args(10)
```

这个参数可以被设置为 0 以移除此限制，此时将解析所有接收到的请求参数：

```lua

 local args = ngx.req.get_post_args(0)
```

强烈不推荐移除 `max_args` 限制。


> English Source

**syntax:** *args, err = ngx.req.get_post_args(max_args?)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;*

Returns a Lua table holding all the current request POST query arguments (of the MIME type `application/x-www-form-urlencoded`). Call [ngx.req.read_body](#ngxreqread_body) to read the request body first or turn on the [lua_need_request_body](#lua_need_request_body) directive to avoid errors.

```nginx

 location = /test {
     content_by_lua '
         ngx.req.read_body()
         local args, err = ngx.req.get_post_args()
         if not args then
             ngx.say("failed to get post args: ", err)
             return
         end
         for key, val in pairs(args) do
             if type(val) == "table" then
                 ngx.say(key, ": ", table.concat(val, ", "))
             else
                 ngx.say(key, ": ", val)
             end
         end
     ';
 }
```

Then

```bash

 # Post request with the body 'foo=bar&bar=baz&bar=blah'
 $ curl --data 'foo=bar&bar=baz&bar=blah' localhost/test
```

will yield the response body like

```bash

 foo: bar
 bar: baz, blah
```

Multiple occurrences of an argument key will result in a table value holding all of the values for that key in order.

Keys and values will be unescaped according to URI escaping rules.

With the settings above,

```bash

 # POST request with body 'a%20b=1%61+2'
 $ curl -d 'a%20b=1%61+2' localhost/test
```

will yield:

```bash

 a b: 1a 2
```

Arguments without the `=<value>` parts are treated as boolean arguments. `POST /test` with request body `foo&bar` will yield:

```bash

 foo: true
 bar: true
```

That is, they will take Lua boolean values `true`. However, they are different from arguments taking empty string values. `POST /test` with request body `foo=&bar=` will return something like

```bash

 foo:
 bar:
```

Empty key arguments are discarded. `POST /test` with body `=hello&=world` will yield empty outputs for instance.

Note that a maximum of 100 request arguments are parsed by default (including those with the same name) and that additional request arguments are silently discarded to guard against potential denial of service attacks.  

However, the optional `max_args` function argument can be used to override this limit:

```lua

 local args = ngx.req.get_post_args(10)
```

This argument can be set to zero to remove the limit and to process all request arguments received:

```lua

 local args = ngx.req.get_post_args(0)
```

Removing the `max_args` cap is strongly discouraged.

[返回目录](#nginx-api-for-lua)
