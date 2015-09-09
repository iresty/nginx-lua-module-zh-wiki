Cosockets 不是在任何地方都可用
----------------------------------

归咎于`nginx`内核的各种限制规则，cosocket API 在这些场景中是被禁的：
[set_by_lua*](#set_by_lua)， [log_by_lua*](#log_by_lua)， [header_filter_by_lua*](#header_filter_by_lua)， 和 [body_filter_by_lua](#body_filter_by_lua)。

cosocket在[init_by_lua*](#init_by_lua) 和 [init_worker_by_lua*](#init_worker_by_lua)小节中也是被禁的，但我们后面将会添加这些环境的支持，因为在nginx内核上是没有这个限制的（或者这个限制是可以被绕过的）。

这里有个绕路方法，前提是原始场景 *不* 需要等待cosocket结果。就是说，通过[ngx.timer.at](#ngxtimerat) API 创建一个零延迟的`timer`，在`timer`中完成cosocket的处理结果，用这种异步的方式进行协作。

[返回目录](#table-of-contents)

> English source:

Cosockets Not Available Everywhere
----------------------------------

Due the internal limitations in the nginx core, the cosocket API are disabled in the following contexts: [set_by_lua*](#set_by_lua), [log_by_lua*](#log_by_lua), [header_filter_by_lua*](#header_filter_by_lua), and [body_filter_by_lua](#body_filter_by_lua).

The cosockets are currently also disabled in the [init_by_lua*](#init_by_lua) and [init_worker_by_lua*](#init_worker_by_lua) directive contexts but we may add support for these contexts in the future because there is no limitation in the nginx core (or the limitation might be worked around).

There exists a work-around, however, when the original context does *not* need to wait for the cosocket results. That is, creating a 0-delay timer via the [ngx.timer.at](#ngxtimerat) API and do the cosocket results in the timer handler, which runs asynchronously as to the original context creating the timer.

[Back to TOC](#table-of-contents)
