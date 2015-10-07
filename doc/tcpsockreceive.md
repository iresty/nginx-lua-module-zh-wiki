
tcpsock:receive
---------------
**语法:** *data, err, partial = tcpsock:receive(size)*

**语法:** *data, err, partial = tcpsock:receive(pattern?)*

**环境:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

根据读取规则或大小，从当前连接 socket 中接收数据。

该方法是一个异步操作就像 [send](#tcpsocksend) 方法一样，并且是 100% 非阻塞的。

成功情况下，返回接收到的数据；失败情况，返回 `nil` 、一个错误描述信息以及目前接收到的部分数据。

如果指定一个像数字的参数（里面是数字的字符串），它将解析为大小。该方法将不回返回直到读取明确大小的数据或有错误发生。

如果一个不像数字的字符串参数被指定，它将解析为“规则”。支持下面的规则：

* `'*a'`：从 socket 中读取内容直到连接被关闭。这里是不执行 `end-of-line` 翻译。
* `'*l'`：从 socket 中读取一行文本信息。行结束符是 `Line Feed` （LF） 字符（ASCII 10），前面是一个可选 `Carriage Return` （CR）字符（ASCII 13）。返回行信息中不包含 CR 和 LF 字符。实际上，所有的 CR 字符在此规则中都是被忽略的。

如果没有参数指定，它将被假定为规则 `'*l'` ，也就是说，使用按行读取的规则。

读操作超时控制，是由 [lua_socket_read_timeout](#lua_socket_read_timeout) 配置指令和 [settimeout](#tcpsocksettimeout) 方法设置的。而后者有更高的优先级，例如：

```lua

 sock:settimeout(1000)  -- one second timeout
 local line, err, partial = sock:receive()
 if not line then
     ngx.say("failed to read a line: ", err)
     return
 end
 ngx.say("successfully read a line: ", line)
```


调用这个方法 *之前* 调用 [settimeout](#tcpsocksettimeout) 方法设置超时时间，是非常重要的。

自从 `v0.8.8` 版本，当出现读取超时错误时，该方法不再自动关闭当前连接。对于其他连接错误，该方法总是会自动关闭连接。

该特性是在 `v0.5.0rc1` 版本首次引入的。

[返回目录](#nginx-api-for-lua)

> English source:


tcpsock:receive
---------------
**syntax:** *data, err, partial = tcpsock:receive(size)*

**syntax:** *data, err, partial = tcpsock:receive(pattern?)*

**context:** *rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.**

Receives data from the connected socket according to the reading pattern or size.

This method is a synchronous operation just like the [send](#tcpsocksend) method and is 100% nonblocking.

In case of success, it returns the data received; in case of error, it returns `nil` with a string describing the error and the partial data received so far.

If a number-like argument is specified (including strings that look like numbers), then it is interpreted as a size. This method will not return until it reads exactly this size of data or an error occurs.

If a non-number-like string argument is specified, then it is interpreted as a "pattern". The following patterns are supported:

* `'*a'`: reads from the socket until the connection is closed. No end-of-line translation is performed;
* `'*l'`: reads a line of text from the socket. The line is terminated by a `Line Feed` (LF) character (ASCII 10), optionally preceded by a `Carriage Return` (CR) character (ASCII 13). The CR and LF characters are not included in the returned line. In fact, all CR characters are ignored by the pattern.

If no argument is specified, then it is assumed to be the pattern `'*l'`, that is, the line reading pattern.

Timeout for the reading operation is controlled by the [lua_socket_read_timeout](#lua_socket_read_timeout) config directive and the [settimeout](#tcpsocksettimeout) method. And the latter takes priority. For example:

```lua

 sock:settimeout(1000)  -- one second timeout
 local line, err, partial = sock:receive()
 if not line then
     ngx.say("failed to read a line: ", err)
     return
 end
 ngx.say("successfully read a line: ", line)
```

It is important here to call the [settimeout](#tcpsocksettimeout) method *before* calling this method.

Since the `v0.8.8` release, this method no longer automatically closes the current connection when the read timeout error happens. For other connection errors, this method always automatically closes the connection.

This feature was first introduced in the `v0.5.0rc1` release.

[Back to TOC](#nginx-api-for-lua)

