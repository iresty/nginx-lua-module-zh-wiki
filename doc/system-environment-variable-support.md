系统环境变量的支持
===================================

如果你想访问系统环境变量，例如`foo`，在Lua中通过标准Lua API[os.getenv](http://www.lua.org/manual/5.1/manual.html#pdf-os.getenv)，同时你应当在`nginx.conf`文件的[环境小节](http://nginx.org/en/docs/ngx_core_module.html#env)中列出环境变量。例如：

```nginx

 env foo;
```

[返回目录](#table-of-contents)

> English source:

System Environment Variable Support
===================================

If you want to access the system environment variable, say, `foo`, in Lua via the standard Lua API [os.getenv](http://www.lua.org/manual/5.1/manual.html#pdf-os.getenv), then you should also list this environment variable name in your `nginx.conf` file via the [env directive](http://nginx.org/en/docs/ngx_core_module.html#env). For example,

```nginx

 env foo;
```

[Back to TOC](#table-of-contents)
