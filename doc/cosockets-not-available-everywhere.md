Cosockets Not Available Everywhere
----------------------------------

Due the internal limitations in the nginx core, the cosocket API are disabled in the following contexts: [set_by_lua*](#set_by_lua), [log_by_lua*](#log_by_lua), [header_filter_by_lua*](#header_filter_by_lua), and [body_filter_by_lua](#body_filter_by_lua).

The cosockets are currently also disabled in the [init_by_lua*](#init_by_lua) and [init_worker_by_lua*](#init_worker_by_lua) directive contexts but we may add support for these contexts in the future because there is no limitation in the nginx core (or the limitation might be worked around).

There exists a work-around, however, when the original context does *not* need to wait for the cosocket results. That is, creating a 0-delay timer via the [ngx.timer.at](#ngxtimerat) API and do the cosocket results in the timer handler, which runs asynchronously as to the original context creating the timer.

[Back to TOC](#table-of-contents)

> English source:

Cosockets Not Available Everywhere
----------------------------------

Due the internal limitations in the nginx core, the cosocket API are disabled in the following contexts: [set_by_lua*](#set_by_lua), [log_by_lua*](#log_by_lua), [header_filter_by_lua*](#header_filter_by_lua), and [body_filter_by_lua](#body_filter_by_lua).

The cosockets are currently also disabled in the [init_by_lua*](#init_by_lua) and [init_worker_by_lua*](#init_worker_by_lua) directive contexts but we may add support for these contexts in the future because there is no limitation in the nginx core (or the limitation might be worked around).

There exists a work-around, however, when the original context does *not* need to wait for the cosocket results. That is, creating a 0-delay timer via the [ngx.timer.at](#ngxtimerat) API and do the cosocket results in the timer handler, which runs asynchronously as to the original context creating the timer.

[Back to TOC](#table-of-contents)
