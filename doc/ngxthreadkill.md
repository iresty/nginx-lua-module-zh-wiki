ngx.thread.kill
---------------
**语法:** *ok, err = ngx.thread.kill(thread)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

杀死一个正在运行的轻线程（通过 [ngx.thread.spawn](#ngxthreadspawn) 创建）。成功时返回一个 true ，其他情况则返回一个错误字符描述信息。

根据目前的实现，只有父协程（或“轻线程”）可以终止一个“线程”。同样，正在挂起运行 Nginx 子请求（例如调用 [ngx.location.capture](#ngxlocationcapture)）的“轻线程”，是不能被杀死的，这要归咎于 Nginx 内核限制。

该 API 是从 `v0.9.9` 版本开始首次引入。

[返回目录](#nginx-api-for-lua)

> English source:

ngx.thread.kill
---------------
**syntax:** *ok, err = ngx.thread.kill(thread)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Kills a running "light thread" created by [ngx.thread.spawn](#ngxthreadspawn). Returns a true value when successful or `nil` and a string describing the error otherwise.

According to the current implementation, only the parent coroutine (or "light thread") can kill a thread. Also, a running "light thread" with pending NGINX subrequests (initiated by [ngx.location.capture](#ngxlocationcapture) for example) cannot be killed due to a limitation in the NGINX core.

This API was first enabled in the `v0.9.9` release.

[Back to TOC](#nginx-api-for-lua)
