# Time wait bucket table overflow 报错

## 涉及内核参数： net.ipv4.tcp_max_tw_buchets
参数解析：net.ipv4.tcp_max_tw_buchets 用于调整linux 内核中管理 TIME-WAIT 状态的连接数。当处于 TIME-WAIT 的连接和需要转换为 TIME-WAIT 的连接数之和超过 net.ipv4.tcp_max_tw_buchets 的参数值时，messages 日志就会报错 `time wait bucket table overflow` ，同时内核就会关闭处于 tw 状态的连接。 当发生这种状况时可以根据实际情况调高 net.ipv4.max_tw_buckets 的参数值并优化 tcp 的业务逻辑。


### 问题现象：
/var/log/messages 报错：

``` shell
Feb 18 12:28:38 i-*** kernel: TCP: time wait bucket table overflow
Feb 18 12:28:44 i-*** kernel: printk: 227 messages suppressed.
Feb 18 12:28:44 i-*** kernel: TCP: time wait bucket table overflow
Feb 18 12:28:52 i-*** kernel: printk: 121 messages suppressed.
Feb 18 12:28:52 i-*** kernel: TCP: time wait bucket table overflow
Feb 18 12:28:53 i-*** kernel: printk: 351 messages suppressed.
Feb 18 12:28:53 i-*** kernel: TCP: time wait bucket table overflow
Feb 18 12:28:59 i-*** kernel: printk: 319 messages suppressed.
```

执行命令 `netstat ant | grep TIME_WAIT | wc -l` 统计处于 time-wait 的 tcp 连接数，发现特别大

### 解决办法：
1. 执行命令 `netstat ant | grep TIME_WAIT | wc -l` 统计处于 time-wait 的 tcp 连接数
2. 执行命令`vim /etc/sysctl.conf` 查看 net.ipv4.max_tw_buckets ，调高参数值
3. 执行 `sysctl -p` 使配置生效

