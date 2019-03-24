# php fpm 相关知识梳理
## php 原理

nginx 接收到一个web 请求之后通过 fastcgi_pass 转发给 fpm ,fpm 是一个进程管理器，分为 master 进程和 worker 进程两种， master 进程负责监听 fpm 端口,worker 进程负责处理请求，每个 worker 进程都有一个 PHP 解析器。

#### cgi 和 fastcgi 区别
* cgi 协议：每接收一个请求，fork 一个进程，处理完毕结束进程
* fastcgi 协议：每个进程可以处理多个请求，支持高并发，性能更强

### fpm 配置
```
[global]
pid = run/php-fpm.pid                #pid设置 
error_log = logs/php-fpm.log         #error log 设置
emergency_restart_threshold = 20     #
emergency_restart_interval = 60s     # 每60秒出现的php-cgi错误进程数超过20个就重启fpm
process_control_timeout = 0          #设置子进程接收主进程复用信号的超时时间
process.max = 2048                   #
daemonize = yes                      #后台执行fpm。调试的时候改为no
rlimit_files = 65535                 #主进程可打开文件描述符限制
rlimit_core = 67108864               #主进程的核数
events.mechanism = epoll             #event事件机制

[default]
user = www                           #启动进程时设置的用户和用户组
group = www
listen = 127.0.0.1:8999              #fpm监听的端口
listen.allowed_clients = 127.0.0.1   #允许fastcgi访问的ip白名单
pm = dynamic                         # fpm 进程的控制模式 分为 static 和 dynamic 两种
pm.max_children = 128                #静态模式下fpm进程数量
pm.start_servers = 10                #动态模式下fpm进程起始数量
pm.min_spare_servers = 5             #动态模式下fpm进程最小数量
pm.max_spare_servers = 20            #动态模式下fpm进程最大数量
pm.max_requests = 1500               #fpm子进程能够处理的最大请求数 当请求超过这个数值时 fpm 进程会重启，释放该进程占据的内存和资源
request_terminate_timeout = 30       #设置单个请求的超时中止时间 单位: s
catch_workers_output = no            #重定向运行过程中的 stderr 和 stdout
security.limit_extensions = ""       #允许 php 解析的文件后缀，默认只解析 .php 文件


```

### php 版本差异
##### php7 比 php5 的差别    
* 性能提升了2倍
* 支持了匿名类，php5.3支持了匿名函数 现在又支持了匿名类
* php7 将所有之前出现的致命错误都改成了异常抛出
##### php8 性能展望
* 关键功能 JIT（just-in-time）将代码转换为另一种字节码的技术，使 php 运行更快



## fpm 优化项

###优化一： fpm 进程占用内存不释放
fpm 作为 php 的进程管理器，每处理一个请求都会将该请求的内存和地址回收，*但并不释放给操作系统*，便于该进程处理下一个请求,久而久之，内存就会被大量占据。
解决方案一：设置一个合适的 pm.min_spare_servers 和 pm.max_spare_servers 参数 使 fpm 进程数始终保持在一个合理的范围内，销毁多余的进程释放内存空间。
解决方案二：降低 pm_max_request 让单个 fpm 进程的请求累积的少一些就将该进程重启，释放内存资源

###优化二：pm.max_children 参数如何设置？
标准回答：根据机器内存决定，一个 fpm 进程占据 20M - 30M 内存。
* 原则上越多越好，php-cgi 进程多了，请求排队的少，并发能力强，处理快
* max_children 配置不合适会造成服务间歇性 502 ,一个 fpm 进程处理的请求累积到 max_request 之后就会重启导致 502 

###优化四：零点峰值突发瞬间流量导致机器 load 飙升。
* 原因：
* `pm.min_spare_servers=5 pm.max_spare_servers=20 `  太小会导致峰值流量来临时待命的 fpm 进程不足，大量的 fork 会使 load 瞬间飙高
* 解决方案：修改 fpm 参数为：
```
pm.min_spare_servers=200
pm.max_spare_servers=300

```













