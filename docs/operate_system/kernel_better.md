# 常见内核优化项

## 内核优化案例
# [Time wait bucket table overflow 报错](./kernel_better_tw_bucket.md)



|参数|备注|默认值|
|:------:|:---------------------:|:----:|
|net.ipv4.tcp_max_syn_backlog = 8192|表示 syn 队列长度，默认1024，可以容纳更多的网络连接数使系统可以处理更多的并发连接|1024|
|net.ipv4.tcp_tw_reuse = 1|打开 time-wait 套接字重用功能，对于高并发连接数巨大的web服务器非常有用|0 表示关闭|
|net.ipv4.tcp_tw_recycle = 1|开启 tcp 连接中的 time-wait sockets 的快速回收功能，一般和上一条tw重用一起使用|0 表示关闭|
|net.ipv4.tcp_fin_timeout = 30 |tcp_fin_timeout 是 FIN_WAIT_2 状态的时长；在高并发的 web 服务器上，减少FIN_WAIT_2 时长，可以避免无效数据报占用大量内存|默认是 60 单位：s|

#### 参考文档
* https://blog.csdn.net/preamble_1/article/details/70849310
* https://help.aliyun.com/knowledge_detail/41334.html

常见内核优化项
```
# Controls IP packet forwarding
net.ipv4.ip_forward = 0

# Controls source route verification
net.ipv4.conf.default.rp_filter = 1

# Do not accept source routing
net.ipv4.conf.default.accept_source_route = 0

# Controls the System Request debugging functionality of the kernel
kernel.sysrq = 0

# Controls whether core dumps will append the PID to the core filename.
# Useful for debugging multi-threaded applications.
kernel.core_uses_pid = 1

# Controls the use of TCP syncookies
net.ipv4.tcp_syncookies = 1

# Controls the default maxmimum size of a mesage queue
kernel.msgmnb = 65536

# Controls the maximum size of a message, in bytes
kernel.msgmax = 65536

# Controls the maximum shared segment size, in bytes
kernel.shmmax = 68719476736

# Controls the maximum number of shared memory segments, in pages
kernel.shmall = 4294967296
kernel.panic = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.default.arp_ignore = 1
net.ipv4.conf.default.arp_announce = 2
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.ipv4.ip_no_pmtu_disc = 1
#==========weihua2====================
net.ipv4.tcp_rmem = 8192        87380   8738000
net.ipv4.tcp_wmem = 4096        65536   6553600
net.ipv4.ip_local_port_range = 10000     65000
#优化系统套接字缓冲区
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
#=====================================
#打开TIME-WAIT套接字重用功能，对于存在大量连接的Web服务器非常有效。
net.ipv4.tcp_tw_recycle=1
net.ipv4.tcp_tw_reuse=1

#减少处于FIN-WAIT-2连接状态的时间，使系统可以处理更多的连接。
net.ipv4.tcp_fin_timeout=30

#减少TCP KeepAlive连接侦测的时间，使系统可以处理更多的连接。
net.ipv4.tcp_keepalive_time=1800

#增加TCP SYN队列长度，使系统可以处理更多的并发连接。
net.ipv4.tcp_max_syn_backlog=8192

#使系统内存使用更倾向于使用物理内存
#
vm.zone_reclaim_mode=0
#配置vip时需要修改的参数
#net.ipv4.conf.all.arp_ignore=1
#net.ipv4.conf.all.arp_announce=2

net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384

#防止过早使用swap
vm.swappiness = 0

net.core.netdev_max_backlog = 16384
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

