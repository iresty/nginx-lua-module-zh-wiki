短小请求引起丢失数据
----------------------------------------

Nginx提前销毁一个请求的可能（至少）：

* 400 (Bad Request)                 -- 错误请求
* 405 (Not Allowed)                 -- 不允许
* 408 (Request Timeout)             -- 请求超时
* 414 (Request URI Too Large)       -- 请求URI过大
* 494 (Request Headers Too Large)   -- 请求Headers过大
* 499 (Client Closed Request)       -- 客户端关闭请求
* 500 (Internal Server Error)       -- 内部服务错误
* 501 (Not Implemented)             -- 未实现

这意味着正常执行阶段被跳过，如重写或访问阶段。这也意味着，后面的所有阶段，例如
[log_by_lua](#log_by_lua)，将无法获得在这个阶段存放的普通信息。

[返回目录](#table-of-contents)

> English source:

Missing data on short circuited requests
----------------------------------------

Nginx may terminate a request early with (at least):

* 400 (Bad Request)
* 405 (Not Allowed)
* 408 (Request Timeout)
* 414 (Request URI Too Large)
* 494 (Request Headers Too Large)
* 499 (Client Closed Request)
* 500 (Internal Server Error)
* 501 (Not Implemented)

This means that phases that normally run are skipped, such as the rewrite or
access phase. This also means that later phases that are run regardless, e.g.
[log_by_lua](#log_by_lua), will not have access to information that is normally set in those
phases.

[Back to TOC](#table-of-contents)
