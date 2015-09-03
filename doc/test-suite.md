测试套件
==========

为了运行测试套件，依赖下面这些条件：

* Nginx version >= 1.4.2

* Perl modules:
    * Test::Nginx: <https://github.com/openresty/test-nginx>

* Nginx 模块:
    * [ngx_devel_kit](https://github.com/simpl/ngx_devel_kit)
    * [ngx_set_misc](https://github.com/openresty/set-misc-nginx-module)
    * [ngx_auth_request](http://mdounin.ru/files/ngx_http_auth_request_module-0.2.tar.gz) (this is not needed if you're using Nginx 1.5.4+.
    * [ngx_echo](https://github.com/openresty/echo-nginx-module)
    * [ngx_memc](https://github.com/openresty/memc-nginx-module)
    * [ngx_srcache](https://github.com/openresty/srcache-nginx-module)
    * ngx_lua (i.e., this module)
    * [ngx_lua_upstream](https://github.com/openresty/lua-upstream-nginx-module)
    * [ngx_headers_more](https://github.com/openresty/headers-more-nginx-module)
    * [ngx_drizzle](https://github.com/openresty/drizzle-nginx-module)
    * [ngx_rds_json](https://github.com/openresty/rds-json-nginx-module)
    * [ngx_coolkit](https://github.com/FRiCKLE/ngx_coolkit)
    * [ngx_redis2](https://github.com/openresty/redis2-nginx-module)

`configure`时添加的这些模块顺序是非常重要的。因为在filter链中不同的过滤模块位置决定最终输出。正确的添加顺序如上所示。

* 第三方 Lua 库:
    * [lua-cjson](http://www.kyne.com.au/~mark/software/lua-cjson.php)

* 应用:
    * mysql: 创建数据库 'ngx_test', 对用户'ngx_test'赋予所有权限，密码也是'ngx_test'。
    * memcached: 监听默认端口，11211.
    * redis: 监听默认端口, 6379.

查看[developer build script](https://github.com/openresty/lua-nginx-module/blob/master/util/build2.sh)内容，在搭建测试环境时确定更多细节。

在默认的测试模式下启动测试套件：

    cd /path/to/lua-nginx-module
    export PATH=/path/to/your/nginx/sbin:$PATH
    prove -I/path/to/test-nginx/lib -r t

运行指定的测试文件：

    cd /path/to/lua-nginx-module
    export PATH=/path/to/your/nginx/sbin:$PATH
    prove -I/path/to/test-nginx/lib t/002-content.t t/003-errors.t

在一个特别的测试文件中，运行指定的测试块，对你需要进行块测试部分添加一行`--- ONLY`信息，并使用`prove`工具运行这个`.t`文件。

此外，还有其他各种测试方式，基于mockeagain， valgrind 等。参考[Test::Nginx documentation](http://search.cpan.org/perldoc?Test::Nginx)，有更多不同高级测试方式的资料。也可以看看在Amazon EC2的Nginx集群测试报告 <http://qa.openresty.org.> 。

[返回目录](#table-of-contents)

> English source:
Test Suite
==========

The following dependencies are required to run the test suite:

* Nginx version >= 1.4.2

* Perl modules:
    * Test::Nginx: <https://github.com/openresty/test-nginx>

* Nginx modules:
    * [ngx_devel_kit](https://github.com/simpl/ngx_devel_kit)
    * [ngx_set_misc](https://github.com/openresty/set-misc-nginx-module)
    * [ngx_auth_request](http://mdounin.ru/files/ngx_http_auth_request_module-0.2.tar.gz) (this is not needed if you're using Nginx 1.5.4+.
    * [ngx_echo](https://github.com/openresty/echo-nginx-module)
    * [ngx_memc](https://github.com/openresty/memc-nginx-module)
    * [ngx_srcache](https://github.com/openresty/srcache-nginx-module)
    * ngx_lua (i.e., this module)
    * [ngx_lua_upstream](https://github.com/openresty/lua-upstream-nginx-module)
    * [ngx_headers_more](https://github.com/openresty/headers-more-nginx-module)
    * [ngx_drizzle](https://github.com/openresty/drizzle-nginx-module)
    * [ngx_rds_json](https://github.com/openresty/rds-json-nginx-module)
    * [ngx_coolkit](https://github.com/FRiCKLE/ngx_coolkit)
    * [ngx_redis2](https://github.com/openresty/redis2-nginx-module)

The order in which these modules are added during configuration is important because the position of any filter module in the
filtering chain determines the final output, for example. The correct adding order is shown above.

* 3rd-party Lua libraries:
    * [lua-cjson](http://www.kyne.com.au/~mark/software/lua-cjson.php)

* Applications:
    * mysql: create database 'ngx_test', grant all privileges to user 'ngx_test', password is 'ngx_test'
    * memcached: listening on the default port, 11211.
    * redis: listening on the default port, 6379.

See also the [developer build script](https://github.com/openresty/lua-nginx-module/blob/master/util/build2.sh) for more details on setting up the testing environment.

To run the whole test suite in the default testing mode:

    cd /path/to/lua-nginx-module
    export PATH=/path/to/your/nginx/sbin:$PATH
    prove -I/path/to/test-nginx/lib -r t


To run specific test files:

    cd /path/to/lua-nginx-module
    export PATH=/path/to/your/nginx/sbin:$PATH
    prove -I/path/to/test-nginx/lib t/002-content.t t/003-errors.t


To run a specific test block in a particular test file, add the line `--- ONLY` to the test block you want to run, and then use the `prove` utility to run that `.t` file.

There are also various testing modes based on mockeagain, valgrind, and etc. Refer to the [Test::Nginx documentation](http://search.cpan.org/perldoc?Test::Nginx) for more details for various advanced testing modes. See also the test reports for the Nginx test cluster running on Amazon EC2: <http://qa.openresty.org.>

[Back to TOC](#table-of-contents)
