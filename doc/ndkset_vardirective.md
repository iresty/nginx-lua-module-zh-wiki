ndk.set_var.DIRECTIVE
---------------------
**语法:** *res = ndk.set_var.DIRECTIVE_NAME*

**环境:** *init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

该机制允许调用这类 nginx C 模块指令：使用 [Nginx Devel Kit](https://github.com/simpl/ngx_devel_kit) (NDK) 的 set_var 的子模块的`ndk_set_var_value` 实现。

例如，下列[set-misc-nginx-module](http://github.com/openresty/set-misc-nginx-module)指令是可以通过这个方式调用的：

* [set_quote_sql_str](http://github.com/openresty/set-misc-nginx-module#set_quote_sql_str)
* [set_quote_pgsql_str](http://github.com/openresty/set-misc-nginx-module#set_quote_pgsql_str)
* [set_quote_json_str](http://github.com/openresty/set-misc-nginx-module#set_quote_json_str)
* [set_unescape_uri](http://github.com/openresty/set-misc-nginx-module#set_unescape_uri)
* [set_escape_uri](http://github.com/openresty/set-misc-nginx-module#set_escape_uri)
* [set_encode_base32](http://github.com/openresty/set-misc-nginx-module#set_encode_base32)
* [set_decode_base32](http://github.com/openresty/set-misc-nginx-module#set_decode_base32)
* [set_encode_base64](http://github.com/openresty/set-misc-nginx-module#set_encode_base64)
* [set_decode_base64](http://github.com/openresty/set-misc-nginx-module#set_decode_base64)
* [set_encode_hex](http://github.com/openresty/set-misc-nginx-module#set_encode_base64)
* [set_decode_hex](http://github.com/openresty/set-misc-nginx-module#set_decode_base64)
* [set_sha1](http://github.com/openresty/set-misc-nginx-module#set_encode_base64)
* [set_md5](http://github.com/openresty/set-misc-nginx-module#set_decode_base64)

举例：

```lua

 local res = ndk.set_var.set_escape_uri('a/b');
 -- now res == 'a%2fb'
```

相似的，下列指令是由 [encrypted-session-nginx-module](http://github.com/openresty/encrypted-session-nginx-module) 提供，他们在 Lua 中也可以被调用：

* [set_encrypt_session](http://github.com/openresty/encrypted-session-nginx-module#set_encrypt_session)
* [set_decrypt_session](http://github.com/openresty/encrypted-session-nginx-module#set_decrypt_session)

这个特性需要 [ngx_devel_kit](https://github.com/simpl/ngx_devel_kit) 模块。

[返回目录](#nginx-api-for-lua)

> English source:

ndk.set_var.DIRECTIVE
---------------------
**syntax:** *res = ndk.set_var.DIRECTIVE_NAME*

**context:** *init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

This mechanism allows calling other nginx C modules' directives that are implemented by [Nginx Devel Kit](https://github.com/simpl/ngx_devel_kit) (NDK)'s set_var submodule's `ndk_set_var_value`.

For example, the following [set-misc-nginx-module](http://github.com/openresty/set-misc-nginx-module) directives can be invoked this way:

* [set_quote_sql_str](http://github.com/openresty/set-misc-nginx-module#set_quote_sql_str)
* [set_quote_pgsql_str](http://github.com/openresty/set-misc-nginx-module#set_quote_pgsql_str)
* [set_quote_json_str](http://github.com/openresty/set-misc-nginx-module#set_quote_json_str)
* [set_unescape_uri](http://github.com/openresty/set-misc-nginx-module#set_unescape_uri)
* [set_escape_uri](http://github.com/openresty/set-misc-nginx-module#set_escape_uri)
* [set_encode_base32](http://github.com/openresty/set-misc-nginx-module#set_encode_base32)
* [set_decode_base32](http://github.com/openresty/set-misc-nginx-module#set_decode_base32)
* [set_encode_base64](http://github.com/openresty/set-misc-nginx-module#set_encode_base64)
* [set_decode_base64](http://github.com/openresty/set-misc-nginx-module#set_decode_base64)
* [set_encode_hex](http://github.com/openresty/set-misc-nginx-module#set_encode_base64)
* [set_decode_hex](http://github.com/openresty/set-misc-nginx-module#set_decode_base64)
* [set_sha1](http://github.com/openresty/set-misc-nginx-module#set_encode_base64)
* [set_md5](http://github.com/openresty/set-misc-nginx-module#set_decode_base64)

For instance,

```lua

 local res = ndk.set_var.set_escape_uri('a/b');
 -- now res == 'a%2fb'
```

Similarly, the following directives provided by [encrypted-session-nginx-module](http://github.com/openresty/encrypted-session-nginx-module) can be invoked from within Lua too:

* [set_encrypt_session](http://github.com/openresty/encrypted-session-nginx-module#set_encrypt_session)
* [set_decrypt_session](http://github.com/openresty/encrypted-session-nginx-module#set_decrypt_session)

This feature requires the [ngx_devel_kit](https://github.com/simpl/ngx_devel_kit) module.

[Back to TOC](#nginx-api-for-lua)