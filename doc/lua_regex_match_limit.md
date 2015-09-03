lua_regex_match_limit
---------------------
**语法:** *lua_regex_match_limit &lt;num&gt;*

**默认:** *lua_regex_match_limit 0*

**作用域:** *http*

指定执行[ngx.re API](#ngxrematch)时使用PCRE库的"匹配限制"。引述PCRE手册，“the limit ... has the effect of limiting the amount of backtracking that can take place.”。

当触发了这个限制，在Lua代码的[ngx.re API](#ngxrematch)函数，将返回错误信息"pcre_exec() failed: -8"。

当设置了限制为0，将使用编译PCRE库的默认"match limit"。这也是这个配置的默认值。

这个配置是在`v0.8.5`发行版被首次引入的。

[Back to TOC](#directives)

> English source:

lua_regex_match_limit
---------------------
**syntax:** *lua_regex_match_limit &lt;num&gt;*

**default:** *lua_regex_match_limit 0*

**context:** *http*

Specifies the "match limit" used by the PCRE library when executing the [ngx.re API](#ngxrematch). To quote the PCRE manpage, "the limit ... has the effect of limiting the amount of backtracking that can take place."

When the limit is hit, the error string "pcre_exec() failed: -8" will be returned by the [ngx.re API](#ngxrematch) functions on the Lua land.

When setting the limit to 0, the default "match limit" when compiling the PCRE library is used. And this is the default value of this directive.

This directive was first introduced in the `v0.8.5` release.

[Back to TOC](#directives)

