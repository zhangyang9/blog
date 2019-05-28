# Calico 原理解读

Calico
Calico是纯三层的SDN实现，它基于BPG协议和Linux的路由转发机制，不依赖特殊硬件，没有使用NAT或Tunnel等技术。能够方便的部署在物理服务器，虚拟机（如OpenStack）或者容器环境下，可以无缝集成像OpenStack这种IaaS云架构，能够提供可控的VM、容器、裸机之间的IP通信，同时它自带的基于Iptables的ACL管理组件非常灵活，能够满足比较复杂的安全隔离需求。

Clico网络模型的特点：
在Calico中的数据包并不需要进行封包和解封。
Calico中的数据包，只要被policy允许，就可以在不同租户中的workloads间传递或直接接入互联网或从互联网中进到Calico网络中，并不需要像overlay方案中，数据包必须经过一些特定的节点去修改某些属性。
因为是直接基于三层网络进行数据传输，TroubleShooting会更加容易，同时用户也可以直接用一般的工具进行操作与管理，比如ping、Whireshark等，无需考虑解包之类的事情。
网络安全策略使用ACL定义，基于iptables实现，比起overlay方案中的复杂机制更直观和容易操作。

Calico 的核心组件：

Felix：Calico agent，跑在每台需要运行workload的节点上，主要负责配置路由及 ACLs等信息来确保endpoint的连通状态；

etcd：分布式键值存储，主要负责网络元数据一致性，确保Calico网络状态的准确性；

BGP Client(BIRD)：主要负责把Felix写入kernel的路由信息分发到当前Calico网络，确保workload间的通信的有效性；

BGP Route Reflector(BIRD)：大规模部署时使用，摒弃所有节点互联的mesh模式，通过一个或者多个BGP Route Reflector来完成集中式的路由分发；

详细介绍、文档、源码：[Calico Github](https://github.com/projectcalico)
