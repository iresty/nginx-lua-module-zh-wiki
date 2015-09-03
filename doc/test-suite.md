测试套件
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
