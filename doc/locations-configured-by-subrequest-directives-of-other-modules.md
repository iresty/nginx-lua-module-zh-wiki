其它模块子请求指令的 Location 配置
--------------------------------------------------------------
[ngx.location.capture](#ngxlocationcapture) 和 [ngx.location.capture_multi](#ngxlocationcapture_multi) 指令无法抓取包含以下指令的 location： [add_before_body](http://nginx.org/en/docs/http/ngx_http_addition_module.html#add_before_body), [add_after_body](http://nginx.org/en/docs/http/ngx_http_addition_module.html#add_after_body), [auth_request](http://nginx.org/en/docs/http/ngx_http_auth_request_module.html#auth_request), [echo_location](http://github.com/openresty/echo-nginx-module#echo_location), [echo_location_async](http://github.com/openresty/echo-nginx-module#echo_location_async), [echo_subrequest](http://github.com/openresty/echo-nginx-module#echo_subrequest), 或 [echo_subrequest_async](http://github.com/openresty/echo-nginx-module#echo_subrequest_async) 。

```nginx

 location /foo {
     content_by_lua '
         res = ngx.location.capture("/bar")
     ';
 }
 location /bar {
     echo_location /blah;
 }
 location /blah {
     echo "Success!";
 }
```

```nginx

 $ curl -i http://example.com/foo
```

将不会如预期一样的工作。


> English Source

The [ngx.location.capture](#ngxlocationcapture) and [ngx.location.capture_multi](#ngxlocationcapture_multi) directives cannot capture locations that include the [add_before_body](http://nginx.org/en/docs/http/ngx_http_addition_module.html#add_before_body), [add_after_body](http://nginx.org/en/docs/http/ngx_http_addition_module.html#add_after_body), [auth_request](http://nginx.org/en/docs/http/ngx_http_auth_request_module.html#auth_request), [echo_location](http://github.com/openresty/echo-nginx-module#echo_location), [echo_location_async](http://github.com/openresty/echo-nginx-module#echo_location_async), [echo_subrequest](http://github.com/openresty/echo-nginx-module#echo_subrequest), or [echo_subrequest_async](http://github.com/openresty/echo-nginx-module#echo_subrequest_async) directives.

```nginx

 location /foo {
     content_by_lua '
         res = ngx.location.capture("/bar")
     ';
 }
 location /bar {
     echo_location /blah;
 }
 location /blah {
     echo "Success!";
 }
```

```nginx

 $ curl -i http://example.com/foo
```

will not work as expected.

[返回目录](#nginx-api-for-lua)