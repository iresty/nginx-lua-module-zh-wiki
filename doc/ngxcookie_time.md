ngx.cookie_time
---------------
**语法:** *str = ngx.cookie_time(sec)*

**环境:** *init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

返回一个可以用做 cookie 过期时间的格式化字符串。参数 `sec` 是以秒为单位的时间戳（比如 [ngx.time](#ngxtime) 的返回）。

```nginx

 ngx.say(ngx.cookie_time(1290079655))
     -- yields "Thu, 18-Nov-10 11:27:35 GMT"
```

[返回目录](#nginx-api-for-lua)

> English source:


ngx.cookie_time
---------------
**syntax:** *str = ngx.cookie_time(sec)*

**context:** *init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.**

Returns a formatted string can be used as the cookie expiration time. The parameter `sec` is the time stamp in seconds (like those returned from [ngx.time](#ngxtime)).

```nginx

 ngx.say(ngx.cookie_time(1290079655))
     -- yields "Thu, 18-Nov-10 11:27:35 GMT"
```

[Back to TOC](#nginx-api-for-lua)