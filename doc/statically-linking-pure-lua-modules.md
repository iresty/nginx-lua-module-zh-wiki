静态链接纯净Lua模块
===================================

当使用LuaJIT 2.x时，静态链接一个纯净的Lua字节码模块到可运行的Nginx中是可行的。

首先你要使用`luajit`，把`.lua`的Lua模块编译成`.o`的目标文件（包含导出的字节码数据），然后链接这些`.o`文件到Nginx的构造环境。

用下面这个小例子来印证一下。这里我们的`.lua`文件使用`foo.lua`名字：

```lua

 -- foo.lua
 local _M = {}

 function _M.go()
     print("Hello from foo")
 end

 return _M
```

我们把`.lua`文件编译成`foo.o`文件：

    /path/to/luajit/bin/luajit -bg foo.lua foo.o

这里重要的是`.lua`文件名，它决定了这个模块在业务Lua中是如何使用的。文件名`foo.o`除了扩展名是`.o`以外其他都不重要（只是用来告诉`luajit`什么输出格式）。如果你想对字节码输出去掉Lua调试信息，你可以指明`-b`选项替代原有的`-bg`。

然后你构建Nginx或者OpenResty，对`./configure`脚本添加`--with-ld-opt="foo.o"`选项：

```bash

 ./configure --with-ld-opt="/path/to/foo.o" ...
```

最后，你就可以按照下面的示例，在Lua代码中调用了：

```lua

 local foo = require "foo"
 foo.go()
```

并且，这段代码不在依赖外部的`foo.lua`文件，因为它已经被编译到了`nginx`程序中。

通过`require`调用Lua模块，如果你想使用点方法，如下：

```lua

 local foo = require "resty.foo"
```

在你使用`luajit`命令行工具把他编译成`.o`文件之前，你需要把`foo.lua`文件重命名为`resty_foo.lua`。

`.lua`文件编译成`.o`文件，构建nginx + ngx_lua，这两者使用相同版本的LuaJIT是至关重要的。这是因为LuaJIT字节码格式在不同版本之间可能是不兼容的。当字节码文件出现了不兼容情况，你将看到一行Lua运行时错误信息：没找到Lua模块。

当你拥有多个`.lua`文件需要链接，你可以一次指明所有的`.o`文件，并赋值给`--with-ld-opt`选项即可，参考：

```bash

 ./configure --with-ld-opt="/path/to/foo.o /path/to/bar.o" ...
```

If you have just too many `.o` files, then it might not be feasible to name them all in a single command. In this case, you can build a static library (or archive) for your `.o` files, as in

```bash

 ar rcus libmyluafiles.a *.o
```

then you can link the `myluafiles` archive as a whole to your nginx executable:

```bash

 ./configure \
     --with-ld-opt="-L/path/to/lib -Wl,--whole-archive -lmyluafiles -Wl,--no-whole-archive"
```

where `/path/to/lib` is the path of the directory containing the `libmyluafiles.a` file. It should be noted that the linker option `--whole-archive` is required here because otherwise our archive will be skipped because no symbols in our archive are mentioned in the main parts of the nginx executable.

[Back to TOC](#table-of-contents)

> English source:

Statically Linking Pure Lua Modules
===================================

When LuaJIT 2.x is used, it is possible to statically link the bytecode of pure Lua modules into the Nginx executable.

Basically you use the `luajit` executable to compile `.lua` Lua module files to `.o` object files containing the exported bytecode data, and then link the `.o` files directly in your Nginx build.

Below is a trivial example to demonstrate this. Consider that we have the following `.lua` file named `foo.lua`:

```lua

 -- foo.lua
 local _M = {}

 function _M.go()
     print("Hello from foo")
 end

 return _M
```

And then we compile this `.lua` file to `foo.o` file:

    /path/to/luajit/bin/luajit -bg foo.lua foo.o

What matters here is the name of the `.lua` file, which determines how you use this module later on the Lua land. The file name `foo.o` does not matter at all except the `.o` file extension (which tells `luajit` what output format is used). If you want to strip the Lua debug information from the resulting bytecode, you can just specify the `-b` option above instead of `-bg`.

Then when building Nginx or OpenResty, pass the `--with-ld-opt="foo.o"` option to the `./configure` script:

```bash

 ./configure --with-ld-opt="/path/to/foo.o" ...
```

Finally, you can just do the following in any Lua code run by ngx_lua:

```lua

 local foo = require "foo"
 foo.go()
```

And this piece of code no longer depends on the external `foo.lua` file any more because it has already been compiled into the `nginx` executable.

If you want to use dot in the Lua module name when calling `require`, as in

```lua

 local foo = require "resty.foo"
```

then you need to rename the `foo.lua` file to `resty_foo.lua` before compiling it down to a `.o` file with the `luajit` command-line utility.

It is important to use exactly the same version of LuaJIT when compiling `.lua` files to `.o` files as building nginx + ngx_lua. This is because the LuaJIT bytecode format may be incompatible between different LuaJIT versions. When the bytecode format is incompatible, you will see a Lua runtime error saying that the Lua module is not found.

When you have multiple `.lua` files to compile and link, then just specify their `.o` files at the same time in the value of the `--with-ld-opt` option. For instance,

```bash

 ./configure --with-ld-opt="/path/to/foo.o /path/to/bar.o" ...
```

If you have just too many `.o` files, then it might not be feasible to name them all in a single command. In this case, you can build a static library (or archive) for your `.o` files, as in

```bash

 ar rcus libmyluafiles.a *.o
```

then you can link the `myluafiles` archive as a whole to your nginx executable:

```bash

 ./configure \
     --with-ld-opt="-L/path/to/lib -Wl,--whole-archive -lmyluafiles -Wl,--no-whole-archive"
```

where `/path/to/lib` is the path of the directory containing the `libmyluafiles.a` file. It should be noted that the linker option `--whole-archive` is required here because otherwise our archive will be skipped because no symbols in our archive are mentioned in the main parts of the nginx executable.

[Back to TOC](#table-of-contents)