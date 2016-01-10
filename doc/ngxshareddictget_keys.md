ngx.shared.DICT.get_keys
------------------------
**语法:** *keys = ngx.shared.DICT:get_keys(max_count?)*

**环境:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

获取字典中存储的 key 列表，最多 `<max_count>` 个。

默认时，前 1024 个 key (如果有) 被返回。当参数 `<max_count>` 值为 `0` 时，字典中所有的 key 被返回，即使超过 1024 个。

**警告** 在包含非常多 key 的字典中调用此方法要非常小心。此方法会锁定字典一段时间，会阻塞所有访问字典的 nginx worker 进程。

这个功能最早出现在 `v0.7.3` 版本中。

> English Source

**syntax:** *keys = ngx.shared.DICT:get_keys(max_count?)*

**context:** *init_by_lua&#42;, set_by_lua&#42;, rewrite_by_lua&#42;, access_by_lua&#42;, content_by_lua&#42;, header_filter_by_lua&#42;, body_filter_by_lua&#42;, log_by_lua&#42;, ngx.timer.&#42;*

Fetch a list of the keys from the dictionary, up to `<max_count>`.

By default, only the first 1024 keys (if any) are returned. When the `<max_count>` argument is given the value `0`, then all the keys will be returned even there is more than 1024 keys in the dictionary.

**WARNING** Be careful when calling this method on dictionaries with a really huge number of keys. This method may lock the dictionary for quite a while and block all the nginx worker processes that are trying to access the dictionary.

This feature was first introduced in the `v0.7.3` release.

[返回目录](#nginx-api-for-lua)