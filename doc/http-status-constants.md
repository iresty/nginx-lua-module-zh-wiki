HTTP 状态常量
-------------
**环境:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;, ngx.timer.&#42;*

```nginx

   value = ngx.HTTP_CONTINUE (100) (v0.9.20 版本首次加入)
   value = ngx.HTTP_SWITCHING_PROTOCOLS (101) (v0.9.20 版本首次加入)
   value = ngx.HTTP_OK (200)
   value = ngx.HTTP_CREATED (201)
   value = ngx.HTTP_ACCEPTED (202) (v0.9.20 版本首次加入)
   value = ngx.HTTP_NO_CONTENT (204) (v0.9.20 版本首次加入)
   value = ngx.HTTP_PARTIAL_CONTENT (206) (v0.9.20 版本首次加入)
   value = ngx.HTTP_SPECIAL_RESPONSE (300)
   value = ngx.HTTP_MOVED_PERMANENTLY (301)
   value = ngx.HTTP_MOVED_TEMPORARILY (302)
   value = ngx.HTTP_SEE_OTHER (303)
   value = ngx.HTTP_NOT_MODIFIED (304)
   value = ngx.HTTP_TEMPORARY_REDIRECT (307) (v0.9.20 版本首次加入)
   value = ngx.HTTP_BAD_REQUEST (400)
   value = ngx.HTTP_UNAUTHORIZED (401)
   value = ngx.HTTP_PAYMENT_REQUIRED (402) (v0.9.20 版本首次加入)
   value = ngx.HTTP_FORBIDDEN (403)
   value = ngx.HTTP_NOT_FOUND (404)
   value = ngx.HTTP_NOT_ALLOWED (405)
   value = ngx.HTTP_NOT_ACCEPTABLE (406) (v0.9.20 版本首次加入)
   value = ngx.HTTP_REQUEST_TIMEOUT (408) (v0.9.20 版本首次加入)
   value = ngx.HTTP_CONFLICT (409) (v0.9.20 版本首次加入)
   value = ngx.HTTP_GONE (410)
   value = ngx.HTTP_UPGRADE_REQUIRED (426) (v0.9.20 版本首次加入)
   value = ngx.HTTP_TOO_MANY_REQUESTS (429) (v0.9.20 版本首次加入)
   value = ngx.HTTP_CLOSE (444) (v0.9.20 版本首次加入)
   value = ngx.HTTP_ILLEGAL (451) (v0.9.20 版本首次加入)
   value = ngx.HTTP_INTERNAL_SERVER_ERROR (500)
   value = ngx.HTTP_METHOD_NOT_IMPLEMENTED (501)
   value = ngx.HTTP_BAD_GATEWAY (502) (v0.9.20 版本首次加入)
   value = ngx.HTTP_SERVICE_UNAVAILABLE (503)
   value = ngx.HTTP_GATEWAY_TIMEOUT (504) (v0.3.1rc38 版本首次加入)
   value = ngx.HTTP_VERSION_NOT_SUPPORTED (505) (v0.9.20 版本首次加入)
   value = ngx.HTTP_INSUFFICIENT_STORAGE (507) (v0.9.20 版本首次加入)
```

> English Source

HTTP status constants
---------------------

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua, log_by_lua&#42;, ngx.timer.&#42;*

```nginx

   value = ngx.HTTP_CONTINUE (100) (first added in the v0.9.20 release)
   value = ngx.HTTP_SWITCHING_PROTOCOLS (101) (first added in the v0.9.20 release)
   value = ngx.HTTP_OK (200)
   value = ngx.HTTP_CREATED (201)
   value = ngx.HTTP_ACCEPTED (202) (first added in the v0.9.20 release)
   value = ngx.HTTP_NO_CONTENT (204) (first added in the v0.9.20 release)
   value = ngx.HTTP_PARTIAL_CONTENT (206) (first added in the v0.9.20 release)
   value = ngx.HTTP_SPECIAL_RESPONSE (300)
   value = ngx.HTTP_MOVED_PERMANENTLY (301)
   value = ngx.HTTP_MOVED_TEMPORARILY (302)
   value = ngx.HTTP_SEE_OTHER (303)
   value = ngx.HTTP_NOT_MODIFIED (304)
   value = ngx.HTTP_TEMPORARY_REDIRECT (307) (first added in the v0.9.20 release)
   value = ngx.HTTP_BAD_REQUEST (400)
   value = ngx.HTTP_UNAUTHORIZED (401)
   value = ngx.HTTP_PAYMENT_REQUIRED (402) (first added in the v0.9.20 release)
   value = ngx.HTTP_FORBIDDEN (403)
   value = ngx.HTTP_NOT_FOUND (404)
   value = ngx.HTTP_NOT_ALLOWED (405)
   value = ngx.HTTP_NOT_ACCEPTABLE (406) (first added in the v0.9.20 release)
   value = ngx.HTTP_REQUEST_TIMEOUT (408) (first added in the v0.9.20 release)
   value = ngx.HTTP_CONFLICT (409) (first added in the v0.9.20 release)
   value = ngx.HTTP_GONE (410)
   value = ngx.HTTP_UPGRADE_REQUIRED (426) (first added in the v0.9.20 release)
   value = ngx.HTTP_TOO_MANY_REQUESTS (429) (first added in the v0.9.20 release)
   value = ngx.HTTP_CLOSE (444) (first added in the v0.9.20 release)
   value = ngx.HTTP_ILLEGAL (451) (first added in the v0.9.20 release)
   value = ngx.HTTP_INTERNAL_SERVER_ERROR (500)
   value = ngx.HTTP_METHOD_NOT_IMPLEMENTED (501)
   value = ngx.HTTP_BAD_GATEWAY (502) (first added in the v0.9.20 release)
   value = ngx.HTTP_SERVICE_UNAVAILABLE (503)
   value = ngx.HTTP_GATEWAY_TIMEOUT (504) (first added in the v0.3.1rc38 release)
   value = ngx.HTTP_VERSION_NOT_SUPPORTED (505) (first added in the v0.9.20 release)
   value = ngx.HTTP_INSUFFICIENT_STORAGE (507) (first added in the v0.9.20 release)
```

[返回目录](#nginx-api-for-lua)