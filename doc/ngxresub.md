ngx.re.sub
----------
**语法:** *newstr, n, err = ngx.re.sub(subject, regex, replace, options?)*

**环境:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

使用 Perl 兼容正则表达式 `regex` 匹配字符串 `subject`，将第一个结果替换为字符串或函数类型参数 `replace`。可选参数 `options` 含义与 [ngx.re.match](#ngxrematch) 相同。

这个方法返回结果字符串以及成功替换的数量。当发生失败时，例如正则表达式或 `<replace>` 字符串参数语法错，将返回 `nil` 以及一个描述错误的字符串。

当 `replace` 是一个字符串时，它们被视为一个特殊字符串替换模板。例如：

```lua

 local newstr, n, err = ngx.re.sub("hello, 1234", "([0-9])[0-9]", "[$0][$1]")
 if newstr then
     -- newstr == "hello, [12][1]34"
     -- n == 1
 else
     ngx.log(ngx.ERR, "error: ", err)
     return
 end
```

其中 `$0` 指模板匹配的完整子字符串，`$1` 指第一个括号内匹配的子串。

花括号可以被用来从背景字符串中消除变量名歧义。

```lua

 local newstr, n, err = ngx.re.sub("hello, 1234", "[0-9]", "${0}00")
     -- newstr == "hello, 100234"
     -- n == 1
```

要在 `replace` 中使用美元字符(`$`)，可以使用另外一个该符号作为转义，例如，

```lua

 local newstr, n, err = ngx.re.sub("hello, 1234", "[0-9]", "$$")
     -- newstr == "hello, $234"
     -- n == 1
```

不要使用反斜线转义美元字符；它不会象你想象的那样工作。

当 `replace` 参数是一个 "函数" 时，它将被通过参数 "匹配表" 调用，用来生成替换字符串。被送入 `replace` 函数的 "匹配表" 与 [ngx.re.match](#ngxrematch) 的返回值相同。例如：

```lua

 local func = function (m)
     return "[" .. m[0] .. "][" .. m[1] .. "]"
 end
 local newstr, n, err = ngx.re.sub("hello, 1234", "( [0-9] ) [0-9]", func, "x")
     -- newstr == "hello, [12][1]34"
     -- n == 1
```

在 `replace` 函数返回值中的美元字符没有任何特殊含义。

这个方法需要在 Nginx 中启用 PCRE 库。 ([Known Issue With Special Escaping Sequences](#special-escaping-sequences)).

这个功能最早出现在 `v0.2.1rc13` 版本中。



> English Source

**syntax:** *newstr, n, err = ngx.re.sub(subject, regex, replace, options?)*

**context:** *init_worker_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Substitutes the first match of the Perl compatible regular expression `regex` on the `subject` argument string with the string or function argument `replace`. The optional `options` argument has exactly the same meaning as in [ngx.re.match](#ngxrematch).

This method returns the resulting new string as well as the number of successful substitutions. In case of failures, like syntax errors in the regular expressions or the `<replace>` string argument, it will return `nil` and a string describing the error.

When the `replace` is a string, then it is treated as a special template for string replacement. For example,

```lua

 local newstr, n, err = ngx.re.sub("hello, 1234", "([0-9])[0-9]", "[$0][$1]")
 if newstr then
     -- newstr == "hello, [12][1]34"
     -- n == 1
 else
     ngx.log(ngx.ERR, "error: ", err)
     return
 end
```

where `$0` referring to the whole substring matched by the pattern and `$1` referring to the first parenthesized capturing substring.

Curly braces can also be used to disambiguate variable names from the background string literals: 

```lua

 local newstr, n, err = ngx.re.sub("hello, 1234", "[0-9]", "${0}00")
     -- newstr == "hello, 100234"
     -- n == 1
```

Literal dollar sign characters (`$`) in the `replace` string argument can be escaped by another dollar sign, for instance,

```lua

 local newstr, n, err = ngx.re.sub("hello, 1234", "[0-9]", "$$")
     -- newstr == "hello, $234"
     -- n == 1
```

Do not use backlashes to escape dollar signs; it will not work as expected.

When the `replace` argument is of type "function", then it will be invoked with the "match table" as the argument to generate the replace string literal for substitution. The "match table" fed into the `replace` function is exactly the same as the return value of [ngx.re.match](#ngxrematch). Here is an example:

```lua

 local func = function (m)
     return "[" .. m[0] .. "][" .. m[1] .. "]"
 end
 local newstr, n, err = ngx.re.sub("hello, 1234", "( [0-9] ) [0-9]", func, "x")
     -- newstr == "hello, [12][1]34"
     -- n == 1
```

The dollar sign characters in the return value of the `replace` function argument are not special at all.

This method requires the PCRE library enabled in Nginx.  ([Known Issue With Special Escaping Sequences](#special-escaping-sequences)).

This feature was first introduced in the `v0.2.1rc13` release.

[返回目录](#nginx-api-for-lua)