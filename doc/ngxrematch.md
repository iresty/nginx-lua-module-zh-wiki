ngx.re.match
------------
**语法:** *captures, err = ngx.re.match(subject, regex, options?, ctx?, res_table?)*

**环境:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

使用 Perl 兼容正则表达式 `regex` 匹配字符串 `subject`，并使用可选的参数 `options` 作为正则表达式选项。

仅返回第一个匹配结果，无结果时返回 `nil`。当出错时，例如正则表达式出错或者超出 PCRE 堆栈限制，将返回 `nil` 以及一个描述错误的字符串。

当匹配成功时，返回一个 Lua 表 `captures`，其中 `captures[0]` 存储(整个模板)匹配出的完整子字符串，`captures[1]` 存储第一个括号内的子模板匹配结果，`captures[2]` 存储第二个，以此类推。

```lua

 local m, err = ngx.re.match("hello, 1234", "[0-9]+")
 if m then
     -- m[0] == "1234"

 else
     if err then
         ngx.log(ngx.ERR, "error: ", err)
         return
     end

     ngx.say("match not found")
 end
```

```lua

 local m, err = ngx.re.match("hello, 1234", "([0-9])[0-9]+")
 -- m[0] == "1234"
 -- m[1] == "1"
```


自 `v0.7.14` 版本后，本模块支持正则表达式命名捕获(Named capture)，结果以键值对的方式与数字编号的结果在同一个 Lua 表中返回。

```lua

 local m, err = ngx.re.match("hello, 1234", "([0-9])(?<remaining>[0-9]+)")
 -- m[0] == "1234"
 -- m[1] == "1"
 -- m[2] == "234"
 -- m["remaining"] == "234"
```

在 `captures` 表中，不匹配的子模板将返回 `nil` 值。

```lua

 local m, err = ngx.re.match("hello, world", "(world)|(hello)|(?<named>howdy)")
 -- m[0] == "hello"
 -- m[1] == nil
 -- m[2] == "hello"
 -- m[3] == nil
 -- m["named"] == nil
```

通过指定 `options` (选项)来控制匹配操作的执行方式。支持以下选项字符。


    a             锚定模式 (仅从目标字符串开始位置匹配)

    d             启用 DFA 模式(又名最长令牌匹配语义)。
                  此选项需要 PCRE 6.0 以上版本，否则将抛出 Lua 异常。
                  此选项最早出现在 ngx_lua v0.3.1rc30 版本中。

    D             启用重复命名模板支持。子模板命名可以重复，在结果中以数组方式返回。例如：
                    local m = ngx.re.match("hello, world",
                                           "(?<named>\w+), (?<named>\w+)",
                                           "D")
                    -- m["named"] == {"hello", "world"}
                  此选项最早出现在 v0.7.14 版本中，需要 PCRE 8.12 以上版本支持.

    i             大小写不敏感模式 (类似 Perl 的 /i 修饰符)

    j             启用 PCRE JIT 编译，此功能需要 PCRE 8.21 以上版本以 --enable-jit 选项编译。
                  为达到最佳性能，此选项应与 'o' 选项同时使用。
                  此选项最早出现在 ngx_lua v0.3.1rc30 版本中。

    J             启用 PCRE Javascript 兼容模式。
                  此选项最早出现在 v0.7.14 版本中，需要 PCRE 8.12 以上版本支持.

    m             多行模式 (类似 Perl 的 /m 修饰符)

    o             仅编译一次模式 (类似 Perl 的 /o 修饰符)
                  启用 worker 进程级正则表达式编译缓存。

    s             单行模式 (类似 Perl 的 /s 修饰符)

    u             UTF-8 模式。此选项需要 PCRE 以 --enable-utf8 选项编译，否则将抛出 Lua 异常。

    U             类似 "u" 模式，但禁用了 PCRE 对目标字符串的 UTF-8 合法性检查。
                  此选项最早出现在 ngx_lua v0.8.1 版本中。

    x             扩展模式 (类似 Perl 的 /x 修饰符)


这些选项可以组合使用：

```nginx

 local m, err = ngx.re.match("hello, world", "HEL LO", "ix")
 -- m[0] == "hello"
```

```nginx

 local m, err = ngx.re.match("hello, 美好生活", "HELLO, (.{2})", "iu")
 -- m[0] == "hello, 美好"
 -- m[1] == "美好"
```

