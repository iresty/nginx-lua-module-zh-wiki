## 使用摘要

```nginx
 # 设置纯Lua扩展库PATH(';;' is the default path):
 lua_package_path '/foo/bar/?.lua;/blah/?.lua;;';

 # 设置C扩展Lua模块，CPATH(can also use ';;'):
 lua_package_cpath '/bar/baz/?.so;/blah/blah/?.so;;';

 server {
     location /inline_concat {
         # default_type指令设置默认MIME头:
         default_type 'text/plain';

         set $a "hello";
         set $b "world";
         # 内嵌lua代码
         set_by_lua $res "return ngx.arg[1]..ngx.arg[2]" $a $b;
         echo $res;
     }

     location /rel_file_concat {
         set $a "foo";
         set $b "bar";
         # 脚本路径是nginx prefix的相对地址
         # 文件 $ngx_prefix/conf/concat.lua 内容:
         #
         #    return ngx.arg[1]..ngx.arg[2]
         #
         set_by_lua_file $res conf/concat.lua $a $b;
         echo $res;
     }

     location /abs_file_concat {
         set $a "fee";
         set $b "baz";
         # 绝对地址直接使用
         set_by_lua_file $res /usr/nginx/conf/concat.lua $a $b;
         echo $res;
     }

     location /lua_content {
         # 通过default_type设置默认的MIME type
         default_type 'text/plain';

         content_by_lua "ngx.say('Hello,world!')";
     }

     location /nginx_var {
         # 通过default_type设置默认的MIME type
         default_type 'text/plain';

         # 外部尝试访问： /nginx_var?a=hello,world
         content_by_lua "ngx.print(ngx.var['arg_a'], '\\n')";
     }

     location /request_body {
          # 强制读取请求 body (默认是关)
          lua_need_request_body on;
          client_max_body_size 50k;
          client_body_buffer_size 50k;

          content_by_lua 'ngx.print(ngx.var.request_body)';
     }

     # 在子查询中直接发起非阻塞 IO 调用
     location /lua {
         # 通过default_type设置默认的MIME type：
         default_type 'text/plain';

         content_by_lua '
             local res = ngx.location.capture("/some_other_location")
             if res.status == 200 then
                 ngx.print(res.body)
             end';
     }

     # GET /recur?num=5
     location /recur {
         # 通过default_type设置默认的MIME type：
         default_type 'text/plain';

         content_by_lua '
            local num = tonumber(ngx.var.arg_num) or 0

            if num > 50 then
                ngx.say("num too big")
                return
            end

            ngx.say("num is: ", num)

            if num > 0 then
                res = ngx.location.capture("/recur?num=" .. tostring(num - 1))
                ngx.print("status=", res.status, " ")
                ngx.print("body=", res.body)
            else
                ngx.say("end")
            end
            ';
     }

     location /foo {
         rewrite_by_lua '
             res = ngx.location.capture("/memc",
                 { args = { cmd = "incr", key = ngx.var.uri } }
             )
         ';

         proxy_pass http://blah.blah.com;
     }

     location /blah {
         access_by_lua '
             local res = ngx.location.capture("/auth")

             if res.status == ngx.HTTP_OK then
                 return
             end

             if res.status == ngx.HTTP_FORBIDDEN then
                 ngx.exit(res.status)
             end

             ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR)
         ';

         # proxy_pass/fastcgi_pass/postgres_pass/...
     }

     location /mixed {
         rewrite_by_lua_file /path/to/rewrite.lua;
         access_by_lua_file /path/to/access.lua;
         content_by_lua_file /path/to/content.lua;
     }

     # 在代码路径中使用nginx变量
     * use nginx var in code path
     # 注意： nginx var 的变量一定要谨慎，否则将会带来非常大的风险
     location ~ ^/app/([-_a-zA-Z0-9/]+) {
         set $path $1;
         content_by_lua_file /path/to/lua/app/root/$path.lua;
     }

     location / {
        lua_need_request_body on;

        client_max_body_size 100k;
        client_body_buffer_size 100k;

        access_by_lua '
            -- 检测客户端 IP 地址是否在我们的黑名单中
            if ngx.var.remote_addr == "132.5.72.3" then
                ngx.exit(ngx.HTTP_FORBIDDEN)
            end

            -- 检测客户端 body 数据是否包含敏感词汇
            if ngx.var.request_body and
                     string.match(ngx.var.request_body, "fsck")
            then
                return ngx.redirect("/terms_of_use.html")
            end

            -- tests passed
        ';

        # proxy_pass/fastcgi_pass/etc settings
     }
 }
```


