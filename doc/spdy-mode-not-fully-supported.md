SPDY 模式支持不完美
-----------------------------

一些ngx_lua提供的Lua APIs在Nginx的`SPDY`模式下确定不能工作：ngx.location.capture](#ngxlocationcapture)， [ngx.location.capture_multi](#ngxlocationcapture_multi)，和[ngx.req.socket](#ngxreqsocket)。

[返回目录](#table-of-contents)

> English source:

SPDY Mode Not Fully Supported
-----------------------------

Certain Lua APIs provided by ngx_lua do not work in Nginx's SPDY mode yet: [ngx.location.capture](#ngxlocationcapture), [ngx.location.capture_multi](#ngxlocationcapture_multi), and [ngx.req.socket](#ngxreqsocket).

[Back to TOC](#table-of-contents)
