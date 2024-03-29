#### Linux基本操作

* 获得CPU详细信息

`
cat /proc/cpuinfo
`

* 查看物理CPU的个数

`
cat /proc/cpuinfo |grep "physical id"|sort |uniq|wc -l
`

* 查看逻辑CPU的个数

`
cat /proc/cpuinfo |grep "processor"|wc -l 
`

* 显示Linux所有的系统参数

`
sysctl -a
`

* 从指定的配置文件中加载系统参数，如不指定即从/etc/sysctl.conf中加载

`
sysctl -p
`
* 查看操作系统可使用的最大句柄数

`
sysctl -a | grep fs.file-max
`
* 查看当前的使用情况

`
sysctl -a | grep fs.file-nr    返回当前已分配的，正在使用的，上限
`
* 限制用户的句柄数

```
vim /etc/security/limits.conf
root soft nofiles 65535
root hard nofiles 65535   
notice: hard 必须大于等于soft的值
```
* 服务器端TCP三次握手的参数

```
1. net.ipv4.tcp_max_syn_backlog = 262144  //SYN队列未完成握手
2. net.core.somaxconn     //ACCEPT队列已完成握手 ---> 系统级最大的backlog队列长度
3. net.ipv4.tcp_retries1 = 3   //限制重传次数  ---> 达到上限后，更新路由缓存
4. net.ipv4.tcp_retries2 = 15  // 达到上限后，关闭tcp连接
```
* TCP的缓冲区
```
1. net.ipv4.tcp_rmem = 4096 87380 6291456   //读缓存最小值，默认值，最大值；单位为字节，覆盖net.core.rmem_max
2. net.ipv4.tcp_wmen = 4096 16384 4194304   //写缓存最小值，默认值，最大值；单位字节，覆盖net.core.wmen_max
3. net.ipv4.tcp_mem = 1541646  2055528 3083292   //系统无内存压力，启动压力模式阈值，最大值；单位为页的数量
4. net.ipv4.tcp_moderate_rcvbuf = 1    // 开启自动调整缓存模式
```
* TCP的Nagle算法
```
优点：
   1. 避免一个连接上同时存在大量小报文
   2. 最多只存在一个小报文
   3. 合并多个小报文一起发送
   4. 提高带宽的利用率
 吞吐量优先：启用Nagle算法， tcp_nodelay off
 低时延优先：禁用Nagle算法，tcp_nodelay on
```
* Linux 的TCP 的Keep-Alive功能
```
1. 应用场景：
   a. 检测实际断掉的连接
   b. 用户维护与客户端间的防火墙有活跃的包
   
2. 参数设置
   a. 发送心跳周期
      net.ipv4.tcp_keepalive_time = 7200
   b. 探测报发送间隔
      net.ipv4.tcp_keepalive_intvl = 75
   c. 探测包重试次数
      net.ipv4.tcp_keepalive_probcs = 9
 3. Nginx的TCP的KeepAlive
    so_keepalive = 30::10
    keepidle  keepintvl   keepcnt
```
### Linux 之Centos8 的 Nmcli配置网络
```
1. nmcli 命令服务
systemctl status NetworkManager

2. 查看网卡信息
   nmcli

3. 查看网络设备状态
 nmcli device status
 
4. 查看网络详细信息（ip + dns + gateway)
  nmcli device show ens32
  
5. 设置静态ip地址
   nmcli connection modify ens32 ipv4.address '192.168.1.35'
   
6. 设置DNS
   nmcli connection modify ens32 ipv4.dns '114.114.114.114'
   
7. 设置网关
   nmcli connection modify ens32 ipv4.gateway '192.168.1.1'
   
8. 设置IP地址为手动指定
   nmcli connection modify ens32 ipv4.method manual
   
 9. 设置IP地址为dhcp自动获取
   nmcli connection modify ens32 ipv4.method auto
   
 10. 设置开机自动连接
   nmcli connection modify ens32 connection.autoconnect yes
   nmcli connection modify ens32 connection.autoconnect no
   
 11. nmcli重新加载ifcfg-ens32配置文件
   nmcli connection reload
   
 12. 不重启系统让网卡生效的方法：
   nmcli connection up ens32
```
### Centos8之 Cockpit监控
```
1. Cockpit Web系统监控
   systemctl enable --now cockpit.socket
   
2. 开启cockpit服务：
   systemctl start cockpit.service

3. 查看cockpit状态
   systemctl status cockpit.service

4. web系统访问
   https://192.168.2.1:9090
```
