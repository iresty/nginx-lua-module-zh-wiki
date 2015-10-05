
tcpsock:receiveuntil
--------------------
**语法:** *iterator = tcpsock:receiveuntil(pattern, options?)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

该方法返回一个迭代的 Lua 函数，该函数可以被调用读取数据流直到指定的规则或有错误发生。

这里有个例子，使用这个方法读取边界序列为 `--abcedhb` 数据流：

```lua

 local reader = sock:receiveuntil("\r\n--abcedhb")
 local data, err, partial = reader()
 if not data then
     ngx.say("failed to read the data stream: ", err)
 end
 ngx.say("read the data stream: ", data)
```

当不使用任何参数调用时，迭代函数返回的接收数据是指定规则 *之前* 的输入数据流。所以对于上面的例子，如果输入数据流是 `'hello, world! -agentzh\r\n--abcedhb blah blah'` ，然后将返回字符串 `'hello, world! -agentzh'` 。

错误的情况下，迭代函数将返回 `nil` 、错误描述信息以及已经读取到的部分数据内容。

迭代函数可以被多次调用，并且可以安全的与其他 cosocket 方法或其他迭代函数混合调用。

当这个迭代函数使用 `size` 参数时的行为有点不同（比如，真正的迭代）。就是说，每次调用它将读取 `size` 大小的数据，最后一次调用（无论找到边界规则或遇到错误）将返回 `nil` 。该迭代函数的最后一次成功调用， `err` 的返回值也将是 `nil` 。在最后一次成功调用（返回数据 `nil`，错误信息 `nil`）之后，该迭代函数将会被重置。细看下面的例子：

```lua

 local reader = sock:receiveuntil("\r\n--abcedhb")

 while true do
     local data, err, partial = reader(4)
     if not data then
         if err then
             ngx.say("failed to read the data stream: ", err)
             break
         end

         ngx.say("read done")
         break
     end
     ngx.say("read chunk: [", data, "]")
 end
```

对于输入数据是 `'hello, world! -agentzh\r\n--abcedhb blah blah'` ，使用上面的示例代码我们将得到下面输出：


    read chunk: [hell]
    read chunk: [o, w]
    read chunk: [orld]
    read chunk: [! -a]
    read chunk: [gent]
    read chunk: [zh]
    read done


注意，当边界规则对数据流解析有歧义时，实际返回数据长度 *可能* 会略大于 `size` 参数指定的大小限制。在数据流的边界，返回的字符数据长度同样也可能小于 `size` 参数限制。

迭代函数的读操作超时控制，是由 [lua_socket_read_timeout](#lua_socket_read_timeout) 指令配置和 [settimeout](#tcpsocksettimeout) 方法设置的。而后者有更高的优先级，例如：

```lua

 local readline = sock:receiveuntil("\r\n")

 sock:settimeout(1000)  -- one second timeout
 line, err, partial = readline()
 if not line then
     ngx.say("failed to read a line: ", err)
     return
 end
 ngx.say("successfully read a line: ", line)
```

在调用迭代函数（注意 `receiveuntil` 调用在这里是不相干的） *之前* 调用 [settimeout](#tcpsocksettimeout) 方法是非常重要的。

从 `v0.5.1` 版本开始，该方法接收一个可选的 `options` 表参数来控制一些行为。支持下面这些选项：

* `inclusive`

`inclusive` 用一个布尔值来控制返回数据串是否包含规则字符串，默认是 `false`。例如：

```lua

 local reader = tcpsock:receiveuntil("_END_", { inclusive = true })
 local data = reader()
 ngx.say(data)
```

然后对于数据数据流 `"hello world _END_ blah blah blah"` ，根据上面的示例代码将得到 `hello world _END_` 的输出，包含规则字符串 `_END_` 自身。

自从 `v0.8.8` 版本，当出现读取超时错误时，该方法不再自动关闭当前连接。对于其他连接错误，该方法总是会自动关闭连接。

该特性是在 `v0.5.0rc1` 版本首次引入的。

[返回目录](#nginx-api-for-lua)

> English source:

tcpsock:receiveuntil
--------------------
**syntax:** *iterator = tcpsock:receiveuntil(pattern, options?)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

This method returns an iterator Lua function that can be called to read the data stream until it sees the specified pattern or an error occurs.

Here is an example for using this method to read a data stream with the boundary sequence `--abcedhb`:

```lua

 local reader = sock:receiveuntil("\r\n--abcedhb")
 local data, err, partial = reader()
 if not data then
     ngx.say("failed to read the data stream: ", err)
 end
 ngx.say("read the data stream: ", data)
```

When called without any argument, the iterator function returns the received data right *before* the specified pattern string in the incoming data stream. So for the example above, if the incoming data stream is `'hello, world! -agentzh\r\n--abcedhb blah blah'`, then the string `'hello, world! -agentzh'` will be returned.

In case of error, the iterator function will return `nil` along with a string describing the error and the partial data bytes that have been read so far.

The iterator function can be called multiple times and can be mixed safely with other cosocket method calls or other iterator function calls.

The iterator function behaves differently (i.e., like a real iterator) when it is called with a `size` argument. That is, it will read that `size` of data on each invocation and will return `nil` at the last invocation (either sees the boundary pattern or meets an error). For the last successful invocation of the iterator function, the `err` return value will be `nil` too. The iterator function will be reset after the last successful invocation that returns `nil` data and `nil` error. Consider the following example:

```lua

 local reader = sock:receiveuntil("\r\n--abcedhb")

 while true do
     local data, err, partial = reader(4)
     if not data then
         if err then
             ngx.say("failed to read the data stream: ", err)
             break
         end

         ngx.say("read done")
         break
     end
     ngx.say("read chunk: [", data, "]")
 end
```

Then for the incoming data stream `'hello, world! -agentzh\r\n--abcedhb blah blah'`, we shall get the following output from the sample code above:


    read chunk: [hell]
    read chunk: [o, w]
    read chunk: [orld]
    read chunk: [! -a]
    read chunk: [gent]
    read chunk: [zh]
    read done


Note that, the actual data returned *might* be a little longer than the size limit specified by the `size` argument when the boundary pattern has ambiguity for streaming parsing. Near the boundary of the data stream, the data string actually returned could also be shorter than the size limit.

Timeout for the iterator function's reading operation is controlled by the [lua_socket_read_timeout](#lua_socket_read_timeout) config directive and the [settimeout](#tcpsocksettimeout) method. And the latter takes priority. For example:

```lua

 local readline = sock:receiveuntil("\r\n")

 sock:settimeout(1000)  -- one second timeout
 line, err, partial = readline()
 if not line then
     ngx.say("failed to read a line: ", err)
     return
 end
 ngx.say("successfully read a line: ", line)
```

It is important here to call the [settimeout](#tcpsocksettimeout) method *before* calling the iterator function (note that the `receiveuntil` call is irrelevant here).

As from the `v0.5.1` release, this method also takes an optional `options` table argument to control the behavior. The following options are supported:

* `inclusive`

The `inclusive` takes a boolean value to control whether to include the pattern string in the returned data string. Default to `false`. For example,

```lua

 local reader = tcpsock:receiveuntil("_END_", { inclusive = true })
 local data = reader()
 ngx.say(data)
```

Then for the input data stream `"hello world _END_ blah blah blah"`, then the example above will output `hello world _END_`, including the pattern string `_END_` itself.

Since the `v0.8.8` release, this method no longer automatically closes the current connection when the read timeout error happens. For other connection errors, this method always automatically closes the connection.

This method was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

