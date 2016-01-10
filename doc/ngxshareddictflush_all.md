ngx.shared.DICT.flush_all
-------------------------
**语法:** *ngx.shared.DICT:flush_all()*

**环境:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

清空字典中的所有内容。这个方法并不实际释放字典占用的内存块，而是标记所有存在的内容为已过期。

这个功能最早出现在 `v0.5.0rc17` 版本中。

更多功能请参考 [ngx.shared.DICT.flush_expired](#ngxshareddictflush_expired) 和 [ngx.shared.DICT](#ngxshareddict)。

> English Source

**syntax:** *ngx.shared.DICT:flush_all()*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Flushes out all the items in the dictionary. This method does not actuall free up all the memory blocks in the dictionary but just marks all the existing items as expired.

This feature was first introduced in the `v0.5.0rc17` release.

See also [ngx.shared.DICT.flush_expired](#ngxshareddictflush_expired) and [ngx.shared.DICT](#ngxshareddict).

[返回目录](#nginx-api-for-lua)