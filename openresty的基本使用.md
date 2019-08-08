###    关于OpenResty的使用

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
* OpenResty执行流程

![](https://github.com/Yangliangfeng/Linux/raw/master/file/images/openresty.png)
