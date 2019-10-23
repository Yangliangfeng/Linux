###    关于OpenResty的使用

* OpenResty基本语法
```
1. pairs与ipairs的区别

   ipairs只能打印table有序序列（数组），pairs能打印table中的所有元素（哈希和数组）
   
2. table.remove 删除指定元素

   table.remove只能删除table中的带下标的有序序列（数组）
   
   删除哈希值只能用:table.first = nil,其中first是table其中的key
   
3. table.concat 拼接字符串
   
   只对table的有序序列有效
```
* OpenResty第三方客户端
```
1. OpenResty没有提供Http客户端，需要使用第三方提供，第三方库：https://github.com/pintsized/lua-resty-http

2. 配置
   将 lua-resty-http/lib/resty/ 目录下的 http.lua 和 http_headers.lua 两个文件拷贝到
   
   /usr/local/openresty/lualib/resty 目录下
   
3. 配置https协议
  server虚拟主机模块设置：
  lua_ssl_verify_depth 2;
  lua_ssl_trusted_certificate "/etc/ssl/certs/ca-bundle.crt";

4. 更多openresty模块：https://github.com/bungle/awesome-resty

```
* OpenResty发起http请求

```
1. 场景：
   用户浏览器请求url访问到nginx服务器，但此请求业务需要再次请求其他业务；
   如用户请求订单服务获取订单详情，可订单详情中需要返回商品信息，也就需要再请求商品服务获取商品信息；
   这样就需要nginx需要有发起http请求的能力，而不是让用户浏览器在请求商品访问
   
   nginx服务发起http请求区分内部请求 和 外部请求
   
2. 内部请求
   1）capture请求方法
   语法： ngx.location.capture(uri,{options？});
   options可以传参数和设置请求方式
   
   案例：  
      local res = ngx.location.capture("/product",{
         method = ngx.HTTP_GET,   #请求方式
         args = {a=1,b=2},  #get方式传参数
         body = "c=3&d=4" #post方式传参数
      });
    
   ngx.location.capture 方法就是发起http的请求，但是它只能请求 内部服务，不能直接请求外部服务
   
   这边有一种情况，这样的定义，用户用浏览器直接请求商品服务也照样请求

   可很多时候我们会要求商品请求 是不对外暴露的，也就是用户无法直接访问商品服务请求。
   那我们只要在内部请求那边加上一个关键字，internal 就可以了
   location /product {  #商品服务请求
       internal;
      echo "商品请求";
   }
   这样直接访问就报404错误了
   
2. capture_multi 并发请求

   语法：res1,res2, ... = ngx.location.capture_multi({ 
								{uri, options?}, 
								{uri, options?}, 
								...
						})
3. 外部请求
   location /product {
      internal;
      proxy_pass "https://s.taobao.com/search?q=iphone";
   }
   location /order {
      content_by_lua_block {
      local res = ngx.location.capture("/product");
      ngx.say(res.status)
      ngx.say(res.body)
        }
   }
   在商品服务那边用的proxy_pass 请求外部http请求，这样就达到了请求外部http的目的。
   
```
* OpenResty执行流程

