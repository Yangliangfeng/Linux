###  基于Linux的openResty与Lua脚本

* openResty的基本使用
```
1. github的下载地址

https://github.com/openresty/openresty/releases

2. 安装

1）编译
./configure --with-luajit --with-pcre --with-http_gzip_static_module --with-http_realip_module 
--with-http_geoip_module --with-http_ssl_module  --with-http_stub_status_module

--with-http_gzip_static_module #静态文件压缩

--with-http_stub_status_module #监控nginx状态

--with-http_realip_module #通过这个模块允许我们改变客户端请求头中客户端IP地

址值(例如X-Real-IP 或 X-Forwarded-For)，意义在于能够使得后台服务器记录原始客户端的IP地址

--with-pcre #设置PCRE库（pcre pcre-devel）

--with-http_ssl_module #使用https协议模块。（openssl openssl-devel）

--with-http_geoip_module #增加了根据ip获得城市信息，经纬度等模块 （GeoIP-devel）

2）安装
make && make install

3）设置环境变量

# vi /etc/profile

export NGINX_HOME=/usr/local/openresty/nginx

export PATH=$PATH:$NGINX_HOME/sbin

# source /etc/profile ##生效

```