> English Version

```nginx
 # set search paths for pure Lua external libraries (';;' is the default path):
 lua_package_path '/foo/bar/?.lua;/blah/?.lua;;';

 # set search paths for Lua external libraries written in C (can also use ';;'):
 lua_package_cpath '/bar/baz/?.so;/blah/blah/?.so;;';

 server {
     location /inline_concat {
         # MIME type determined by default_type:
         default_type 'text/plain';

         set $a "hello";
         set $b "world";
         # inline Lua script
         set_by_lua $res "return ngx.arg[1]..ngx.arg[2]" $a $b;
         echo $res;
     }

     location /rel_file_concat {
         set $a "foo";
         set $b "bar";
         # script path relative to nginx prefix
         # $ngx_prefix/conf/concat.lua contents:
         #
         #    return ngx.arg[1]..ngx.arg[2]
         #
         set_by_lua_file $res conf/concat.lua $a $b;
         echo $res;
     }

     location /abs_file_concat {
         set $a "fee";
         set $b "baz";
         # absolute script path not modified
         set_by_lua_file $res /usr/nginx/conf/concat.lua $a $b;
         echo $res;
     }

     location /lua_content {
         # MIME type determined by default_type:
         default_type 'text/plain';

         content_by_lua "ngx.say('Hello,world!')";
     }

     location /nginx_var {
         # MIME type determined by default_type:
         default_type 'text/plain';

         # try access /nginx_var?a=hello,world
         content_by_lua "ngx.print(ngx.var['arg_a'], '\\n')";
     }

     location /request_body {
          # force reading request body (default off)
          lua_need_request_body on;
          client_max_body_size 50k;
          client_body_buffer_size 50k;

          content_by_lua 'ngx.print(ngx.var.request_body)';
     }

     # transparent non-blocking I/O in Lua via subrequests
     location /lua {
         # MIME type determined by default_type:
         default_type 'text/plain';

         content_by_lua '
             local res = ngx.location.capture("/some_other_location")
             if res.status == 200 then
                 ngx.print(res.body)
             end';
     }

     # GET /recur?num=5
     location /recur {
         # MIME type determined by default_type:
         default_type 'text/plain';

         content_by_lua '
            local num = tonumber(ngx.var.arg_num) or 0

            if num > 50 then
                ngx.say("num too big")
                return
            end

            ngx.say("num is: ", num)

            if num > 0 then
                res = ngx.location.capture("/recur?num=" .. tostring(num - 1))
                ngx.print("status=", res.status, " ")
                ngx.print("body=", res.body)
            else
                ngx.say("end")
            end
            ';
     }

     location /foo {
         rewrite_by_lua '
             res = ngx.location.capture("/memc",
                 { args = { cmd = "incr", key = ngx.var.uri } }
             )
         ';

         proxy_pass http://blah.blah.com;
     }

     location /blah {
         access_by_lua '
             local res = ngx.location.capture("/auth")

             if res.status == ngx.HTTP_OK then
                 return
             end

             if res.status == ngx.HTTP_FORBIDDEN then
                 ngx.exit(res.status)
             end

             ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR)
         ';

         # proxy_pass/fastcgi_pass/postgres_pass/...
     }

     location /mixed {
         rewrite_by_lua_file /path/to/rewrite.lua;
         access_by_lua_file /path/to/access.lua;
         content_by_lua_file /path/to/content.lua;
     }

     # use nginx var in code path
     # WARNING: contents in nginx var must be carefully filtered,
     # otherwise there'll be great security risk!
     location ~ ^/app/([-_a-zA-Z0-9/]+) {
         set $path $1;
         content_by_lua_file /path/to/lua/app/root/$path.lua;
     }

     location / {
        lua_need_request_body on;

        client_max_body_size 100k;
        client_body_buffer_size 100k;

        access_by_lua '
            -- check the client IP address is in our black list
            if ngx.var.remote_addr == "132.5.72.3" then
                ngx.exit(ngx.HTTP_FORBIDDEN)
            end

            -- check if the request body contains bad words
            if ngx.var.request_body and
                     string.match(ngx.var.request_body, "fsck")
            then
                return ngx.redirect("/terms_of_use.html")
            end

            -- tests passed
        ';

        # proxy_pass/fastcgi_pass/etc settings
     }
 }
```