在优化性能时，`o` 选项非常有用，因为正则表达式模板将仅仅被编译一次，之后缓存在 worker 级的缓存中，并被此 nginx worker 处理的所有请求共享。缓存数量上限可以通过 [lua_regex_cache_max_entries](#lua_regex_cache_max_entries) 指令调整。


可选的第四个参数 `ctx` 是一个 Lua 表，包含可选的 `pos` 域。当 `ctx` 表的 `pos` 域有值时，`ngx.re.match` 将从该位置起执行匹配(位置下标从 1 开始)。不论 `ctx` 表中是否已经有 `pos` 域，`ngx.re.match` 将在正则表达式被成功匹配后，设置 `pos` 域值为完整匹配子字符串 *之后* 的位置。当匹配失败时，`ctx` 表将保持不变。

```lua

 local ctx = {}
 local m, err = ngx.re.match("1234, hello", "[0-9]+", "", ctx)
      -- m[0] = "1234"
      -- ctx.pos == 5
```

```lua

 local ctx = { pos = 2 }
 local m, err = ngx.re.match("1234, hello", "[0-9]+", "", ctx)
      -- m[0] = "34"
      -- ctx.pos == 5
```

参数 `ctx` 表与正则表达式修饰符 `a` 组合使用，可以用来建立一个基于 `ngx.re.match` 的词法分析器。

注意，当指定参数 `ctx` 时，参数 `options` 不能空缺，当不需要使用 `options` 来指定正则表达式选项时，必须使用 Lua 空字符串 (`""`) 作为占位符。

这个方法需要在 Nginx 中启用 PCRE 库。 ([Known Issue With Special Escaping Sequences](#special-escaping-sequences)).

要想确认 PCRE JIT 是否已经启用，需要在 Nginx 或 ngx_openresty 的 `./configure` 配置脚本中，添加 `--with-debug` 选项激活 Nginx 的调试日志。然后，在 `error_log` 指令中启用 `error` 错误日志级别。当 PCRE JIT 启用时，将出现下述信息：


    pcre JIT compiling result: 1


自 `0.9.4` 版本开始，此函数接受第五个参数，`res_table`，让调用者可以自己指定存储所有匹配结果的 Lua 表。自 `0.9.6` 版本开始，调用者需要自己确保这个表是空的。这个功能对表预分配、重用以及节省 Lua 回收机制 (GC) 非常有用。

这个功能最早出现在 `v0.2.1rc11` 版本中。



> English Source

**syntax:** *captures, err = ngx.re.match(subject, regex, options?, ctx?, res_table?)*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Matches the `subject` string using the Perl compatible regular expression `regex` with the optional `options`.

Only the first occurrence of the match is returned, or `nil` if no match is found. In case of errors, like seeing a bad regular expression or exceeding the PCRE stack limit, `nil` and a string describing the error will be returned.

When a match is found, a Lua table `captures` is returned, where `captures[0]` holds the whole substring being matched, and `captures[1]` holds the first parenthesized sub-pattern's capturing, `captures[2]` the second, and so on.

```lua

 local m, err = ngx.re.match("hello, 1234", "[0-9]+")
 if m then
     -- m[0] == "1234"

 else
     if err then
         ngx.log(ngx.ERR, "error: ", err)
         return
     end

     ngx.say("match not found")
 end
```

```lua

 local m, err = ngx.re.match("hello, 1234", "([0-9])[0-9]+")
 -- m[0] == "1234"
 -- m[1] == "1"
```

Named captures are also supported since the `v0.7.14` release
and are returned in the same Lua table as key-value pairs as the numbered captures.

```lua

 local m, err = ngx.re.match("hello, 1234", "([0-9])(?<remaining>[0-9]+)")
 -- m[0] == "1234"
 -- m[1] == "1"
 -- m[2] == "234"
 -- m["remaining"] == "234"
```

Unmatched subpatterns will have `nil` values in their `captures` table fields.

```lua

 local m, err = ngx.re.match("hello, world", "(world)|(hello)|(?<named>howdy)")
 -- m[0] == "hello"
 -- m[1] == nil
 -- m[2] == "hello"
 -- m[3] == nil
 -- m["named"] == nil
```

Specify `options` to control how the match operation will be performed. The following option characters are supported:


    a             anchored mode (only match from the beginning)

    d             enable the DFA mode (or the longest token match semantics).
                  this requires PCRE 6.0+ or else a Lua exception will be thrown.
                  first introduced in ngx_lua v0.3.1rc30.

    D             enable duplicate named pattern support. This allows named
                  subpattern names to be repeated, returning the captures in
                  an array-like Lua table. for example,
                    local m = ngx.re.match("hello, world",
                                           "(?<named>\w+), (?<named>\w+)",
                                           "D")
                    -- m["named"] == {"hello", "world"}
                  this option was first introduced in the v0.7.14 release.
                  this option requires at least PCRE 8.12.

    i             case insensitive mode (similar to Perl's /i modifier)

    j             enable PCRE JIT compilation, this requires PCRE 8.21+ which
                  must be built with the --enable-jit option. for optimum performance,
                  this option should always be used together with the 'o' option.
                  first introduced in ngx_lua v0.3.1rc30.

    J             enable the PCRE Javascript compatible mode. this option was
                  first introduced in the v0.7.14 release. this option requires
                  at least PCRE 8.12.

    m             multi-line mode (similar to Perl's /m modifier)

    o             compile-once mode (similar to Perl's /o modifier),
                  to enable the worker-process-level compiled-regex cache

    s             single-line mode (similar to Perl's /s modifier)

    u             UTF-8 mode. this requires PCRE to be built with
                  the --enable-utf8 option or else a Lua exception will be thrown.

    U             similar to "u" but disables PCRE's UTF-8 validity check on
                  the subject string. first introduced in ngx_lua v0.8.1.

    x             extended mode (similar to Perl's /x modifier)


These options can be combined:

```nginx

 local m, err = ngx.re.match("hello, world", "HEL LO", "ix")
 -- m[0] == "hello"
```

```nginx

 local m, err = ngx.re.match("hello, 美好生活", "HELLO, (.{2})", "iu")
 -- m[0] == "hello, 美好"
 -- m[1] == "美好"
```

The `o` option is useful for performance tuning, because the regex pattern in question will only be compiled once, cached in the worker-process level, and shared among all requests in the current Nginx worker process. The upper limit of the regex cache can be tuned via the [lua_regex_cache_max_entries](#lua_regex_cache_max_entries) directive.

The optional fourth argument, `ctx`, can be a Lua table holding an optional `pos` field. When the `pos` field in the `ctx` table argument is specified, `ngx.re.match` will start matching from that offset (starting from 1). Regardless of the presence of the `pos` field in the `ctx` table, `ngx.re.match` will always set this `pos` field to the position *after* the substring matched by the whole pattern in case of a successful match. When match fails, the `ctx` table will be left intact.

```lua

 local ctx = {}
 local m, err = ngx.re.match("1234, hello", "[0-9]+", "", ctx)
      -- m[0] = "1234"
      -- ctx.pos == 5
```

```lua

 local ctx = { pos = 2 }
 local m, err = ngx.re.match("1234, hello", "[0-9]+", "", ctx)
      -- m[0] = "34"
      -- ctx.pos == 5
```

The `ctx` table argument combined with the `a` regex modifier can be used to construct a lexer atop `ngx.re.match`.

Note that, the `options` argument is not optional when the `ctx` argument is specified and that the empty Lua string (`""`) must be used as placeholder for `options` if no meaningful regex options are required.

This method requires the PCRE library enabled in Nginx.  ([Known Issue With Special Escaping Sequences](#special-escaping-sequences)).

To confirm that PCRE JIT is enabled, activate the Nginx debug log by adding the `--with-debug` option to Nginx or ngx_openresty's `./configure` script. Then, enable the "debug" error log level in `error_log` directive. The following message will be generated if PCRE JIT is enabled:


    pcre JIT compiling result: 1


Starting from the `0.9.4` release, this function also accepts a 5th argument, `res_table`, for letting the caller supply the Lua table used to hold all the capturing results. Starting from `0.9.6`, it is the caller's responsibility to ensure this table is empty. This is very useful for recycling Lua tables and saving GC and table allocation overhead.

This feature was introduced in the `v0.2.1rc11` release.

[返回目录](#nginx-api-for-lua)