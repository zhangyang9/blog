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
## fpm 工作机制

理解 Nginx 与 PHP-FPM 通信的工作机制
                    
[TOC]
了解基本原理
浏览器访问网页的过程
请求静态页面
Browser请求http://xxx.com/aa.html -> Web Server（Nginx/Apache）分发 -> 找到aa.html文件返回给Browser。
请求动态脚本
Browser请求http://xxx.com/bb.php -> Web Server（Nginx/Apache）分发 -> PHP解析器（PHP-CGI程序）-> 返回处理结果给Web Server -> 返回数据给Browser。
原理：服务器根据配置文件，知道这是一个PHP脚本文件，需要去找PHP解析器来处理。
 PHP解析器会解析php.ini文件初始化执行环境，然后处理请求，再以标准的数据格式返回处理结果，最后退出进程。
CGI 程序到 FPM 进化史

image
CGI（Common Gateway Interface）
CGI是服务器与后台语言交互的协议，有了这个协议，开发者可以使用任何语言处理服务器转发过来的请求，动态地生成内容，保证了传递过来的数据是标准格式的（规定了以什么样的格式传哪些数据（URL、查询字符串、POST数据、HTTP header等等）），方便了开发者。
PHP-CGI（PHP CGI）
PHP语言对应与服务器交互的CGI程序就是PHP-CGI。
CGI程序本身只能解析请求、返回结果，不会进程管理，所以有一个致命的缺点，那就是每处理一个请求都需要fork一个全新的进程，随着Web的兴起，高并发越来越成为常态，这样低效的方式明显不能满足需求（每一次web请求都会有启动和退出进程，也就是最为人诟病的fork-and-execute模式，这样一在大规模并发下，就死翘翘了）。
就这样，FastCGI诞生了，CGI程序很快就退出了历史的舞台。
FastCGI（Fast CGI）
FastCGI，顾名思义就是更快的CGI程序，用来提高CGI程序性能，它允许在一个进程内处理多个请求，而不是一个请求处理完毕就直接结束进程，性能上有了很大的提高。
* 提高性能？那么CGI程序的性能问题在哪呢？
PHP解析器会解析php.ini文件，初始化执行环境，就是这里了。
标准的CGI程序对每个请求都会执行这些步骤（不闲累啊！启动进程很累的说！），所以处理每个请求的时间会比较长。这明显不合理嘛！
* 那么FastCGI是怎么做的呢？
首先，FastCGI会先启一个master进程，解析配置文件，初始化执行环境，然后再启动多个worker进程。当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。
这样就避免了重复的劳动，效率自然是高。
而且当worker不够用时，master可以根据配置预先启动几个worker等着。
当然空闲worker太多时，也会停掉一些，这样就提高了性能，也节约了资源。这就是FastCGI的对进程的管理。
ps：也有一些能够调度PHP-CGI进程的程序，比如说由lighthttpd分离出来的spawn-fcgi。好了，PHP-FPM也是这么个东东，在长时间的发展后，逐渐得到了大家的认可（要知道前几年大家可是抱怨PHP-FPM稳定性太差的），也越来越流行。
image
PHP-FPM（FastCGI Process Manager）
它是FastCGI协议的一个实现，任何实现了FastCGI协议的服务器都能够与之通信。
 FPM之于标准的FastCGI程序，也提供了一些增强功能，具体可以参考官方文档：PHP: FPM Installation。
*  FPM是一个PHP进程管理器，包含master和worker两种进程。
master进程只有一个，负责监听端口，接收来自服务器的请求，而worker进程则一般有多个（具体数量根据实际需要配置），每个进程内部都嵌入了一个PHP解释器，是PHP代码真正执行的地方，下面是我本机上FPM的进程情况：1个master进程，2个worker进程。
$ ps -ef | grep fpm
root       130     1  0 01:37 ?        00:00:01 php-fpm: master process (/usr/local/php/etc/php-fpm.conf)
php-fpm    131   130  0 01:37 ?        00:00:00 php-fpm: pool www
php-fpm    133   130  0 01:43 ?        00:00:00 php-fpm: pool www
* 从FPM接收到请求，到处理完毕，其具体的流程如下：
1.  FPM的master进程接收到请求。
2.  master进程根据配置指派特定的worker进程进行请求处理，如果没有可用进程，返回错误，这也是我们配合Nginx遇到502错误比较多的原因。
3.  worker进程处理请求，如果超时，返回504错误。
4. 请求处理结束，返回结果。
*  FPM从接收到处理请求的流程就是这样了，那么Nginx又是如何发送请求给FPM的呢？
这就需要从Nginx层面来说明了。
我们知道，Nginx不仅仅是一个Web服务器，也是一个功能强大的Proxy服务器，除了进行http请求的代理，也可以进行许多其他协议请求的代理，包括本文与FPM相关的FastCGI协议。为了能够使Nginx理解FastCGI协议，Nginx提供了FastCGI模块来将http请求映射为对应的FastCGI请求。
Nginx的FastCGI模块提供了fastcgi_param指令来主要处理这些映射关系，下面 是Nginx的一个配置文件实例，其主要完成的工作是将Nginx中的变量翻译成PHP中能够理解的变量。
$ cat /usr/local/nginx/conf/fastcgi.conf
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;
fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;
fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;
fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;
除此之外，非常重要的就是fastcgi_pass指令了，这个指令用于指定FPM进程监听的地址，Nginx会把所有的PHP请求翻译成FastCGI请求之后再发送到这个地址。下面一个简单的可以工作的Nginx配置文件：
server {
    listen 80;
    server_name test.me;
    root /usr/local/web/myproject/public;
    index index.php index.html index.htm;
    access_log /usr/local/nginx/logs/test-access.log;
    error_log  /usr/local/nginx/logs/test-error.log;
    location / {
      try_files $uri $uri/ /index.php?$query_string;
    }
    location ~\.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME /usr/local/web/myproject/public/$fastcgi_script_name;
        fastcgi_pass unix:/usr/local/php/var/run/php-fpm.sock;
        fastcgi_index index.php;
    }
}
在这个配置文件中，我们新建了一个虚拟主机，监听80端口，项目根目录为 /usr/local/web/myproject/public。然后我们通过location指令，将所有的以.php结尾的请求都交给FastCGI模块处理，从而把所有的PHP请求都交给了FPM处理，从而完成Nginx到FPM的闭环。
如此以来，Nginx与FPM通信的整个流程应该比较清晰了。
image
* 修改了php.ini配置文件后，使用PHP-FPM为什么能平滑重启？
修改php.ini之后，PHP-CGI进程是没办法平滑重启的。
 PHP-FPM对此的处理机制是新的worker进程用新的配置，已经存在的worker进程处理完手上的活就可以歇着了，通过这种机制来平滑过渡。
参考via:
深入理解PHP之：Nginx 与 FPM 的工作机制
搞不清FastCgi与PHP-fpm之间是个什么样的关系
你确定你真的懂Nginx与PHP的交互？
CGI、FastCGI和PHP-FPM关系图解













