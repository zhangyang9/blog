# falcon_plus 文档

1. falcon_plus 的架构是什么样子的，分为几个模块？
![falcon架构图](../../pictures/falcon_image1.png)
```
* agent 埋在节点的 agent
* transfer 做数据汇总用的，将数据按照hash规则分片之后将数据push到 graph judge alarm 等模块
* graph 画图
* judge 判断是否告警 将告警数据 event 写入 redis
* alarm 从 redis 读取告警 event ,并发到不同渠道（可做报警合并等优化） 已经发出的告警信息会存储在 mysql 中便于用户读取，默认存储 7 天
* hbs heartbeat server 所有的 agent 每分钟发送一次请求到 hbs 
* nodata 当 agent 超时未上报的时候 nodata 就会自动插入一条 nodata 数据 主要用于判断 agent 死活，补充 judge 模块在 节点 不上报时的数据缺失
* aggraegator 将集群下所有节点的某指标信息聚合


```
2. falcon_plus 的优点有哪些？不足之处有哪些？
```
优点很多，包括部署简单、配置高效、画图和报警策略等自定义程度高，其实 falcon 的每一个模块设计都有它的过人之处，包括，
值得一提的是它的数据模型，它的数据模型采用的是 类似于 opentsdb 相似的数据格式： metric监控指标 + endpoint监控节点 + k-v键值对属性标签。相比于 zabbix 单一的 hostname - metric 数据模型（每增加10个监控指标就要加10条数据）来说， 显然在多级监控和海量指标的场景下 falcon 部署起来更轻量
=========
不足之处也很多，
* 安全性不到位 dashboard、alarmdash 不登录就能看到，恶意用户可以抓站然后输入脏数据等
* 复杂度高，对开发人员来说可能只是简单的配置一个告警，内置的机器分组、策略模板、告警继承等概念不易接受
* 权限设计不够全面，所有的权限都应该有权限控制 现在只做了登录权限
再深层次的缺点的化，看 git 里的 pr 就可以看到，比如代码内部对并发的处理，hash的算法选型等都有过优化
```
2. falcon_plus 的底层存储时什么样子的？有哪些数据存储方式？
```
存储方式： rddtool 、 redis 、 mysql 、 opentsdb


```
3. 在使用过程中对 falcon_plus 的优化点有哪些？


