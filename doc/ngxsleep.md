ngx.sleep
---------
**语法:** *ngx.sleep(seconds)*

**环境:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;*

无阻塞地休眠特定秒。时间可以精确到 0.001 秒 (毫秒)。

在后台，此方法使用 Nginx 的定时器。

自版本 `0.7.20` 开始，`0` 也可以作为时间参数被指定。

这个方法最早在版本 `0.5.0rc30` 中出现。


> English Source

**syntax:** *ngx.sleep(seconds)*

**context:** *rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, ngx.timer.&#42;*

Sleeps for the specified seconds without blocking. One can specify time resolution up to 0.001 seconds (i.e., one milliseconds).

Behind the scene, this method makes use of the Nginx timers.

Since the `0.7.20` release, The `0` time argument can also be specified.

This method was introduced in the `0.5.0rc30` release.


[返回目录](#nginx-api-for-lua)