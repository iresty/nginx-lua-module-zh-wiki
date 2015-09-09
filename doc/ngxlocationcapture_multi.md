ngx.location.capture_multi
--------------------------
**语法:** *res1, res2, ... = ngx.location.capture_multi({ {uri, options?}, {uri, options?}, ... })*

**环境:** *rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

与 [ngx.location.capture](#ngxlocationcapture) 类似，并允许多个子请求并行访问。

此函数根据输入列表并行发起多个子请求，并按原顺序返回结果。例如，

```lua

 res1, res2, res3 = ngx.location.capture_multi{
     { "/foo", { args = "a=3&b=4" } },
     { "/bar" },
     { "/baz", { method = ngx.HTTP_POST, body = "hello" } },
 }

 if res1.status == ngx.HTTP_OK then
     ...
 end

 if res2.body == "BLAH" then
     ...
 end
```

此函数只有在所有子请求中止后才返回。
总等待时间是耗时最长一个子请求的时间，而不是所有子请求等待时间的总和。

当子请求数量预先未知时，可以通过 Lua 表来存储所有请求和返回。

```lua

 -- 创建请求表
 local reqs = {}
 table.insert(reqs, { "/mysql" })
 table.insert(reqs, { "/postgres" })
 table.insert(reqs, { "/redis" })
 table.insert(reqs, { "/memcached" })

 -- 同时执行所有请求，并等待所有返回
 local resps = { ngx.location.capture_multi(reqs) }

 -- 循环返回表
 for i, resp in ipairs(resps) do
     -- 处理返回表 "resp"
 end
```

[ngx.location.capture](#ngxlocationcapture) 函数是本函数的特殊形式。逻辑上说，[ngx.location.capture](#ngxlocationcapture) 函数可以按下面方法实现：

```lua

 ngx.location.capture =
     function (uri, args)
         return ngx.location.capture_multi({ {uri, args} })
     end
```

请参考 [subrequest directives of other modules](#locations-configured-by-subrequest-directives-of-other-modules) 了解目标 location 的配置限制。


> English Source

**syntax:** *res1, res2, ... = ngx.location.capture_multi({ {uri, options?}, {uri, options?}, ... })*

**context:** *rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\*

Just like [ngx.location.capture](#ngxlocationcapture), but supports multiple subrequests running in parallel.

This function issues several parallel subrequests specified by the input table and returns their results in the same order. For example,

```lua

 res1, res2, res3 = ngx.location.capture_multi{
     { "/foo", { args = "a=3&b=4" } },
     { "/bar" },
     { "/baz", { method = ngx.HTTP_POST, body = "hello" } },
 }

 if res1.status == ngx.HTTP_OK then
     ...
 end

 if res2.body == "BLAH" then
     ...
 end
```

This function will not return until all the subrequests terminate.
The total latency is the longest latency of the individual subrequests rather than the sum.

Lua tables can be used for both requests and responses when the number of subrequests to be issued is not known in advance:

```lua

 -- construct the requests table
 local reqs = {}
 table.insert(reqs, { "/mysql" })
 table.insert(reqs, { "/postgres" })
 table.insert(reqs, { "/redis" })
 table.insert(reqs, { "/memcached" })

 -- issue all the requests at once and wait until they all return
 local resps = { ngx.location.capture_multi(reqs) }

 -- loop over the responses table
 for i, resp in ipairs(resps) do
     -- process the response table "resp"
 end
```

The [ngx.location.capture](#ngxlocationcapture) function is just a special form
of this function. Logically speaking, the [ngx.location.capture](#ngxlocationcapture) can be implemented like this

```lua

 ngx.location.capture =
     function (uri, args)
         return ngx.location.capture_multi({ {uri, args} })
     end
```

Please also refer to restrictions on capturing locations configured by [subrequest directives of other modules](#locations-configured-by-subrequest-directives-of-other-modules).

[返回目录](#nginx-api-for-lua)