![](https://github.com/Yangliangfeng/Linux/raw/master/file/images/openresty.png)

* 重写rewrite阶段
```
1. if指令

   语法：if (condition){...}
   作用域：server,location
   
   上面的if和(之间需要留空格，否则会报错
   
   1）条件可以为一个变量
      如果一个变量名进行条件判断，空字符串'' 或 字符串为'0',都表示为假 false
      
   2）条件为表达式
      正则表达式匹配：
      = ,!= 比较的一个变量和字符串
      ~：与指定正则表达式模式匹配时返回“真”，判断匹配与否时区分字符大小写；
      ~*：与指定正则表达式模式匹配时返回“真”，判断匹配与否时不区分字符大小写；
      !~：与指定正则表达式模式不匹配时返回“真”，判断匹配与否时区分字符大小写；
      !~*：与指定正则表达式模式不匹配时返回“真”，判断匹配与否时不区分字符大小写；
      
   3）文件及目录匹配判断
      -f, !-f：判断指定的路径是否为存在且为文件；
      -d, !-d：判断指定的路径是否为存在且为目录；
      -e, !-e：判断指定的路径是否存在，文件或目录均可；
      -x, !-x：判断指定路径的文件是否存在且可执行；
      
    注意:
      1)nginx if 没有对应的else
      2)if 表达式中 是不能用  &&  ||

2. Rewrite规则
   
   1）301 与 302区别：
      301 redirect: 301 代表永久性转移(Permanently Moved)
      302 redirect: 302 代表暂时性转移(Temporarily Moved)
      
      301表示旧地址A的资源已经被永久地移除了（这个资源不可访问了），搜索引擎在抓取新内容的同时也将旧的网址交换为
      
      重定向之后的网址；302表示旧地址A的资源还在（仍然可以访问），这个重定向只是临时地从旧地址A跳转到地址B，搜索
      
      引擎会抓取新的内容而保存旧的网址
      
   2）rewrite指令
   
      语法：rewrite regex replacement [flag];
      作用：使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向
      作用域：server，location，if，并且只能对域名后边的除去传递的参数外的字符串起作用
      例如http://www.a.com/api/index.jsp?id=1&u=str 只对api/index.jsp重写
      
      regex：perl兼容正则表达式语句进行规则匹配
      replacement：将正则匹配的内容替换成replacement
      flag标记：rewrite支持的flag标记
      
      正则表达式元字符:
      .     :匹配除换行符以外的任意字符
      ?     :重复0次或1次
      +     :重复1次或更多次
      *     :重复0次或更多次
      \d    :匹配数字
      ^     :匹配字符串的开始字符
      $     :匹配字符串的结束字符
      {n}   :重复n次
      {n,}  :重复n次或更多次
      [c]   :匹配单个字符c
      [a-z] :匹配a-z小写字母的任意一个
      
      在rewrite中，如果使用小括号()，那么在小括号之间匹配的内容，可以在后面通过$1来引用，
      $2表示的是前面第二个()里的内容
      
      flag标志位：
         last : 相当于Apache的[L]标记，表示完成rewrite
         break : 停止执行当前虚拟主机的后续rewrite指令集
         redirect : 返回302临时重定向，地址栏会显示跳转后的地址
         permanent : 返回301永久重定向，地址栏会显示跳转后的地址
         
      last 和 break 区别：
         last： 停止当前这个请求，并根据rewrite匹配的规则重新发起一个请求。新请求又从第一阶段开始执行
         break：相对last，break并不会重新发起一个请求，只是跳过当前的rewrite阶段，并执行本请求后续的执行阶段
         
         break与last都停止处理后续rewrite指令集，不同之处在与last会重新发起新的请求，
         而break不会。当请求break时，如匹配内容存在的话，可以直接请求成功
         
     案例：
         location /break/ {  
            rewrite ^/break/(.*) /test/$1 break;  #----break；/test/$1 会在根目录下的查找/test/$1文件
         }  

         location /last/ {  
            rewrite ^/last/(.*) /test/$1 last;    #----last；/test/$1 会重新走一遍location匹配流程
         }

         location /test/ {  
            echo "test page";
         }
         
     执行顺序是：
         a）执行server块的rewrite指令
         b）执行location匹配
         c）执行选定的location中的rewrite指令
         如果其中某步URI被重写，则重新循环执行a-c，直到找到真实存在的文件；循环超过10次，
         则返回500 Internal Server Error错误。
         
3. rewrite_by_lua
   
   语法：rewrite_by_lua <lua-script-str>
   语境：http、server、location、location if
   阶段：rewrite tail
   
   执行内部URL重写或者外部重定向，典型的如伪静态化的URL重写。其默认执行在rewrite处理阶段的最后
   
   1）ngx.redirect ---重定向
      语法：ngx.redirect(uri, status?)
      该方法会给客户端返回一个301/302重定向，具体是301还是302取决于设定的status值，
      如果不指定status值，默认是返回302
      
   2）ngx.req.set_uri ---内部重写
      语法： ngx.req.set_uri(uri, jump?)
      
      通过参数uri重写当前请求的uri；参数jump，表明是否进行locations的重新匹配。
      当jump为true时，调用ngx.req.set_uri后，nginx将会根据修改后的uri，重新匹配新的locations；
      如果jump为false，将不会进行locations的重新匹配，而仅仅是修改了当前请求的URI而已。jump的默认值为false
      
      rewrite ^ /lua_rewrite_3;             等价于  ngx.req.set_uri("/lua_rewrite_3", false);
      rewrite ^ /lua_rewrite_3 break;       等价于  ngx.req.set_uri("/lua_rewrite_3", false); 
      rewrite ^ /lua_rewrite_4 last;        等价于  ngx.req.set_uri("/lua_rewrite_4", true);
      
      案例：
         location /foo {
            rewrite_by_lua_block {
                ngx.req.set_uri_args({a = 1, b = 2});//如果ngx.req.set_uri()为true，ngx.req.set_uri_args放在前面
                ngx.req.set_uri("/bar/index.html", true);
            }
         }
   
```
