ngx.shared.DICT.set
-------------------
**语法:** *success, err, forcible = ngx.shared.DICT:set(key, value, exptime?, flags?)*

**环境:** *init_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

无条件在基于共享内存的字典 [ngx.shared.DICT](#ngxshareddict) 中设置一个 key-value (键-值)对。返回三个值：

* `success`: 布尔值，用以标识 key-value 对是否存储成功；
* `err`: 文字描述的错误信息，例如 `"no memory"` (内存不足)；
* `forcible`: 布尔值，用以标识是否有其他可用项被强制删除，因为共享内存区块中的存储空间不足。

`value` 参数可以是 Lua 布尔值、数字、字符串，或 `nil`。其类型也将被存储在字典中，之后通过 [get](#ngxshareddictget) 方法可以原样取出。

通过可选的 `exptime` 参数给写入的 key-value 对设定过期时间 (单位秒)。时间的精度是 `0.001` 秒。如果 `exptime` 送入 `0` (默认值)，此项将永不过期。

通过可选的 `flags` 参数给写入项设定一个用户标志，之后可以与值一起被取出。用户标志在内部被存储为一个 32 位无符号整数。默认值是 `0`。此参数最早出现在 `v0.5.0rc2` 版本中。

当无法给当前的 key-value 项分配内存时，`set` 方法将根据最近最少使用 (Least-Recently Used, LRU) 算法，删除存储中已有的项。需要注意的是，LRU 算法在这里优先考虑过期时间。如果在删除了最多数十项以后剩余存储空间依旧不足 (由于 [lua_shared_dict](#lua_shared_dict) 定义的总存储空间限制或内存碎片化)，`success` 将返回 `false`，同时 `err` 将返回 `no memory`。

如果此方法执行成功的同时，根据 LRU 算法强制移除了字典中其他尚未过期的项，`forcible` 返回值将是 `true`。如果存储时没有移除其他有效项，`forcible` 返回值将是 `false`。

此方法第一个参数必须是该字典对象本身，例如，

```lua

 local cats = ngx.shared.cats
 local succ, err, forcible = cats.set(cats, "Marry", "it is a nice cat!")
```

或使用 Lua 的“语法糖”形式来调用方法：

```lua

 local cats = ngx.shared.cats
 local succ, err, forcible = cats:set("Marry", "it is a nice cat!")
```

这两种形式完全等价。

这个功能最早出现在 `v0.3.1rc22` 版本中。

这里需要注意的是，虽然在内部 key-value 对设置是原子性的，但这种原子性无法跨过方法调用边界。

更多功能请参考 [ngx.shared.DICT](#ngxshareddict)。


> English Source

**syntax:** *success, err, forcible = ngx.shared.DICT:set(key, value, exptime?, flags?)*

**context:** *init_by_lua*\**, set_by_lua*\**, rewrite_by_lua*\**, access_by_lua*\**, content_by_lua*\**, header_filter_by_lua*\**, body_filter_by_lua*\**, log_by_lua*\**, ngx.timer.*\*

Unconditionally sets a key-value pair into the shm-based dictionary [ngx.shared.DICT](#ngxshareddict). Returns three values:

* `success`: boolean value to indicate whether the key-value pair is stored or not.
* `err`: textual error message, can be `"no memory"`.
* `forcible`: a boolean value to indicate whether other valid items have been removed forcibly when out of storage in the shared memory zone.

The `value` argument inserted can be Lua booleans, numbers, strings, or `nil`. Their value type will also be stored into the dictionary and the same data type can be retrieved later via the [get](#ngxshareddictget) method.

The optional `exptime` argument specifies expiration time (in seconds) for the inserted key-value pair. The time resolution is `0.001` seconds. If the `exptime` takes the value `0` (which is the default), then the item will never be expired.

The optional `flags` argument specifies a user flags value associated with the entry to be stored. It can also be retrieved later with the value. The user flags is stored as an unsigned 32-bit integer internally. Defaults to `0`. The user flags argument was first introduced in the `v0.5.0rc2` release.

When it fails to allocate memory for the current key-value item, then `set` will try removing existing items in the storage according to the Least-Recently Used (LRU) algorithm. Note that, LRU takes priority over expiration time here. If up to tens of existing items have been removed and the storage left is still insufficient (either due to the total capacity limit specified by [lua_shared_dict](#lua_shared_dict) or memory segmentation), then the `err` return value will be `no memory` and `success` will be `false`.

If this method succeeds in storing the current item by forcibly removing other not-yet-expired items in the dictionary via LRU, the `forcible` return value will be `true`. If it stores the item without forcibly removing other valid items, then the return value `forcible` will be `false`.

The first argument to this method must be the dictionary object itself, for example,

```lua

 local cats = ngx.shared.cats
 local succ, err, forcible = cats.set(cats, "Marry", "it is a nice cat!")
```

or use Lua's syntactic sugar for method calls:

```lua

 local cats = ngx.shared.cats
 local succ, err, forcible = cats:set("Marry", "it is a nice cat!")
```

These two forms are fundamentally equivalent.

This feature was first introduced in the `v0.3.1rc22` release.

Please note that while internally the key-value pair is set atomically, the atomicity does not go across the method call boundary.

See also [ngx.shared.DICT](#ngxshareddict).

[返回目录](#nginx-api-for-lua)