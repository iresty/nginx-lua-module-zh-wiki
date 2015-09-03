lua_regex_cache_max_entries
---------------------------
**语法:** *lua_regex_cache_max_entries &lt;num&gt;*

**默认:** *lua_regex_cache_max_entries 1024*

**作用域:** *http*

在工作进程级别，指定正则表达式编译缓存允许的最大数目。

正则表达式被用于[ngx.re.match](#ngxrematch)， [ngx.re.gmatch](#ngxregmatch)， [ngx.re.sub](#ngxresub)， 和 [ngx.re.gsub](#ngxregsub)，如果使用`o` (既，编译一次的标识)正则选项，将会被缓存。

允许的默认数量为1024，当达到此限制，新的正则表达式将不会被缓存（就像没指定`o`选项一样），将会有且仅只有一个告警信息在 `error.log` 文件中：

    2011/08/27 23:18:26 [warn] 31997#0: *1 lua exceeding regex cache max entries (1024), ...

对于部分正则表达式（字符串的各种替换，如 [ngx.re.sub](#ngxresub) 和 [ngx.re.gsub](#ngxregsub)），不要使用`o`选项，这类正则每次都不一样，缓存无法被利用。这样我们可以避免撞上最大数的限制。

[Back to TOC](#directives)

> English source:

lua_regex_cache_max_entries
---------------------------
**syntax:** *lua_regex_cache_max_entries &lt;num&gt;*

**default:** *lua_regex_cache_max_entries 1024*

**context:** *http*

Specifies the maximum number of entries allowed in the worker process level compiled regex cache.

The regular expressions used in [ngx.re.match](#ngxrematch), [ngx.re.gmatch](#ngxregmatch), [ngx.re.sub](#ngxresub), and [ngx.re.gsub](#ngxregsub) will be cached within this cache if the regex option `o` (i.e., compile-once flag) is specified.

The default number of entries allowed is 1024 and when this limit is reached, new regular expressions will not be cached (as if the `o` option was not specified) and there will be one, and only one, warning in the `error.log` file:


    2011/08/27 23:18:26 [warn] 31997#0: *1 lua exceeding regex cache max entries (1024), ...


Do not activate the `o` option for regular expressions (and/or `replace` string arguments for [ngx.re.sub](#ngxresub) and [ngx.re.gsub](#ngxregsub)) that are generated *on the fly* and give rise to infinite variations to avoid hitting the specified limit.

[Back to TOC](#directives)