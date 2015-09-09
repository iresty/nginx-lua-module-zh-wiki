ngx.md5
-------
**语法:** *digest = ngx.md5(str)*

**环境:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

通过计算 `str` 字符串返回十六进制的数据。

举一个例子,

```nginx

 location = /md5 {
     content_by_lua 'ngx.say(ngx.md5("hello"))';
 }
```

将会输出


    5d41402abc4b2a76b9719d911017c592


如果需要返回二进制数据请看 [ngx.md5_bin](ngxmd5_bin.md) 方法。


[Back to TOC](#nginx-api-for-lua)

> English source:

**syntax:** *digest = ngx.md5(str)*

**context:** *set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

Returns the hexadecimal representation of the MD5 digest of the `str` argument.

For example,

```nginx

 location = /md5 {
     content_by_lua 'ngx.say(ngx.md5("hello"))';
 }
```

yields the output


    5d41402abc4b2a76b9719d911017c592


See [ngx.md5_bin](#ngxmd5_bin) if the raw binary MD5 digest is required.

