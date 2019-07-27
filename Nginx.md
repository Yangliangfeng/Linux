# Nginx

* nginx下载源代码包

wget http://nginx.org/download/nginx-1.14.1.tar.gz

* 安装依赖
```
1. 安装Nginx需要用到pcre程序

wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.41.tar.gz

2. 安装epel扩展源

rpm -ivh http://dl.fedoraproject.org/pub/epel/epel-release-lastest-7.noarch.rpm

3. yum clean all   yum list

4.安装依赖包

yum -y install gcc gcc-c++ autoconf automake zlib zlib-devel openssl openssl-devel pcre*

5.解压pcre
tar zxvf pcre-8.41.tar.gz -C /usr/local/src/

6. 创建Nginx运行用户
useradd -M -s /sbin/nologin nginx
```
* Nginx编译参数
```
./configure --prefix=/usr/local/nginx \
--with-http_dav_module --with-http_stub_status_module --with-http_addition_module \
--with-http_sub_module --with-http_flv_module --with-http_mp4_module \
--with-pcre=/usr/local/src/pcre-8.41 --user=nginx --group=nginx
```
* Nginx加入环境变量
```
1. vim /etc/profile

export PATH=/usr/local/nginx/sbin:$PATH

2. source /etc/profile  //刷新全局环境变量
```
* 设置worker进程绑定到具体CPU
```
1. 查看cpu个数
grep -c processor /proc/cpuinfo

2.绑定
worker_processes 2; # 2核CPU的配置
worker_cpu_affinity 01 10;

worker_processes 4; # 4核CPU的配置
worker_cpu_affinity 0001 0010 0100 1000;

worker_processes 8; # 8核CPU的配置
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 1000000;
```
* Nginx维护
```
1. 检查配置文件是否有语法错误

nginx -t

2. 查看Nginx版本和配置参数

nginx -V 

3. 优雅而平滑的关闭nginx

nginx -s quit

4. 强制关闭Nginx

nginx -s stop

5. 启动nginx

nginx

6. 重新加载配置文件

nginx -s reload

7. 重新开始记录日志文件

nginx -s reopen

```
* nginx的组成
```
1. Nginx 二进制可执行文件

   由各模块源码编译出的一个文件
  
2. Nginx.conf 配置文件

   控制 Nginx 的行为
  
3. access.log 访问日志
  
   记录每一条 http 请求信息
  
4. error.log 错误日志

   定位问题
```
* nginx的热部署（升级新版本的nginx）
```
1. 升级nginx的新版本，只是替换nginx的可执行的二进制文件nginx

2. 备份老的nginx,拷贝新的nginx可执行的二进制文件到相应的目录下

3. 给正在运行的nginx主进程USR2这个信号：   kill -USR2 nginx主进程号
   此时，老的worker进程就不再监听80端口了

4. kill -WINCH nginx主进程号，优雅的关闭老版本的nginx
```
* nginx的日志备份
```
1. 把当前的日志复制都要备份的文件夹

2. 重新生成新的日志
nginx -s reload

```
* Nginx主进程支持的信号
```
1. QUIT  优雅的关闭
kill -QUIT  Nginx的主进程号
nginx -s quit

2. HUP   重新加载配置文件
kill -HUP  Nginx的主进程号
nginx -s reload

3. USR1  重新打开日志
kill -USR1  Nginx的主进程号

4. USR2  平滑升级
kill -USR2  Nginx的主进程号  nginx的版本需要升级的时候，不需要停止nginx，就能对nginx升级

5. WINCH
kill -WINCH nginx主进程号   配合USR2对nginx升级，优雅的关闭nginx旧版本的进程
```
* nginx的进程管理--->信号
```
nginx是一个多进程的程序，多进程之间通信是通过共享内存，信号等方式，但是做进程之间的管理时，只使用信号的方式。

Master进程：
1. 启动worker进程
2. 监控worker进程的CHLD信号（Linux操作系统中规定，子进程终止的时候，会向父进程发送CHLD信号）
   nginx中，如果worker进程出现故障，会向master进程发送CHLD信号，然后，master进程会重新拉取一个worker进程
3. 管理worker进程
4. 接收信号：TERM，INT（强制关闭nginx），QUIT，HUP，USER1，USR2，WINCH

Worker进程：
1. 接收信号：TERM，INT（强制关闭nginx），QUIT，USER1，WINCH

nginx命令行：
1. reload: HUP
2. reopen: USR1
3. stop: TERM
4. QUIT
```
* reload加载配置文件的流程
```
1. 向master发送HUP信号（reload命令）
2. master进程校验配置语法是否正确
3. master进程打开新的进程端口
4. master进程用新的配置启动新的worker子进程
5. master进程向老worker子进程发送quit信号
6. 老worker进程关闭监听句柄，处理完当前连接后
```
* reload加载流程:

![](https://github.com/Yangliangfeng/PHP/raw/master/Images/1560355226.png)

```
关于reload加载流程的注释：
   如果有一些请求出现了问题，客户端长时间没有处理，就会导致这个请求长时间占用老的worker进程，worker进程
   就会一直存在。只会影响当前老的worker进程，不会对新的进程产生影响。在新版本的nginx中，利用
   woker_shutdown_timeout来进行设置。
```

* worker子进程优雅的关闭

![](https://github.com/Yangliangfeng/PHP/raw/master/Images/nginx_shutdown.png)

* 补充知识点：TCP三次握手原理示例图

![](https://github.com/Yangliangfeng/PHP/raw/master/Images/tcp_connect.png)

* Nginx事件
```
1. nginx是由事件驱动的框架，这里的事件是指网络事件。Nginx每次连接会对应两个网络事件，一个“读事件”，一个“写事件”。
```
* DNS域名的配置
```
1. 223.5.5.5  （阿里DNS）
2. 114.114.114.114 (国内移动、电信和联通通用的DNS)
3. 180.76.76.76 (百度的公共DNS服务器)
4. resolver 114.114.114.114 223.5.5.5 valid=3600s（valid设置DNS缓存失效时间，自己根据情况判断，建议600以上）
```
* Nginx默认没有编译的参数
```
1. --with-http_realip_module

2. --with-http_request_module   //统一的用户权限验证系统

3. --with-http_sub_filter_module  //将响应中的指定字符串替换成新的字符串

4. --with-http_addition_module    //在响应前或者后增加内容，而增加的内容的方式是通过新增子请求的响应完成的

5. --with_http_geoip_module       //根据IP地址创建新变量
```








