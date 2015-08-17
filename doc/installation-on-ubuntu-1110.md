在Ubuntu 11.10上安装
----------------------------

注意：这里推荐使用LuaJIT 2.0 或 LuaJIT 2.1替换掉标准Lua 5.1解析器。

如果标准解析器是必须的，执行下面的命令完成在Ubuntu上的安装：

```bash

 apt-get install -y lua5.1 liblua5.1-0 liblua5.1-0-dev
```

所有反馈都应该是正确的，除了一条小"纠结"：

`liblua.so`库在liblua5.1包中已经发生改变，只能使用`liblua5.1.so`，并且需要被连接到`/usr/lib`，这样才可以在configure执行阶段被找到。

```bash

 ln -s /usr/lib/x86_64-linux-gnu/liblua5.1.so /usr/lib/liblua.so
```

[返回目录](#table-of-contents)

> English source:

Installation on Ubuntu 11.10
----------------------------

Note that it is recommended to use LuaJIT 2.0 or LuaJIT 2.1 instead of the standard Lua 5.1 interpreter wherever possible.

If the standard Lua 5.1 interpreter is required however, run the following command to install it from the Ubuntu repository:

```bash

 apt-get install -y lua5.1 liblua5.1-0 liblua5.1-0-dev
```

Everything should be installed correctly, except for one small tweak.

Library name `liblua.so` has been changed in liblua5.1 package, it only comes with `liblua5.1.so`, which needs to be symlinked to `/usr/lib` so it could be found during the configuration process.

```bash

 ln -s /usr/lib/x86_64-linux-gnu/liblua5.1.so /usr/lib/liblua.so
```

[Back to TOC](#table-of-contents)