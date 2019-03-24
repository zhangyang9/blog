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

