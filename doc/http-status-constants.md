HTTP 状态常量
-------------
**环境:** *init_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua, log_by_lua*\**, ngx.timer.*\*

```nginx

   value = ngx.HTTP_OK (200)
   value = ngx.HTTP_CREATED (201)
   value = ngx.HTTP_SPECIAL_RESPONSE (300)
   value = ngx.HTTP_MOVED_PERMANENTLY (301)
   value = ngx.HTTP_MOVED_TEMPORARILY (302)
   value = ngx.HTTP_SEE_OTHER (303)
   value = ngx.HTTP_NOT_MODIFIED (304)
   value = ngx.HTTP_BAD_REQUEST (400)
   value = ngx.HTTP_UNAUTHORIZED (401)
   value = ngx.HTTP_FORBIDDEN (403)
   value = ngx.HTTP_NOT_FOUND (404)
   value = ngx.HTTP_NOT_ALLOWED (405)
   value = ngx.HTTP_GONE (410)
   value = ngx.HTTP_INTERNAL_SERVER_ERROR (500)
   value = ngx.HTTP_METHOD_NOT_IMPLEMENTED (501)
   value = ngx.HTTP_SERVICE_UNAVAILABLE (503)
   value = ngx.HTTP_GATEWAY_TIMEOUT (504) (v0.3.1rc38 版本加入)
```

> English Source

**context:** *init_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua, log_by_lua*\**, ngx.timer.*\*

```nginx

   value = ngx.HTTP_OK (200)
   value = ngx.HTTP_CREATED (201)
   value = ngx.HTTP_SPECIAL_RESPONSE (300)
   value = ngx.HTTP_MOVED_PERMANENTLY (301)
   value = ngx.HTTP_MOVED_TEMPORARILY (302)
   value = ngx.HTTP_SEE_OTHER (303)
   value = ngx.HTTP_NOT_MODIFIED (304)
   value = ngx.HTTP_BAD_REQUEST (400)
   value = ngx.HTTP_UNAUTHORIZED (401)
   value = ngx.HTTP_FORBIDDEN (403)
   value = ngx.HTTP_NOT_FOUND (404)
   value = ngx.HTTP_NOT_ALLOWED (405)
   value = ngx.HTTP_GONE (410)
   value = ngx.HTTP_INTERNAL_SERVER_ERROR (500)
   value = ngx.HTTP_METHOD_NOT_IMPLEMENTED (501)
   value = ngx.HTTP_SERVICE_UNAVAILABLE (503)
   value = ngx.HTTP_GATEWAY_TIMEOUT (504) (first added in the v0.3.1rc38 release)
```

[返回目录](#nginx-api-for-lua)