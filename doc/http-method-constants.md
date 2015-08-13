HTTP 方法常量
-------------
**环境:** *init_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua, log_by_lua*\**, ngx.timer.*\*


      ngx.HTTP_GET
      ngx.HTTP_HEAD
      ngx.HTTP_PUT
      ngx.HTTP_POST
      ngx.HTTP_DELETE
      ngx.HTTP_OPTIONS   (v0.5.0rc24 版本加入)
      ngx.HTTP_MKCOL     (v0.8.2 版本加入)
      ngx.HTTP_COPY      (v0.8.2 版本加入)
      ngx.HTTP_MOVE      (v0.8.2 版本加入)
      ngx.HTTP_PROPFIND  (v0.8.2 版本加入)
      ngx.HTTP_PROPPATCH (v0.8.2 版本加入)
      ngx.HTTP_LOCK      (v0.8.2 版本加入)
      ngx.HTTP_UNLOCK    (v0.8.2 版本加入)
      ngx.HTTP_PATCH     (v0.8.2 版本加入)
      ngx.HTTP_TRACE     (v0.8.2 版本加入)


这些常量一般被用在 [ngx.location.capture](#ngxlocationcapture) 和 [ngx.location.capture_multi](#ngxlocationcapture_multi) 方法中。


> English Source 

**context:** *init_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua, log_by_lua*\**, ngx.timer.*\*


      ngx.HTTP_GET
      ngx.HTTP_HEAD
      ngx.HTTP_PUT
      ngx.HTTP_POST
      ngx.HTTP_DELETE
      ngx.HTTP_OPTIONS   (added in the v0.5.0rc24 release)
      ngx.HTTP_MKCOL     (added in the v0.8.2 release)
      ngx.HTTP_COPY      (added in the v0.8.2 release)
      ngx.HTTP_MOVE      (added in the v0.8.2 release)
      ngx.HTTP_PROPFIND  (added in the v0.8.2 release)
      ngx.HTTP_PROPPATCH (added in the v0.8.2 release)
      ngx.HTTP_LOCK      (added in the v0.8.2 release)
      ngx.HTTP_UNLOCK    (added in the v0.8.2 release)
      ngx.HTTP_PATCH     (added in the v0.8.2 release)
      ngx.HTTP_TRACE     (added in the v0.8.2 release)


These constants are usually used in [ngx.location.capture](#ngxlocationcapture) and [ngx.location.capture_multi](#ngxlocationcapture_multi) method calls.

[返回目录](#nginx-api-for-lua)