ngx.hmac_sha1
-------------
**语法:** *digest = ngx.hmac_sha1(secret_key, str)*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

通过 `str` 待运算数据和 `secret_key` 密钥串生成结果。关于[HMAC-SHA1](http://en.wikipedia.org/wiki/HMAC)。

通过 `HMAC-SHA1` 的运算会得到二进制数据，如果你想要把结果转为文本形式，你可以使用 [ngx.encode_base64](ngxencode_base64.md) 函数。

举一个例子,

```lua

 local key = "thisisverysecretstuff"
 local src = "some string we want to sign"
 local digest = ngx.hmac_sha1(key, src)
 ngx.say(ngx.encode_base64(digest))
```

将会输出


    R/pvxzHC4NLtj7S+kXFg/NePTmk=


这个 `API` 需要在安装 `Nginx` 时启用 `OpenSSL` 库（通常通过 `./configure` 脚本的 `--with-http_ssl_module` 选项来控制）


这个方法最早出现在版本 `v0.3.1rc29` 中。

[Back to TOC](#nginx-api-for-lua)


> English source:
**syntax:** *digest = ngx.hmac_sha1(secret_key, str)*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

Computes the [HMAC-SHA1](http://en.wikipedia.org/wiki/HMAC) digest of the argument `str` and turns the result using the secret key `<secret_key>`.

The raw binary form of the `HMAC-SHA1` digest will be generated, use [ngx.encode_base64](#ngxencode_base64), for example, to encode the result to a textual representation if desired.

For example,

```lua

 local key = "thisisverysecretstuff"
 local src = "some string we want to sign"
 local digest = ngx.hmac_sha1(key, src)
 ngx.say(ngx.encode_base64(digest))
```

yields the output


    R/pvxzHC4NLtj7S+kXFg/NePTmk=


This API requires the OpenSSL library enabled in the Nginx build (usually by passing the `--with-http_ssl_module` option to the `./configure` script).

This function was first introduced in the `v0.3.1rc29` release.