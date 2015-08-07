lua-variable-scope

在代码中导入模块时应注意一些细节，推介使用如下格式：

```
 local module_name = require('module_name')
```

而非：

```
require('module_name')
```

理由如下所述，从设计上讲，（Lua）全局环境的生命周期和一个Nginx的请求的生命周期是相同的。为了做到会话隔离，每个请求都有自己的Lua全局变量环境。Lua模块在第一次请求打到服务器上的时候被加载起来并且通过require函数添加到包管理列表中，以供后续代码逻辑引用，但是module()函数存在一个副作用，它会将**包含模块的函数的table设置到全局变量 _G 中**

>   
可以使用如下代码查看module()函数对全局变量的影响：  

```

	for n in pairs(_G) do print(n) end


```


但是这个全局变量在请求结束的时候会被清除，并且**每个子请求都有自己的全局变量环境**。因此，如果在子请求中如果访问父请求的全局变量环境会因为访问了一个nil值而得到一个Lua的异常。

总的来说，因为如下的三种原因在ngx_lua环境中使用Lua全局变量不是好主意：  

	1.滥用全局变量的副作用会对并发场景产生副作用，比如当使用者把这些变量看作是本地变量的时候；
	2.第二点只是经验之谈，Lua的全局变量需要向上查找一个全局环境。而这个全局环境只是另外一个table而已（_G）
	3.在上述的全局环境中（_G）的全局变量中只是一些占位的名称，这会导致出错很难排查

所以，我们极力推介在使用变量的时候总是使用local来定义以限定起生效范围是有理由的

使用工具(lua-releng tool)[https://github.com/openresty/nginx-devel-utils/blob/master/lua-releng]查找Lua代码中使用全局变量的地方
> $ lua-releng   
  
       Checking use of Lua global variables in file lib/foo/bar.lua ...  
        1       [1489]  SETGLOBAL       7 -1    ; contains
        55      [1506]  GETGLOBAL       7 -3    ; setvar
        3       [1545]  GETGLOBAL       3 -4    ; varexpand

上述输出说明文件lib/foo/bar.lua的1489行写入一个名为contains的全局变量；
1506行读取一个名为setvar的全局变量；
1545行读取一个名为varexpand的全局变量，


这个工具能保证Lua模块中的局部变量全部是用local关键字定义过的，否则将会抛出一个运行时库。这样能阻止类似变量这样的资源的竞争。理由请参考(Data Sharing within an Nginx Worker)[http://wiki.nginx.org/HttpLuaModule#Data_Sharing_within_an_Nginx_Worker]