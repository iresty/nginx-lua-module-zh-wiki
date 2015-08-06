**语法:** *from, to, err = ngx.re.find(subject, regex, options?, ctx?, nth?)*

**环境:** *init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

与 [ngx.re.match](#ngxrematch) 类似但只返回匹配结果子字符串的开始索引 (`from`) 和结束索引 (`to`)。返回的索引值是基于 1 的，可以直接被用于 Lua 的 [string.sub](http://www.lua.org/manual/5.1/manual.html#pdf-string.sub) API 函数来获取匹配结果子串。

当出现任何错误时 (例如错误的正则表达式或任何 PCRE 运行时错误)，这个 API 函数返回两个 `nil` 以及一个描述错误的的字符串。

如果匹配不成功，此函数返回一个 `nil` 值。

下面是一个示例：

```lua

 local s = "hello, 1234"
 local from, to, err = ngx.re.find(s, "([0-9]+)", "jo")
 if from then
     ngx.say("from: ", from)
     ngx.say("to: ", to)
     ngx.say("matched: ", string.sub(s, from, to))
 else
     if err then
         ngx.say("error: ", err)
         return
     end
     ngx.say("not matched!")
 end
```

此示例将输出

    from: 8
    to: 11
    matched: 1234


因为此 API 函数并不创建任何新 Lua 字符串或 Lua 表，运行速度大大快于 [ngx.re.match](#ngxrematch)。所以如果可能请尽量使用本函数。

自 `0.9.3` 版本开始，添加第 5 个可选参数 `nth`，用来指定第几个子匹配结果索引被返回。当 `nth` 为 0 (默认值) 时，返回完整匹配子串索引；当 `nth` 为 1 时，第一个(括号内的)子匹配结果索引被返回； 当 `nth` 为 2 时，第二个子匹配结果索引被返回，以此类推。当被指定的子匹配没有结果时，返回 `nil`。下面是一个例子：

```lua

 local str = "hello, 1234"
 local from, to = ngx.re.find(str, "([0-9])([0-9]+)", "jo", nil, 2)
 if from then
     ngx.say("matched 2nd submatch: ", string.sub(str, from, to))  -- yields "234"
 end
```

此 API 函数自 `v0.9.2` 版开始提供。



> English source:

**syntax:** *from, to, err = ngx.re.find(subject, regex, options?, ctx?, nth?)*

**context:** *init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

Similar to [ngx.re.match](#ngxrematch) but only returns the begining index (`from`) and end index (`to`) of the matched substring. The returned indexes are 1-based and can be fed directly into the [string.sub](http://www.lua.org/manual/5.1/manual.html#pdf-string.sub) API function to obtain the matched substring.

In case of errors (like bad regexes or any PCRE runtime errors), this API function returns two `nil` values followed by a string describing the error.

If no match is found, this function just returns a `nil` value.

Below is an example:

```lua

 local s = "hello, 1234"
 local from, to, err = ngx.re.find(s, "([0-9]+)", "jo")
 if from then
     ngx.say("from: ", from)
     ngx.say("to: ", to)
     ngx.say("matched: ", string.sub(s, from, to))
 else
     if err then
         ngx.say("error: ", err)
         return
     end
     ngx.say("not matched!")
 end
```

This example produces the output

    from: 8
    to: 11
    matched: 1234

Because this API function does not create new Lua strings nor new Lua tables, it is much faster than [ngx.re.match](#ngxrematch). It should be used wherever possible.

Since the `0.9.3` release, an optional 5th argument, `nth`, is supported to specify which (submatch) capture's indexes to return. When `nth` is 0 (which is the default), the indexes for the whole matched substring is returned; when `nth` is 1, then the 1st submatch capture's indexes are returned; when `nth` is 2, then the 2nd submatch capture is returned, and so on. When the specified submatch does not have a match, then two `nil` values will be returned. Below is an example for this:

```lua

 local str = "hello, 1234"
 local from, to = ngx.re.find(str, "([0-9])([0-9]+)", "jo", nil, 2)
 if from then
     ngx.say("matched 2nd submatch: ", string.sub(str, from, to))  -- yields "234"
 end
```

This API function was first introduced in the `v0.9.2` release.
